# TagMetaView

## Introduction

**`TagMetaView`** is a widget in Supervisely that displays a single TagMeta and provides a convenient way to view the name, color, and tag value type.

## Function signature

```python
tag_meta_view = TagMetaView(
    tag_meta=sly.TagMeta("Animal", sly.TagValueType.ANY_STRING, color=[255, 0, 0]),
    show_type_text=True,
    limit_long_names=False,
    widget_id=None
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/31589e86-d107-4d53-967e-43a48affcf05)

## Parameters

|     Parameters     |    Type   |                            Description                           |
| :----------------: | :-------: | :--------------------------------------------------------------: |
|     `tag_meta`     | `TagMeta` |                       Supervisely tag meta                       |
|  `show_type_text`  |   `bool`  |                 If `True` display tag value type                 |
| `limit_long_names` |   `bool`  | If `False` show the entire tag name if the name is quite lengthy |
|     `widget_id`    |   `str`   |                         ID of the widget                         |

### tag\_meta

Supervisely tag meta

**type:** `TagMeta`

```python
tag_meta = sly.TagMeta(
    name="Animal",
    value_type=sly.TagValueType.ANY_STRING,
    color=[255, 0, 0]
)

tag_meta_view = TagMetaView(tag_meta=tag_meta)
```

### show\_type\_text

If `True` display tag value type next to tag name.

**type:** `bool`

**default value:** `True`

```python
tag_meta = sly.TagMeta(
    name="Animal",
    value_type=sly.TagValueType.ANY_STRING,
    color=[255, 0, 0]
)

tag_meta_view = TagMetaView(tag_meta=tag_meta, show_shape_text=False)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/4313a0c3-d596-40e5-8c66-fa3f92b11b2f)

### limit\_long\_names

If `False` show the entire tag name if the name is quite lengthy

**type:** `bool`

**default value:** `False`

```python
tag_meta = sly.TagMeta(
    name="Animal",
    value_type=sly.TagValueType.ANY_STRING,
    color=[255, 0, 0]
)


tag_meta_view = TagMetaView(
    tag_meta=tag_meta,
    show_shape_icon=True,
    limit_long_names=True
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/fea29ce0-5198-485c-a97a-53a32b1077b5)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/013\_tag\_meta\_view/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/013\_tag\_meta\_view/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Field, TagMetaView, ProjectThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Project ID` we will use

```python
project_id = sly.env.project_id()
```

### Get Project info and meta from server

```python
project_info = api.project.get_info_by_id(id=project_id)
project_meta = api.project.get_meta(id=project_id)
project_meta = sly.ProjectMeta.from_json(project_meta)
```

### Prepare `TagMeta` for each tag in project

Get TagMetas from project meta

```python
tag_metas = project_meta.tag_metas
```

### Initialize `TagMetaView` widget

In this tutorial we will create list of `TagMetaView` objects for each tag in project.

```python
tag_meta_view = [
    TagMetaView(
        tag_meta=tag_meta,
        show_type_text=True,
        limit_long_names=False,
    )
    for tag_meta in tag_metas
]
```

### Create app layout

Prepare a layout for app using `Card`, `Field` widgets with the `content` parameter and place widgets that we've just created in the `Container` widget.

```python
tag_metas_container = Container(widgets=tag_meta_view)
tag_metas_field = Field(
    content=tag_metas_container,
    title="Project tags:",
)

# create widget ProjectThumbnail
project_thumbnail = ProjectThumbnail(project_info)

card = Card(
    title="TagMetaView",
    content=Container(widgets=[project_thumbnail, tag_metas_field]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/98452051-08ea-4596-a515-483a632608bd)
