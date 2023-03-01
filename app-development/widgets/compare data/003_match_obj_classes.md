# Match Objects Classes

## Introduction

In this tutorial you will learn how to use `MatchObjClasses` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/matchobjclasses)

## Function signature

```python
MatchObjClasses(
    left_collection=None,
    right_collection=None,
    left_name=None,
    right_name=None,
    selectable=False,
    suffix=None,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/221399402-fab17435-f5e8-4746-b331-1e33f52b44ae.png)

## Parameters

|     Parameters     |                       Type                        |                                      Description                                       |
| :----------------: | :-----------------------------------------------: | :------------------------------------------------------------------------------------: |
| `left_collection`  | `Union[ObjClassCollection, List[ObjClass], None]` | List of `ObjClass` or `ObjClassCollection`, containing information about left classes  |
| `right_collection` | `Union[ObjClassCollection, List[ObjClass], None]` | List of `ObjClass` or `ObjClassCollection`, containing information about right classes |
|    `left_name`     |                       `str`                       |                                 Left part classes name                                 |
|    `right_name`    |                       `str`                       |                                Right part classes name                                 |
|    `selectable`    |                      `bool`                       |                          Whether the component is selectable                           |
|      `suffix`      |                       `str`                       |                             Suffix to match classes names                              |
|    `widget_id`     |                       `str`                       |                                    Id of the widget                                    |

### left_collection

Determine information about left classes.

**type:** `Union[ObjClassCollection, List[ObjClass], None]`

**default value:** `None`

### right_collection

Determine information about right classes.

**type:** `Union[ObjClassCollection, List[ObjClass], None]`

**default value:** `None`

```python
match = MatchObjClasses(left_collection=left_classes, right_collection=right_classes)
```

![default](https://user-images.githubusercontent.com/120389559/221399402-fab17435-f5e8-4746-b331-1e33f52b44ae.png)

### left_name

Determine left part classes name.

**type:** `Union[str, None]`

**default value:** `None`

### right_name

Determine right part classes name.

**type:** `Union[str, None]`

**default value:** `None`

```python
match = MatchObjClasses(
    left_collection=left_classes,
    right_collection=right_classes,
    left_name="left classes",
    right_name="right classes",
)
```

![left_right](https://user-images.githubusercontent.com/120389559/221399783-5701401c-fc6e-43ff-99e3-58c872b610a5.png)

### selectable

Whether the components are selectable.

**type:** `bool`

**default value:** `False`

```python
match = MatchObjClasses(left_collection=left_classes, right_collection=right_classes, selectable=True)
```

![selectable](https://user-images.githubusercontent.com/120389559/221399859-42ff1a90-14c5-40f9-a99b-26e64af47c6b.gif)

### suffix

Use to match classes names.

**type:** `Union[str, None]`

**default value:** `None`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                                                                                                      Attributes and Methods                                                                                                                      | Description                                      |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | ------------------------------------------------ |
| `set(left_collection: Union[ObjClassCollection, List[ObjClass], None] = None, right_collection: Union[ObjClassCollection, List[ObjClass], None] = None, left_name=Union[str, None] = None, right_name=Union[str, None] = None, suffix: Union[str, None] = None)` | Set input data in left and right part of widget. |
|                                                                                                                           `get_stat()`                                                                                                                           | Return classes match statistics.                 |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/056_match_obj_classes/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/056_match_obj_classes/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, MatchObjClasses
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `ObjClasses` we will matched

```python
project_id_left = int(os.environ["modal.state.slyProjectId_left"])
meta_json_left = api.project.get_meta(project_id_left)
project_meta_left = sly.ProjectMeta.from_json(meta_json_left)
left_classes = project_meta_left.obj_classes

project_id_right = int(os.environ["modal.state.slyProjectId_right"])
meta_json_right = api.project.get_meta(project_id_right)
project_meta_right = sly.ProjectMeta.from_json(meta_json_right)
right_classes = project_meta_right._obj_classes
```

### Initialize `MatchObjClasses` widget, initiate `suffix` to match classes with similar names

```python
match = MatchObjClasses(
    left_collection=left_classes, right_collection=right_classes, suffix="erity"
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Match Classes",
    content=match,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/221400207-007f741a-8d1c-47aa-8eaa-1179d634b043.gif)
