# TagsListSelector

## Introduction

**`Tags List Selector`** widget allows users to view a list of tag metas and change it dynamically using widget methods. This widget can be used for visualizing and selecting tags in Supervisely.

## Function signature

```python
tags_list_selector = TagsListSelector(
    tag_metas=tag_metas,
    multiple=False,
    empty_notification=None,
    widget_id=None
)
```

{% tabs %}
{% tab title="Default with tags" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/334ccb23-e30c-4d5e-938c-95620f523389" alt=""><figcaption><p>Default with tags</p></figcaption></figure>
{% endtab %}

{% tab title="Default without tags" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/c8991571-a40e-41c5-93e4-53f9482f2739" alt=""><figcaption><p>Default without tags</p></figcaption></figure>
{% endtab %}
{% endtabs %}

## Parameters

|      Parameters      |                    Type                   |                              Description                             |
| :------------------: | :---------------------------------------: | :------------------------------------------------------------------: |
|      `tag_metas`     | `Union[List[TagMeta], TagMetaCollection]` |         Supervisely tag metas collection or list of tag metas        |
|      `multiple`      |                   `bool`                  |                    Enable multiple tags selection                    |
| `empty_notification` |             `NotificationBox`             | Notification that will be displayed when there are no tags in widget |
|      `widget_id`     |                   `str`                   |                           ID of the widget                           |

### tag\_metas

List of `TagMeta` objects or Supervisely object class collection (`TagMetaCollection`).

**type:** `Union[List[TagMeta], TagMetaCollection]`

```python
tags_list_selector = TagsListSelector(
    tag_metas=tag_metas
)
```

### multiple

If `True` then multiple tags can be selected. Otherwise, only one tag can be selected.

**type:** `bool`

**default** `False`

```python
tags_list_selector = TagsListSelector(
    tag_metas=tag_metas,
    multiple=True
)
```

{% tabs %}
{% tab title="Multiple Enabled" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/f80f6c40-5157-4295-bb24-8fc25cd3e8df" alt=""><figcaption><p>Multiple Enabled</p></figcaption></figure>
{% endtab %}

{% tab title="Multiple disabled" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e35b72ac-20b0-4c3b-84ee-e8417de19aaa" alt=""><figcaption><p>Multiple disabled</p></figcaption></figure>
{% endtab %}
{% endtabs %}

### empty\_notification

Notification that will be displayed when there are no tags in widget

**type:** `NotificationBox`

**default** `None`

```python
notification_box = NotificationBox(title="No tags", description="Provide tags to the widget.")
tags_list_selector = TagsListSelector(
    tag_metas=tag_metas,
    empty_notification=notification_box,
)
```

![custom-empty-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/ee8a0c24-ff5e-43c1-a5b2-e6411d93ab9f)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                  |
| :--------------------: | -------------------------------------------- |
|         `set()`        | Set tags to widget.                          |
|  `get_selected_tags()` | Return list of selected tags.                |
|     `select_all()`     | Select all tags.                             |
|    `deselect_all()`    | Deselect all tags.                           |
|       `select()`       | Select tags by names.                        |
|      `deselect()`      | Deselect tags by names.                      |
|    `set_multiple()`    | Set multiple flag.                           |
|    `get_all_tags()`    | Return list of all tags.                     |
|  `@selection_changed`  | Callback triggers when selection is changed. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/017\_tags\_list\_selector/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/017\_tags\_list\_selector/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, TagsListSelector, Text, NotificationBox
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create list of tag metas

```python
cat_tag_meta = sly.TagMeta("cat", sly.TagValueType.NONE)
dog_tag_meta = sly.TagMeta("dog", sly.TagValueType.ANY_NUMBER)
horse_tag_meta = sly.TagMeta("horse", sly.TagValueType.ANY_STRING)
cow_tag_meta = sly.TagMeta("cow", sly.TagValueType.ONEOF_STRING, possible_values=["moo", "mooo"])
zebra_tag_meta = sly.TagMeta("zebra", sly.TagValueType.NONE)
tag_metas = [
    cat_tag_meta,
    dog_tag_meta,
    horse_tag_meta,
    cow_tag_meta,
    zebra_tag_meta,
]

```

### Initialize `TagsListSelector` widget, `NotificationBox` widget for custom notification and `Text` widget for displaying selected tags count

```python
notification_box = NotificationBox(title="No tags", description="Provide tags to the widget.")
tags_list_selector = TagsListSelector(tag_metas, multiple=True, empty_notification=notification_box)
selected_tags_cnt = Text(f"Selected Tags: 0 / {len(tag_metas)}")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created into the `Container` widget.

```python
container = Container(widgets=[selected_tags_cnt, tags_list_selector])

card = Card(
    title="Tags List Selector",
    content=container,
)

layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python
@tags_list_selector.selection_changed
def on_selection_changed(selected_tags):
    selected_tags_cnt.set(f"Selected Tags: {len(selected_tags)} / {len(tag_metas)}", "text")
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/45d1283f-39fc-492b-aa3a-c4eb9155764a)
