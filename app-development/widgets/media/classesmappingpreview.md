# ClassesMappingPreview

## Introduction

**`Classes List Preview`** widget displays classes mapping. It can be used to display new names of classes that were edited by the user in the [Classes Mapping](https://developer.supervisely.com/app-development/apps-with-gui/classes-mapping) widget for example.

## Function signature

```python
classes_mapping_preview = ClassesMappingPreview(
    classes=obj_classes,
    mapping=mapping,
    max_height="128px",
    widget_id=None,
)

```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/48ed7fc1-2d05-4444-8899-16be3362a9e7)

## Parameters

|  Parameters  |                     Type                    |                           Description                           |
| :----------: | :-----------------------------------------: | :-------------------------------------------------------------: |
|   `classes`  | `Union[List[ObjClass], ObjClassCollection]` |  Supervisely object class collection or list of object classes  |
|   `mapping`  |               `Dict[str,str]`               | Dictionary where keys are class names and values are new names. |
| `max_height` |                    `str`                    | Text that will be displayed when there are no classes in widget |
|  `widget_id` |                    `str`                    |                         ID of the widget                        |

### classes

List of `ObjClass` objects or Supervisely object class collection (`ObjClassCollection`).

**type:** `Union[List[ObjClass], ObjClassCollection]`

```python
classes_mapping_preview = ClassesMappingPreview(
    classes=obj_classes
)
```

![classes](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/72c2010a-23c1-4ae8-bd7d-ebe8c722bf78)

### mapping

Dictionary where keys are original class names and values are new names.

**type:** `Dict[str,str]`

**default** `None`

```python
mapping = {obj_class.name: f"{obj_class.name}-detection" for obj_class in obj_classes}
classes_mapping_preview = ClassesMappingPreview(
    classes=obj_classes,
    mapping=mapping
)
```

![mapping](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/3e7f54c4-bf62-421c-903c-0e840988a65c)

### max\_height

Set the maximum height of the widget in pixels. If the content exceeds the maximum height, the scroll bar will appear.

**type:** `str`

**default** `"128px"`

```python
classes_mapping_preview = ClassesMappingPreview(
    classes=obj_classes,
    max_height="100px"
)
```

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                        |
| :--------------------: | ---------------------------------- |
|         `set()`        | Set classes and mapping to widget. |
|     `set_mapping()`    | Set new mapping.                   |
|     `get_mapping()`    | Return current class mapping.      |
|     `get_classes()`    | Return list of classes.            |

## Mini App Example

In this example we will create a mini app with `ClassesMappingPreview` widget. We will create a `ClassesMapping` widget and display changed classes with `ClassesMappingPreview` widget.

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/016\_classes\_mapping\_preview/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/016\_classes\_mapping\_preview/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    ClassesMapping,
    ClassesMappingPreview,
    Button,
    NotificationBox,
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

### Create list of object classes and init `ClassesMapping` widget

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

empty_notification = NotificationBox(
    title="No classes", description="Provide classes to widget in order to map new names."
)
classes_mapping = ClassesMapping(obj_classes, empty_notification=empty_notification)
```

### Initialize `ClassesMappingPreview` widget and `Button` widget for saving and displaying mapping

```python
classes_mapping_preview = ClassesMappingPreview()
save_button = Button("Save", button_size="mini")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've previously created into the `Container` widget.

```python
container = Container([classes_mapping, save_button, classes_mapping_preview])

card = Card(
    title="Classes Mapping Preview",
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
def save_mapping():
    mapping = classes_mapping.get_mapping()
    classes_mapping_preview.set(obj_classes, mapping)
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/5a12d76e-1ce8-49dd-9ec0-1040ccba54ef)
