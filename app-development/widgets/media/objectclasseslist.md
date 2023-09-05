# Object Classes List

## Introduction

**`ObjectClassesList`** is a widget that allows users to view a list of all given object classes. The widget provides a flexible display option with a choice of single-column or multiple-column layouts. It also allows users to select or deselect one or more classes, making it easy to manage and organize object classes. This widget is a useful tool for visualizing and selecting classes in Supervisely.

## Function signature

```python
obj_classes_list = ObjectClassesList(
    object_classes=project_meta.obj_classes,
    selectable=False,
    columns=1,
    widget_id=None
)
```

![objclass-list-default](https://user-images.githubusercontent.com/79905215/218096273-0a52cc67-0ba8-4886-95a0-660e61d6a4eb.png)

## Parameters

|    Parameters    |                    Type                     |                          Description                          |
| :--------------: | :-----------------------------------------: | :-----------------------------------------------------------: |
| `object_classes` | `Union[ObjClassCollection, List[ObjClass]]` | Supervisely object class collection or list of object classes |
|   `selectable`   |                   `bool`                    |                   Enable classes selection                    |
|    `columns`     |                    `int`                    |                       Number of columns                       |
|   `widget_id`    |                    `str`                    |                       ID of the widget                        |

### object_classes

Supervisely object class collection (`ObjClassCollection`) or list of `ObjClass`.

**type:** `Union[ObjClassCollection, List[ObjClass]]`

```python
obj_classes_list = ObjectClassesList(
    object_classes=project_meta.obj_classes
)
```

### selectable

Enable classes selection.

**type:** `bool`

**default** `False`

```python
obj_classes_list = ObjectClassesList(
    object_classes=project_meta.obj_classes,
    selectable=True
)
```

![objclass-list](https://user-images.githubusercontent.com/79905215/218092421-a4c996bb-9679-428b-899d-4bb29d570112.png)

### columns

Number of columns.

**type:** `int`

**default** `1`

```python
obj_classes_list = ObjectClassesList(
    object_classes=project_meta.obj_classes,
    columns=2
)
```

![objclass-list-columns](https://user-images.githubusercontent.com/79905215/218098608-4c22b892-24a0-4b93-8da4-812e0c08d333.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|  Attributes and Methods  | Description                      |
| :----------------------: | -------------------------------- |
| `get_selected_classes()` | Return list of selected classes. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/010_object_classes_list/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/010_object_classes_list/src/main.py)

### Import libraries

```python
import os
from random import randint

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container
from supervisely.app.widgets import Image, ObjectClassesList, ProjectThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Project ID` and `Dataset ID` we will use

```python
project_id = sly.env.project_id()
dataset_id = sly.env.dataset_id()
```

### Get Project info and meta

```python
project_info = api.project.get_info_by_id(id=project_id)

project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
```

### Initialize `ObjectClassesList` widget

```python
obj_classes_list = ObjectClassesList(
    object_classes=project_meta.obj_classes,
    selectable=True,
    columns=3,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
project_thumbnail = ProjectThumbnail(project_info)

cls_select_btn = Button(text="PREVIEW")

select_class_container = Container(widgets=[obj_classes_list, cls_select_btn])
image = Image()

preview_container = Container(
    widgets=[select_class_container, image],
    direction="horizontal",
    fractions=[2, 1],
)

card = Card(
    title="Object Classes List",
    content=Container(widgets=[project_thumbnail, preview_container]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python
@cls_select_btn.click
def show_info():
    classes = obj_classes_list.get_selected_classes()
    obj_classes = api.object_class.get_list(
        project_id=project_id, filters=[{"field": "name", "operator": "in", "value": classes}]
    )

    imgs = api.image.get_filtered_list(
        dataset_id=dataset_id,
        filters=[{"type": "objects_class", "data": {"classId": obj.id}} for obj in obj_classes],
    )
    if len(imgs) == 0:
        image.clean_up()
        return
    index = randint(0, len(imgs) - 1)
    image.set(imgs[index].full_storage_url)
```

![objclasslist-app](https://user-images.githubusercontent.com/79905215/219017000-0bbcdeaa-bd99-459b-908a-4269b4c8e074.gif)
