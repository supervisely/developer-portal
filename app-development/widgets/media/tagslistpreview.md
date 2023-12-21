# TagsListPreview

## Introduction

**`Tags List Preview`** widget simply displays a list of tags. It can be used to display tags that were selected by the user in the [Tags List Selector](https://developer.supervise.ly/app-development/apps-with-gui/tags-list-selector) widget for example.

## Function signature

```python
tags_list_preview = TagsListPreview(
        tag_metas=tag_metas,
        max_width=300,
        empty_text=None,
        widget_id=None,
    )
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/26ec99cc-5536-4a33-9692-393571b0afc7)

## Parameters

|  Parameters  |                     Type                    |                          Description                         |
| :----------: | :-----------------------------------------: | :----------------------------------------------------------: |
|  `tag_metas` | `Union[List[ObjClass], ObjClassCollection]` |     Supervisely tag meta collection or list of tag metas     |
|  `max_width` |                    `int`                    |                    Max width of the widget                   |
| `empty_text` |                    `str`                    | Text that will be displayed when there are no tags in widget |
|  `widget_id` |                    `str`                    |                       ID of the widget                       |

### tag\_metas

List of `TagMeta` objects or Supervisely tag meta collection (`TagMetaCollection`).

**type:** `Union[List[TagMeta], TagMetaCollection]`

```python
tags_list_preview = TagsListPreview(
    tag_metas=tag_metas
)
```

### max\_width

Set the maximum width of the widget in pixels.

**type:** `int`

**default** `300`

```python
tags_list_preview = TagsListPreview(
    tag_metas=tag_metas,
    max_height="100px"
)
```

### empty\_text

Text that will be displayed when there are no tags in widget

**type:** `str`

**default** `None`

```python
empty_text = "No tags selected"
tags_list_preview = TagsListPreview(
    tag_metas=[]
    empty_text=empty_text,
)
```

![empty-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/a662f22d-52d8-4ccb-89a5-ea668984fb09)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description              |
| :--------------------: | ------------------------ |
|         `set()`        | Set tag metas to widget. |

## Mini App Example

In this example we will create a mini app with `TagsListPreview` widget. We will create a `TagsListSelector` widget and display selected tags with `TagsListPreview` widget.

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/018\_tags\_list\_preview/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/018\_tags\_list\_preview/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    TagsListPreview,
    TagsListSelector,
    Text,
)
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create list of tag metas and init `TagsListSelector` widget

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

tags_list_selector = TagsListSelector(tag_metas, multiple=True)
```

### Initialize `TagsListPreview` widget and `Text` widget for displaying number of selected tags

```python
empty_text = Text("No tags selected", "text")
tags_list_preview = TagsListPreview(empty_text=empty_text)
preview_text = Text(f"Selected Tags: 0 / {len(tag_metas)}", "text")
preview_container = Container([preview_text, tags_list_preview])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've previously created into the `Container` widget.

```python
container = Container(widgets=[tags_list_selector, preview_container])

card = Card(
    title="Tags List Preview",
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
    preview_text.set(f"Selected Tags: {len(selected_tags)} / {len(tag_metas)}", "text")
    tags_list_preview.set(selected_tags)
```

![miniapp-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/a0582ead-10bc-4eb3-9ddd-ccca22268aef)
