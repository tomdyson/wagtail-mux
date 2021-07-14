# Wagtail-mux

Upload videos to Mux from the Wagtail admin, keep a local record of Mux assets
and use them in Wagtail pages.

## Local development

1. Create a `.env` file at the root of your project (next to `manage.py`)
2. Add Mux API credentials to `.env`, e.g.

```
MUX_TOKEN_ID=your id
MUX_TOKEN_SECRET=your secret
```

3. Add the app's URLs to your project's `urls.py` (for webhook support):

```python
# Add to imports
from mux import urls as mux_urls

# Add to urlpatterns
urlpatterns = urlpatterns + [
  path("mux/", include(mux_urls)),
]
```

4. In a virtual environment:

```bash
pip install -r requirements.txt
./manage.py migrate
./manage.py createsuperuser
./manage.py mux_sync
./manage.py runserver
```

5. Use ngrok or similar to expose your local development to the web (for Mux
   webhooks):

```bash
ngrok http 8000
```

Then add the HTTPS URL + `/mux/webhook` to the Webhooks setting in the Mux
admin.

6. Add Mux videos to your pages, e.g.

```python
class VideoPage(Page):
    date = models.DateField("Publish date")
    body = RichTextField()
    video = models.ForeignKey(
        "mux.Video", null=True, blank=True, on_delete=models.SET_NULL, related_name="+"
    )

    content_panels = Page.content_panels + [
        FieldPanel("date"),
        FieldPanel("body", classname="full"),
        ModelChooserPanel("video"), # https://github.com/neon-jungle/wagtailmodelchooser
    ]
```

Your template might look something like this:

```html
{% extends "base.html" %}
{% load wagtailcore_tags %}

{% block extra_css %}
  <link rel="stylesheet" href="https://unpkg.com/@mux/videojs-kit@0.1/dist/index.css">
{% endblock %}

{% block content %}
  {% if page.video.mux_public_playback_id %}
    <video id="my-player" class="video-js vjs-16-9" controls preload="auto" width="100%" data-setup='{}'>
      <source src="{{ page.video.mux_public_playback_id }}" type="video/mux" />
    </video>
  {% endif %}
{% endblock %}

{% block extra_js %}
  <script src="https://unpkg.com/@mux/videojs-kit@0.1/dist/index.js"></script>
{% endblock %}
```

## In production

- Make your Mux API credentials available as environment variables
- Update the webhook settings in the Mux admin

## TODO

- [x] Create a Video model with title, tags and Mux references (asset and
  playback)
- [x] Sync Mux assets with local Video model
- [x] Make endpoint to trigger sync from Mux webhooks
- [x] Handle deletions in webhook
- [x] Use modeladmin instead of snippets
- [x] Use a model chooser (e.g.
  [wagtailmodelchooser](https://github.com/neon-jungle/wagtailmodelchooser))
  instead of the snippet chooser
- [x] Enable
  [direct upload](https://docs.mux.com/guides/video/upload-files-directly) from
  admin screen
- [ ] Record demo
- [ ] Remove page models, simplify migrations
- [ ] Create package
  - needs `mux-python==2.0.0rc1` and `wagtail-modelchooser`
- [ ] Handle updates in webhook (e.g. subtitles added)
