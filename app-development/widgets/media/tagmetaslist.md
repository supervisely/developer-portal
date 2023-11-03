# Tag Metas List

## Introduction

**`TagMetasList`** widget allows users to view a list of tag metas, it provide a flexible display option with a choice of single-column or multiple-column layouts. It also allows users to select or deselect one or more tags, making it easy to manage and organize object classes. This widget is a useful tool for visualizing and selecting tags in Supervisely.

## Function signature

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    show_type_text=True,
    limit_long_names=True,
    selectable=True,
    columns=1,
    widget_id=None
)
```

![tagmetas-list-default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/2362213e-b21b-43d4-9dcb-9dbfdb9b93e6)

## Parameters

|     Parameters     |                    Type                    |                           Description                            |
| :----------------: | :----------------------------------------: | :--------------------------------------------------------------: |
|    `tag_metas`     | `Union[TagMetasCollection, List[TagMeta]]` |  Supervisely object class collection or list of object classes   |
|  `show_type_text`  |                   `bool`                   |                 If `True` display tag value type                 |
| `limit_long_names` |                   `bool`                   | If `False` show the entire tag name if the name is quite lengthy |
|    `selectable`    |                   `bool`                   |                     Enable classes selection                     |
|     `columns`      |                   `int`                    |                        Number of columns                         |
|    `widget_id`     |                   `str`                    |                         ID of the widget                         |

### tag_metas

Supervisely object class collection (`TagMetaCollection`) or list of `TagMeta`.

**type:** `Union[TagMetaCollection, List[TagMeta]]`

```python
tag_metas_list = TagMetaList(
    tag_metas=project_meta.tag_metas,
    selectable=False,
    columns=1,
)
```

![tag-metas](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/cd82cf76-55eb-4fea-883a-b1ffd3180900)

### show_type_text

If `True` display tag value type next to tag name.

**type:** `bool`

**default value:** `True`

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    show_shape_text=False,
    selectable=False,
)
```

![tag-metas-no-type](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/1fc3379c-4f90-447a-b4aa-61f91e7a8183)

### limit_long_names

If `False` show the entire tag name if the name is quite lengthy

**type:** `bool`

**default value:** `False`

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    show_shape_icon=True,
    limit_long_names=False,
    selectable=False,
)
```

![tag-metas-limit-name](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/b9d40902-25ba-40d6-8788-a124dba0c7c1)

### selectable

Enable tags selection.

**type:** `bool`

**default** `False`

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    selectable=True
)
```

![tagmetas-selectable](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/2362213e-b21b-43d4-9dcb-9dbfdb9b93e6)

### columns

Number of columns.

**type:** `int`

**default** `1`

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    show_shape_icon=True,
    limit_long_names=True,
    selectable=True,
    columns=3,
)
```

![tag-metas-columns](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/81b52cd3-50b3-4647-8a58-e26c0883228a)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|   Attributes and Methods   | Description                        |
| :------------------------: | ---------------------------------- |
| `get_selected_tag_names()` | Return list of selected tag names. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/014_tag_metas_list/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/014_tag_metas_list/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, TagMetasList, ProjectThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Project ID`

```python
project_id = sly.env.project_id()
```

### Get Project info and meta

```python
project_info = api.project.get_info_by_id(id=project_id)
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
```

### Initialize `TagMetasList` widget

```python
tag_metas_list = TagMetasList(
    tag_metas=project_meta.tag_metas,
    show_type_text=True,
    limit_long_names=True,
    selectable=True,
    columns=1,
)
```

### Add button and preview

```python
show_selected_button = Button("Show tags")
tags_preview = Text("", "text")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
project_thumbnail = ProjectThumbnail(project_info)

card = Card(
    title="TagMetasList",
    content=Container(widgets=[project_thumbnail, tag_metas_list, show_selected_button]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![miniapp](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e76282ad-cb79-42e3-b080-6291035b7947)

### Add button click event to update preview

```python
@show_selected_button.click
def show_selected_tags():
    selected_tag_names = tag_metas_list.get_selected_tag_names()
    tags_preview.set(" ,".join(selected_tag_names), "text")
```
