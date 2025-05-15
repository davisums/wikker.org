---
title: A Django language toggle that also works with Wagtail
description: The default language toggle in Django doesn't work with Wagtail. And the Wagtail toggle, doesn't work with Django. Here's a solution that works with both.
date: 2025-05-15
scheduled: 2025-05-15
tags: startups
layout: layouts/post.njk
image: https://cdn.pixabay.com/photo/2020/08/30/20/54/rice-field-5530707_1280.jpg
---

I've been working on a Django project that uses Wagtail for the CMS. The website is multilingual so I wanted a way for users to easily switch between languages.

## The Django way

The default way of doing this according to the [django docs](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#the-set-language-redirect-view) is to use the `set_language` view. Basically add this to your urls.py and then have a form that posts to it.

```
path("i18n/", include("django.conf.urls.i18n")),
```

And this is what the form looks like:

{% raw %}
```
{% load i18n %}

<form action="{% url 'set_language' %}" method="post">{% csrf_token %}
  <input name="next" type="hidden" value="{{ redirect_to }}">
  <select name="language">
    {% get_current_language as LANGUAGE_CODE %}
    {% get_available_languages as LANGUAGES %}
    {% get_language_info_list for LANGUAGES as languages %}
    {% for language in languages %}
    <option value="{{ language.code }}" {% if language.code==LANGUAGE_CODE %} selected{% endif %}>
      {{ language.name_local }} ({{ language.code }})
    </option>
    {% endfor %}
  </select>
  <input type="submit" value="Go">
</form>
```
{% endraw %}

## The Wagtail way

The previous method was working fine, until I added Wagtail to the project. Wagtail pages were not being translated correctly by the `set_language` view. Luckily, the Wagtail docs show [how to add a language toggle for Wagtail](https://docs.wagtail.org/en/stable/advanced_topics/i18n.html#language-region-selector) pages.

{% raw %}
```
{% load wagtailcore_tags %}

{% if page %}
{% for translation in page.get_translations.live %}
<a href="{% pageurl translation %}" rel="alternate" hreflang="{{ translation.locale.language_code }}">
  {{ translation.locale.language_name_local }}
</a>
{% endfor %}
{% endif %}
```
{% endraw %}

There's a few problems with this one too though:

1. It doesn't work for non-Wagtail Django pages
2. set_language doesn't only change the page language and URL, it also updates the session language. The Wagtail solution doesn't seem to do this.

## Combining the toggle for Django + Wagtail

So, I hacked together a custom view that works with both Django and Wagtail. I'm not sure how robust it is, but it seems to work as I need it to.

```
path("change-language/", views.change_language, name="change_language"),
```

When the view is called for a Wagtail page, it does the following:
- If the request is for a Wagtail page, it gets the page from the Wagtail page id in the request. It then gets the translated URL with the built-in Wagtail method `page.get_translation(locale)`
- Since I didn't want to change the set_language function, in the case of a Wagtail page I needed to overwrite the "next" value in `request.POST`, which isn't really recommended.

If the request is for a non-Wagtail page, it just uses the default Django `set_language` view.

```
def change_language(request):
    wagtail_page_id = request.POST.get("wagtail_page_id")

    if not wagtail_page_id:
        return set_language(request)

    lang_code = request.POST.get(LANGUAGE_QUERY_PARAMETER)

    page = Page.objects.get(id=wagtail_page_id)
    locale = Locale.objects.get(language_code=lang_code)
    request.POST = request.POST.copy()

    try:
        translated_page = page.get_translation(locale)
    except Page.DoesNotExist:
        translated_page = None

    if translated_page and translated_page.live:
        request.POST["next"] = translated_page.get_full_url(request)
        return set_language(request)
    else:
        full_url = request.build_absolute_uri(reverse("index"))
        request.POST["next"] = full_url
        messages.info(request, "We don't have a translation yet for the previous page.")
        return set_language(request)
```

And here's the template code for the HTML form. It includes a hidden input for the Wagtail page id

{% raw %}
```
<form action="{% url 'change_language' %}" method="post">
  {% csrf_token %}
  <input name="next" type="hidden" value="{{ redirect_to }}">
  {% if page and page.is_root == False %}
  <input name="wagtail_page_id" type="hidden" value="{{ page.id }}">
  {% endif %}
  <select name="language" class="">
    {% get_current_language as LANGUAGE_CODE %}
    {% get_available_languages as LANGUAGES %}
    {% get_language_info_list for LANGUAGES as languages %}
    {% for language in languages %}
    <option value="{{ language.code }}" {% if language.code==LANGUAGE_CODE %} selected{% endif %} class="uppercase">
      {{ language.code }}
    </option>
    {% endfor %}
  </select>
  <input type="submit" value="Go">
</form>
```
{% endraw %}