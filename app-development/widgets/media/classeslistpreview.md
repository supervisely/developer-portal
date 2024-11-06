# ClassesListPreview

## Introduction

**`Classes List Preview`** widget just displays a list of classes. It can be used to display classes that were selected by the user in the [Classes List Selector](https://developer.supervisely.com/app-development/apps-with-gui/classes-list-selector) widget for example.

## Function signature

```python
classes_list_preview = ClassesListPreview(
    classes=obj_classes,
    max_height="128px",
    empty_text = None,
    show_shape_title=True,
    show_shape_icon=True,
    widget_id = None,
    )
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/6e287a14-c34c-423e-87e0-a00d3f8b9267)

## Parameters

|     Parameters     |                     Type                    |                           Description                           |
| :----------------: | :-----------------------------------------: | :-------------------------------------------------------------: |
|      `classes`     | `Union[List[ObjClass], ObjClassCollection]` |  Supervisely object class collection or list of object classes  |
|    `max_height`    |                    `bool`                   |                     Max height of the widget                    |
|    `empty_text`    |                    `str`                    | Text that will be displayed when there are no classes in widget |
| `show_shape_title` |                    `bool`                   |       If `True` show name of the shape next to class name       |
|  `show_shape_icon` |                    `bool`                   |         If `True` show icon of the shape near class name        |
|     `widget_id`    |                    `str`                    |                         ID of the widget                        |

### classes

List of `ObjClass` objects or Supervisely object class collection (`ObjClassCollection`).

**type:** `Union[List[ObjClass], ObjClassCollection]`

```python
classes_list_preview = ClassesListPreview(
    classes=obj_classes
)
```

### max\_height

Set the maximum height of the widget in pixels. If the content exceeds the maximum height, the scroll bar will appear.

**type:** `str`

**default** `"128px"`

```python
classes_list_preview = ClassesListPreview(
    classes=obj_classes,
    max_height="100px"
)
```

### empty\_text

Text that will be displayed when there are no classes in widget

**type:** `str`

**default** `None`

```python
empty_text = "No classes to preview"
classes_list_preview = ClassesListPreview(
    classes=[],
    empty_text=empty_text,
)
```

![empty-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/ca229395-0c19-4615-9f4c-8396c750c3f5)

### show\_shape\_title

If `True` show name of the shape next to class name.

**type:** `bool`

**default** `True`

```python
empty_text = "No classes to preview"
classes_list_preview = ClassesListPreview(
    classes=[],
    empty_text=empty_text,
)
```

![disable-shape-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/5a92c443-98cb-4f58-9033-63545a75fc12)

### show\_shape\_icon

If `True` show icon of the shape near class name.

**type:** `bool`

**default** `True`

```python
empty_text = "No classes to preview"
classes_list_preview = ClassesListPreview(
    classes=[],
    empty_text=empty_text,
)
```

![disable-shape-icon](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/bcbb5f4a-6d07-4b72-8ae0-a08e8123cfab)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                                 |
| :--------------------: | --------------------------------------------------------------------------- |
|         `set()`        | Set classes to widget and determine wheter to display shape icon and title. |
|         `get()`        | Return list of classes.                                                     |

## Mini App Example

In this example we will create a mini app with `ClassesListPreview` widget. We will create a `ClassesListSelector` widget and display selected classes with `ClassesListPreview` widget.

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/017\_classes\_list\_preview/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/017\_classes\_list\_preview/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    ClassesListPreview,
    ClassesListSelector,
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

### Create list of object classes and init `ClassesListSelector` widget

```python
cat_obj_class = sly.ObjClass("cat", sly.Rectangle)
dog_obj_class = sly.ObjClass("dog", sly.Polygon)
horse_obj_class = sly.ObjClass("horse", sly.Bitmap)
cow_obj_class = sly.ObjClass("cow", sly.Polyline)
zebra_obj_class = sly.ObjClass("zebra", sly.Point)
obj_classes = [
    cat_obj_class,
    dog_obj_class,
    horse_obj_class,
    cow_obj_class,
    zebra_obj_class,
]

classes_list_selector = ClassesListSelector(obj_classes, multiple=True)
```

### Initialize `ClassesListPreview` widget and `Text` widget for displaying number of selected classes

```python
classes_list_preview = ClassesListPreview()
preview_text = Text(f"Selected Classes: 0 / {len(obj_classes)}", "text")
preview_container = Container([preview_text, classes_list_preview])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've previously created into the `Container` widget.

```python
container = Container(widgets=[classes_list_selector, preview_container])

card = Card(
    title="Classes List Preview",
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
@classes_list_selector.selection_changed
def on_selection_changed(selected_classes):
    preview_text.set(f"Selected Classes: {len(selected_classes)} / {len(obj_classes)}", "text")
    classes_list_preview.set(selected_classes)
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/568a4185-91b0-41fe-959b-0e65f90aceae)
