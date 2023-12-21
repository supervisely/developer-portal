# ClassesMapping

## Introduction

**`Classes Mapping`** widget allows to rename given object classes with new names. It can be useful when you want to rename classes in your project.

## Function signature

```python
classes_mapping = ClassesMapping(
    classes=obj_classes,
    empty_notification=None,
    widget_id=None
)
```

{% tabs %}
{% tab title="Default with classes" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/bd3d8f91-3c49-43c5-b800-f40c19233239" alt=""><figcaption><p>Default with classes</p></figcaption></figure>
{% endtab %}

{% tab title="Default without classes" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/48ca6ab4-fd23-4980-843d-59cddcd3cb50" alt=""><figcaption><p>Default without classes</p></figcaption></figure>
{% endtab %}
{% endtabs %}

## Parameters

|      Parameters      |                     Type                    |                               Description                               |
| :------------------: | :-----------------------------------------: | :---------------------------------------------------------------------: |
|       `classes`      | `Union[List[ObjClass], ObjClassCollection]` |      Supervisely object class collection or list of object classes      |
| `empty_notification` |              `NotificationBox`              | Notification that will be displayed when there are no classes in widget |
|      `widget_id`     |                    `str`                    |                             ID of the widget                            |

### classes

List of `ObjClass` objects or Supervisely object class collection (`ObjClassCollection`).

**type:** `Union[List[ObjClass], ObjClassCollection]`

```python
classes_mapping = ClassesMapping(
    classes=obj_classes
)
```

### empty\_notification

Notification that will be displayed when there are no classes in the widget.

**type:** `NotificationBox`

**default** `None`

```python
empty_notificaiton = NotificationBox(
    title="No classes", description="Provide classes to widget in order to map new names."
)
classes_mapping = ClassesMapping(
    classes=obj_classes,
    empty_notification=empty_notificaiton
)
```

![custom-empty-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/65e45304-e7cb-484e-af45-6c1e43d8b1c4)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                               |
| :--------------------: | --------------------------------------------------------- |
|         `set()`        | Set classes to widget.                                    |
|     `get_classes()`    | Return list of all classes.                               |
|     `get_mapping()`    | Return edited classes mapping, if there were any changes. |
|       `ignore()`       | Ignore classes by indexes.                                |
|     `set_default()`    | Revert changes.                                           |
|     `set_mapping()`    | Set new classes mapping.                                  |
|     `select_all()`     | Select all classes.                                       |
|    `deselect_all()`    | Deselect all classes.                                     |
|       `select()`       | Select classes by `ObjClass`.                             |
|  `@selection_changed`  | Callback triggers when selection is changed.              |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/input/009\_classes\_mapping/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/input/009\_classes\_mapping/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, ClassesMapping, Button, Text, NotificationBox
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create list of object classes

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
```

### Initialize `ClassesMapping` widget, `NotificationBox` widget for custom notification

```python
empty_notification = NotificationBox(
    title="No classes", description="Provide classes to widget in order to map new names."
)
classes_mapping = ClassesMapping(
    classes=obj_classes,
    empty_notification=empty_notification
)
```

### Create `Text` widget for displaying changes and `Button` widget for saving changes

```python
new_classes_names = Text(f"Press button to save changes", "info")
save_button = Button("Save", button_size="mini")
```

### Create app layout

Prepare a layout for the app using the `Card` widget with the `content` parameter and place the widget that we've just created into the `Container` widget.

```python
container = Container([classes_mapping, new_classes_names, save_button])

card = Card(
    title="Classes Mapping",
    content=container,
)
layout = Container(widgets=[card])
```

### Create the app using the layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from the python code

```python
@save_button.click
def save_mapping():
    new_classes_names.hide()
    mapping = classes_mapping.get_mapping()
    new_class_names = []
    for obj_class in obj_classes:
        if obj_class.name in mapping:
            new_class_name = mapping[obj_class.name]["value"]
            if new_class_name != obj_class.name:
                new_class_names.append(new_class_name)
    if len(new_class_names) == 0:
        new_classes_names.set(f"No changes detected", "info")
    else:
        new_classes_names.set(f"New classes names: {','.join(new_class_names)}", "success")
    new_classes_names.show()
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e1c4d3ee-65e1-43b3-abac-fa4739b221d4)
