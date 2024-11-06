# ElementTagsList

## Introduction

**`ElementTagsList`** widget in Supervisely is a widget that allows users to display multiple [elements tags](https://element.eleme.io/1.4/#/en-US/component/tag) in the UI.

## Function signature

```python
ElementTagsList(tags=tags, widget_id=None)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/1aad8b9f-d78f-4da9-b980-18f2da9592e9)

## Parameters

|  Parameters |             Type            |              Description              |
| :---------: | :-------------------------: | :-----------------------------------: |
|    `tags`   | `List[ElementTagsList.Tag]` | List of `ElementTagsList.Tag` objects |
| `widget_id` |            `str`            |            ID of the widget           |

### tags

List of `ElementTagsList.Tag` objects, that will be displayed in the UI. `ElementTagsList.Tag` object is based on `ElementTag` widget and has the same arguments. You can learn about them in the [ElementTag](https://developer.supervisely.com/app-development/widgets/text-elements/elementtag) tutorial.

If you want to add close button to the tag, you can use `ElementTagsList.Tag` object with `closable=True` argument.

**type:** `List[ElementTagsList.Tag]`

```python
tag_names = ["Tag 1", "Tag 2", "Tag 3"]
tags = []
for tag_name in tag_names:
    el_tag = ElementTagsList.Tag(text=tag_name, closable=True)
    tags.append(el_tag)

el_tags_list = ElementTagsList(tags=tags)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/353d38d5-c618-4b98-b37c-d5d4150c6366)

### widget\_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                             |
| :--------------------: | --------------------------------------- |
|      `set_tags()`      | Set tags to widget                      |
|      `get_tags()`      | Get current tags                        |
|      `add_tags()`      | Add tags to existing tags in the widget |
|        `close()`       | Triggers on close button                |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/text elements/009\_element\_tags\_list/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text%20elements/009\_element\_tags\_list/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, ElementTagsList
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create Tag objects for ElementTagsList widget

```python
el_tag = ElementTagsList.Tag(text="Tag")
all_tag_types = [el_tag]
for tag_type in ["primary", "gray", "success", "warning", "danger"]:
    curr_tag = ElementTagsList.Tag(
        text=f"Tag {tag_type}", type=tag_type, hit=True, closable=True, close_transition=False
    )
    all_tag_types.append(curr_tag)
```

### Initialize ElementTagsList widget

```python
el_tags_list = ElementTagsList(tags=all_tag_types)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
card = Card(
    "Element Tags List",
    content=el_tags_list,
)

layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add handlers to the widget

```
@el_tags_list.close
def close_tag(current_tags):
    el_tags_list.add_tags([ElementTagsList.Tag(text="New Tag", closable=True)])
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/688e21c1-5a45-4d4e-8540-7caeddc08353)
