# Grid

## Introduction

In this tutorial you will learn how to use `Grid` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/grid)

## Function signature

```python
Grid(widgets, columns=1, gap=10, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218113232-54061027-c0c2-4c9a-abc6-199808da9755.png)

## Parameters

| Parameters  |      Type      |             Description              |
| :---------: | :------------: | :----------------------------------: |
|  `widgets`  | `List[Widget]` | List if widgets to display on `Grid` |
|  `columns`  |     `int`      |     Number of columns on `Grid`      |
|    `gap`    |     `int`      |    Gap between widgets on `Grid`     |
| `widget_id` |     `str`      |           Id of the widget           |

### widgets

Determine list of `Widgets` to display on `Grid`.

**type:** `List[Widget]`

### columns

Number of columns on `Grid`.

**type:** `int`

**default value:** `1`

```python
grid = Grid(widgets=obj_class_view_widgets, columns=3)
```

![columns](https://user-images.githubusercontent.com/120389559/218115665-afdff2fd-ff83-417f-8637-894627ea70c8.png)

### gap

Determine gap between `Widgets` on `Grid`.

**type:** `int`

**default value:** `10`

```python
grid = Grid(widgets=obj_class_view_widgets, columns=3, gap=50)
```

![gap](https://user-images.githubusercontent.com/120389559/218116106-778e8c3e-8663-4b9a-96d7-3aef82190be8.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/028_grid/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/028_grid/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Grid, ObjectClassView
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize widgets we will use in UI

```python
obj_class_cat = sly.ObjClass(name="cat", geometry_type=sly.Bitmap, color=[255, 0, 0])
obj_class_dog = sly.ObjClass(name="dog", geometry_type=sly.Bitmap, color=[0, 255, 0])
obj_class_sheep = sly.ObjClass(name="sheep", geometry_type=sly.Bitmap, color=[0, 0, 255])
obj_class_horse = sly.ObjClass(name="horse", geometry_type=sly.Bitmap, color=[255, 255, 0])
obj_class_squirrel = sly.ObjClass(name="squirrel", geometry_type=sly.Bitmap, color=[255, 0, 255])

obj_classes = [
    obj_class_cat,
    obj_class_dog,
    obj_class_sheep,
    obj_class_horse,
    obj_class_squirrel,
]

obj_class_view_widgets = [ObjectClassView(obj_class=obj_class) for obj_class in obj_classes]
```

### Initialize `Grid` widget

```python
grid = Grid(
    widgets=obj_class_view_widgets,
    columns=3,
    gap=50,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Grid",
    content=grid,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218116106-778e8c3e-8663-4b9a-96d7-3aef82190be8.png)
