# ClassesColorMapping

## Introduction

**`Classes Color Mapping`** widget allows users to edit class colors on the fly. This widget can be used when you have classes with similar colors and you want to change them to be more distinguishable.

### Function signature

```python
classes_color_mapping = ClassesColorMapping(
    classes=obj_classes,
    grayscale=False,
    widget_id=None
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/defa52ff-f287-4967-ba37-31f88b7284ed)

## Parameters

|  Parameters |                     Type                    |                          Description                          |
| :---------: | :-----------------------------------------: | :-----------------------------------------------------------: |
|  `classes`  | `Union[List[ObjClass], ObjClassCollection]` | Supervisely object class collection or list of object classes |
| `grayscale` |                    `bool`                   |       If `True` convert selected RGB color to grayscale       |
| `widget_id` |                    `str`                    |                        ID of the widget                       |

### classes

List of `ObjClass` objects or Supervisely object class collection (`ObjClassCollection`).

**type:** `Union[List[ObjClass], ObjClassCollection]`

```python
classes_color_mapping = ClassesColorMapping(
    classes=obj_classes,
)
```

### grayscale

If `True` convert selected RGB color value to grayscale.

**type:** `bool`

**default** `False`

```python
classes_color_mapping = ClassesColorMapping(
    classes=obj_classes,
    grayscale=True
)
```

![grayscale-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/623ec1a1-1be3-4f18-b7fa-3a58d69ea9b5)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|       Attributes and Methods      | Description                       |
| :-------------------------------: | --------------------------------- |
|              `set()`              | Set classes to widget.            |
|           `set_colors()`          | Set classes colors.               |
|          `get_classes()`          | Get all classes.                  |
| `get_selected_classes_original()` | Get classes with original colors. |
|  `get_selected_classes_edited()`  | Get classes with selected colors. |
|          `get_mapping()`          | Get class to color map.           |
|          `set_default()`          | Set multiple flag.                |
|             `select()`            | Return list of all classes.       |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/input/010\_classes\_color\_mapping/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/input/010\_classes\_color\_mapping/src/main.py)

#### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    ClassesColorMapping,
    ClassesListPreview,
    Button,
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

### Create list of object classes and init `ClassesColorMapping` widget

```python
cat_obj_class = sly.ObjClass("cat", sly.Rectangle, color=[255, 0, 0])
dog_obj_class = sly.ObjClass("dog", sly.Polygon, color=[0, 255, 0])
horse_obj_class = sly.ObjClass("horse", sly.Bitmap, color=[0, 0, 255])
cow_obj_class = sly.ObjClass("cow", sly.Polyline, color=[255, 255, 0])
zebra_obj_class = sly.ObjClass("zebra", sly.Point, color=[255, 0, 255])
obj_classes = [
    cat_obj_class,
    dog_obj_class,
    horse_obj_class,
    cow_obj_class,
    zebra_obj_class,
]

classes_color_mapping = ClassesColorMapping(obj_classes)
```

### Initialize `ClassesListPreview` widget to display changes and `Button` widget to save changes

```python
classes_list_preview = ClassesListPreview()
new_classes_names = Text(f"Press button to save changes", "info")
save_button = Button("Save", button_size="mini")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created into the `Container` widget.

```python
container = Container([classes_color_mapping, new_classes_names, save_button, classes_list_preview])

card = Card(
    title="Classes Color Mapping",
    content=container,
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
@save_button.click
def save_changes():
    new_classes = classes_color_mapping.get_selected_classes_edited()
    classes_list_preview.set(new_classes)
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/6462951d-7c8a-4e6f-9afa-716ab3558aa1)
