# Object Class View

## Introduction

**`ObjectClassView`** is a widget in Supervisely that displays a single object class and provides a convenient way to view the name, color, as well as the icon and shape type of the class.


## Function signature

```python
obj_class_view = ObjectClassView(
    obj_class=sly.ObjClass("cat", sly.Bitmap, [255, 0, 0]),
    show_shape_text=True,
    show_shape_icon=True,
    widget_id=None
)
```

![objclass-default](https://user-images.githubusercontent.com/79905215/218079475-c5c5c032-8420-4850-b3fc-19dfc19c266a.png)

## Parameters

|    Parameters     |    Type    |         Description          |
| :---------------: | :--------: | :--------------------------: |
|    `obj_class`    | `ObjClass` |   Supervisely object class   |
| `show_shape_text` |   `bool`   | If `True` display shape text |
| `show_shape_icon` |   `bool`   | If `True` display shape icon |
|    `widget_id`    |   `str`    |       ID of the widget       |

### obj_class

Supervisely object class

**type:** `ObjClass`

```python
obj_class = sly.ObjClass(
    name="cat",
    geometry_type=sly.Bitmap,
    color=[255, 0, 0]
)

obj_class_view = ObjectClassView(obj_class=obj_class)
```

### show_shape_text

Display shape text of object class

**type:** `bool`

**default value:** `True`

```python
obj_class_view = ObjectClassView(
    obj_class=obj_class,
    show_shape_text=False
)
```

![objclass-show-text](https://user-images.githubusercontent.com/79905215/218081019-0d0d2ebe-69a8-4e7d-b1ce-e647b005dd7b.png)

### show_shape_icon

Display object class icon

**type:** `bool`

**default value:** `False`

```python
obj_class_view = ObjectClassView(
    obj_class=obj_class,
    show_shape_icon=True
)
```

![objclass-show-icon](https://user-images.githubusercontent.com/79905215/218080581-9344eb4a-3696-4c75-b9ff-8f1ec96722b7.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/009_object_class_view/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/009_object_class_view/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Field, ObjectClassView, ProjectThumbnail
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
```

### Prepare `ObjClass` for each class in project

Prepare dictionary to get geometry type by geometry name

```python
SHAPES_TYPES = {
    "any": sly.AnyGeometry,
    "bitmap": sly.Bitmap,
    "point": sly.Point,
    "polygon": sly.Polygon,
    "rectangle": sly.Rectangle,
    "line": sly.Polyline,
}
```

create ObjClass for each class in project

```python
classes = [
    sly.ObjClass(
        name=obj["title"],
        geometry_type=SHAPES_TYPES[obj["shape"]],
    )
    for obj in project_meta["classes"]
]
```

### Create `ObjClassCollection` from ObjClasses

```python
classes = sly.ObjClassCollection(classes)
```

### Initialize `ObjectClassView` widget

In this tutorial we will create list of `ObjectClassView` objects for each class in project.

```python
obj_class_view = [
    ObjectClassView(
        obj_class=obj_class,
        show_shape_text=True,
        show_shape_icon=True,
    )
    for obj_class in classes
]
```

### Create app layout

Prepare a layout for app using `Card`, `Field` widgets with the `content` parameter and place widgets that we've just created in the `Container` widget.

```python
obj_class_container = Container(widgets=obj_class_view)
obj_class_field = Field(
    content=obj_class_container,
    title="Project classes:",
)

project_thumbnail = ProjectThumbnail(project_info)

card = Card(
    title="ObjectClassView",
    content=Container(widgets=[project_thumbnail, obj_class_field]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![objclassview-app](https://user-images.githubusercontent.com/79905215/218984921-7bad1e0e-5600-4230-b069-9c457961b49b.png)
