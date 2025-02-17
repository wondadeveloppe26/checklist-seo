# Check list SEO ![Build](https://travis-ci.org/itarverne/checklist-seo.svg?branch=master) 


[![Maintainability](https://api.codeclimate.com/v1/badges/1ea9094958cb77a0c1a9/maintainability)](https://codeclimate.com/github/itarverne/checklist-seo/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/1ea9094958cb77a0c1a9/test_coverage)](https://codeclimate.com/github/itarverne/checklist-seo/test_coverage)

All the SEO check in app Django built for Wagtail

![](./seo/static/images/seo_logo.png)

# Features

<table>
    <thead>
        <tr>
            <th>Result</th>
            <th>Features</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=5><img src="./seo/static/images/seo_panel.png" /></td>
	        <td>Keyword repartition</td>
	    </tr>
	    <tr>
            <td>Length content </td>
	    </tr>
	    <tr>
            <td>Check title article length</td>
	    </tr>
	    <tr>
            <td>Url is optimized</td>
	    </tr>
	    <tr>
            <td>Number internal Links</td>
	    </tr>
    </tbody>
</table>

# Installation

## Pypi

[![PyPI version](https://img.shields.io/pypi/v/checklist-seo)](https://pypi.org/project/checklist-seo/)

`pip install checklist-seo`

## Installing the application in Django

To use this application, you need first to add it to your config file.

In your config file (ex: settings.py):

```
# Application definition

INSTALLED_APPS = [
	...
	'seo'
	...
]
```

## SEO Panel

To setup the keyword for SEO, you need to add a special SEO Panel that will appear in your page creation in wagtail admin.

The module contains a model in models/SeoPage, the model need to be used as a base for your page models.

Example of your model:

```python
class HomePage(SeoPage):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    delay = models.IntegerField(default=0, validators=[MaxValueValidator(99), MinValueValidator(0)])
    body = StreamField([
        ('text', RichTextBlock(blank=True, features=['h2', 'h3', 'h4', 'bold', 'italic', 'link',
                                                     'code', 'ol', 'ul', 'hr', 'document-link', 'image', 'embed', 'superscript', 'subscript', 'strikethrough', 'blockquote'])),
        ('rawHtml', RawHTMLBlock(blank=True)),
    ], blank=True)
    images_keyword = models.CharField(max_length=250, blank=True)
    selected_image = models.ForeignKey(
        'wagtailimages.Image',
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name='+'
    )

    keep_slug = models.BooleanField(
        verbose_name=('Keep current slug'),
        default=False,
        help_text=("Keep current slug or save to generate a new slug.")
    )

    def _get_autogenerated_slug(self, base_slug):
        """Redefinition of wagtail's _get_autogenerated_slug so you can use your own slug generator."""
        return self.slug

    search_fields = Page.search_fields + [
        index.SearchField('intro'),
    ]

    content_panels = Page.content_panels + [
        MultiFieldPanel([
            FieldPanel('date'),
            FieldRowPanel([
                FieldPanel('delay'),
            ]),
        ], heading="Blog information"),
        FieldPanel('intro'),
        StreamFieldPanel('body'),
        FieldRowPanel([
            FieldPanel('images_keyword'),
        ], heading="Images"),
        ImageChooserPanel(field_name="selected_image", heading="Image sélectionnée"),
    ]

    promote_panels = [
        MultiFieldPanel([
            FieldPanel('slug'),
            FieldPanel('keep_slug'),
            FieldPanel('seo_title'),
            FieldPanel('show_in_menus'),
            FieldPanel('search_description'),
        ], heading="Common Page Configuration"),
    ]

    edit_handler = TabbedInterface([
        ObjectList(content_panels, heading='Content'),
        ObjectList(promote_panels, heading="Promote"),
        SeoPage.seo_object_list,
        ObjectList(Page.settings_panels, heading='Settings')
    ])
```

## Routing

In your routing projet file `urls.py`
```python
from django.conf.urls import url
from django.urls import include

urlpatterns = [
    ...
    url(r'^seo/', include('seo.urls'), name='seo'),
]
```

## Static

To get the CSS / JS / Image file from this app to your projet
`python manage.py collectstatic`

## DB Migration

Now you can detect the change
`python manage.py makemigrations`

And apply it on DB
`python manage.py migration`


## Test

`pytest`
