# Match Tag Metas

## Introduction

In this tutorial you will learn how to use `MatchTagMetas` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/matchtagmetas)

## Function signature

```python
MatchTagMetas(
    left_collection=None,
    right_collection=None,
    left_name=None,
    right_name=None,
    selectable=False,
    suffix=None,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/221365567-f5b7359e-9e92-49be-910a-b613d566f7f2.png)

## Parameters

|     Parameters     |                      Type                       |                                    Description                                    |
| :----------------: | :---------------------------------------------: | :-------------------------------------------------------------------------------: |
| `left_collection`  | `Union[TagMetaCollection, List[TagMeta], None]` | List of `TagMeta` or `TagMetaCollection`, containing information about left tags  |
| `right_collection` | `Union[TagMetaCollection, List[TagMeta], None]` | List of `TagMeta` or `TagMetaCollection`, containing information about right tags |
|    `left_name`     |                      `str`                      |                                Left part tags name                                |
|    `right_name`    |                      `str`                      |                               Right part tags name                                |
|    `selectable`    |                     `bool`                      |                        Whether the component is selectable                        |
|      `suffix`      |                      `str`                      |                             Suffix to match tag names                             |
|    `widget_id`     |                      `str`                      |                                 Id of the widget                                  |

### left_collection

Determine information about left tags.

**type:** `Union[TagMetaCollection, List[TagMeta], None]`

**default value:** `None`

### right_collection

Determine information about right tags.

**type:** `Union[TagMetaCollection, List[TagMeta], None]`

**default value:** `None`

```python
match = MatchTagMetas(left_collection=tag_metas_left, right_collection=tag_metas_right)
```

![default](https://user-images.githubusercontent.com/120389559/221365567-f5b7359e-9e92-49be-910a-b613d566f7f2.png)

### left_name

Determine left part tags name.

**type:** `Union[str, None]`

**default value:** `None`

### right_name

Determine right part tags name.

**type:** `Union[str, None]`

**default value:** `None`

```python
match = MatchTagMetas(
    left_collection=tag_metas_left,
    right_collection=tag_metas_right,
    left_name="left tags",
    right_name="right tags",
)
```

![left_right](https://user-images.githubusercontent.com/120389559/221367169-9c4e1a6a-6eda-4330-8c27-d60e315525d3.png)

### selectable

Whether the components are selectable.

**type:** `bool`

**default value:** `False`

```python
match = MatchTagMetas(left_collection=tag_metas_left, right_collection=tag_metas_right, selectable=True)
```

![selectable](https://user-images.githubusercontent.com/120389559/221365872-27f6442c-7e0c-4e0e-bd64-8dedad7e75c9.gif)

### suffix

Use to match tag names.

**type:** `Union[str, None]`

**default value:** `None`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                                                                                                    Attributes and Methods                                                                                                                    | Description                                      |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | ------------------------------------------------ |
| `set(left_collection: Union[TagMetaCollection, List[TagMeta], None] = None, right_collection: Union[TagMetaCollection, List[TagMeta], None] = None, left_name=Union[str, None] = None, right_name=Union[str, None] = None, suffix: Union[str, None] = None)` | Set input data in left and right part of widget. |
|                                                                                                                         `get_stat()`                                                                                                                         | Return tags match statistics.                    |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/055_match_tags/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/055_match_tags/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, MatchTagMetas
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `TagMeta` we will matched

```python
project_id_left = int(os.environ["modal.state.slyProjectId_left"])
meta_json_left = api.project.get_meta(project_id_left)
project_meta_left = sly.ProjectMeta.from_json(meta_json_left)
tag_metas_left = project_meta_left.tag_metas

project_id_right = int(os.environ["modal.state.slyProjectId_right"])
meta_json_right = api.project.get_meta(project_id_right)
project_meta_right = sly.ProjectMeta.from_json(meta_json_right)
tag_metas_right = project_meta_right.tag_metas
```

### Initialize `MatchTagMetas` widget, initiate `suffix` to match tags with similar names

```python
match = MatchTagMetas(
    left_collection=tag_metas_left, right_collection=tag_metas_right, suffix="afes"
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Match Tags",
    content=match,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/221366720-e23eb733-9a9d-4ce6-bb0c-7dc45c1d4f60.gif)
