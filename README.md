PagesBundle
==================

 * [configuration](#configuration)
 * [page data](#pagedata-usage)
 * [translations](#translations)
 * [static texts](#static-texts)

Adds support for pages (with their routes - slug).

 * slug is generated from "**title**" field
 * if some page has set "**is_homepage**" property to true, then request on "/" path is matched with this page
 * if you set new homepage, other "homepage" pages are restored to be regular pages

## configuration

Default configuration is as follows (so if you don't want to change default templates you don't have to configure anything)

```yml
pages:
    page_default_template: PagesBundle:page:index.html.twig
    widget_default_template: PagesBundle:widget/templates:default.html.twig
```

## Assoc structure of entites
 * Page `N -- N` PageWidget
 * PageWidget `N -- N` PageWidgetData

**Page**

| title | content | is_homepage | slug | template |
| ----------------- | ------------- | ----------------- | ------------- | ----------------- | ------------- |
| Homepage |  lorem ipsum  | 1 | /homepage | PagesBundle:page:homepage.html.twig |

**PageWidget**

| title | content | position | ordering | template |
| ----------------- | ------------- | ----------------- | ------------- | ----------------- | ------------- |
| HP widget |  lorem ipsum  | top | 1 | PagesBundle:widget/templates:custom.html.twig |


 **PageWidgetData**

| name | entity | conditions | ordering | count |
| ----------------- | ------------- | ----------------- | ------------- | ------------- |
| example_clinic_listing |  \AppBundle\Entity\Clinic  | [] | name DESC, id ASC | 3 |


## Pagedata - usage
create widget with its own data; data example here:

 * **name**: `example_clinic_listing` - name of field in *page.data* array
 * **entity**: `\AppBundle\Entity\Clinic`
 * **conditions**:
```
[
{
 "method": "gte",
 "params": [ "id", 1 ]
},
{ ... }
]
```
 * **count**: `3` - max 3 records (0 or blank = unlimited)
 * **ordering**: `name DESC, id ASC` - ordering of records

in twig template:
```twig
// PagesBundle/Resources/views/page/homepage.html.twig
{% dump(page) %}

// used to render all widgets with "position" property set to "top"
{% include 'PagesBundle:widget:render.html.twig' with { 'widgets': page.widgets, 'position': 'top' } only %}
```

```twig
// PagesBundle/Resources/views/widget/render.html.twig
{% set tpl_default = 'PagesBundle:widget/templates:default.html.twig' %}

{% for widget in widgets if widget.position == position %}
	{% include [widget.template, tpl_default] with { 'widget': widget } only %}
{% endfor %}
```

```twig
// PagesBundle/Resources/views/widget/templates/default.html.twig
<div class="widget">
    {{widget.content | raw}}
    {{ dump(widget.data) }}
    // i.e. widget.data['example_clinic_listing']
</div>
```


## translations

translation files: `XXXBundle/Resources/translations/messages.XX.yml`
examples with all possible cases (simple text, text with variable, plural text)

```yml
# messages.en.yml
basic example: translated text
var example: translated text with %var%
choice example: "{0} There are no apples|{1} There is one apple|]1,Inf[ There are %count% apples %var%"
```

twig template:

```
{{ 'basic example'|trans }}
{{ 'var example'|trans({'%var%': 'Fabien'}) }}
{{ 'choice example'|transchoice(5, {'%var%': 'Fabien'}) }}
```

## # static texts
you can set some static texts to `Page` entity in admin panel (field "texts").

```
{
    "sample_header": "H1 texst",
    "another_text": "Lorem ipsum.."
}
```

twig template:

```
{{ _text['sample_header'] }}
```
