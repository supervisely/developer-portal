# SelectClass

## Introduction

**`SelectClass`** widget in Supervisely is a dropdown menu that allows users to select object classes from a predefined list of `ObjClass` objects and also create new classes on the fly. It supports features like filtering and multiple selection. This widget is commonly used in applications that need to work with annotation classes, such as filtering objects, class mapping, or configuring model training parameters.

The `SelectClass` widget includes event handlers for tracking class selection changes and new class creation events.

## Function signature

```python
SelectClass(
    classes=[],
    filterable=True,
    placeholder="Select class",
    show_add_new_class=True,
    size=None,
    multiple=False,
    widget_id=None,
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765891962281.png" alt=""><figcaption></figcaption></figure>

## Parameters

|      Parameters      |                     Type                      |                   Description                   |
| :------------------: | :-------------------------------------------: | :---------------------------------------------: |
|      `classes`       |  `Union[List[ObjClass], ObjClassCollection]`  | List of ObjClass objects or ObjClassCollection. |
|     `filterable`     |               `Optional[bool]`                | Enable search/filter functionality in dropdown. |
|    `placeholder`     |                `Optional[str]`                |   Placeholder text when no class is selected.   |
| `show_add_new_class` |               `Optional[bool]`                | Whether to show button for adding new classes.  |
|        `size`        | `Optional[Literal["large", "small", "mini"]]` |          Size of the select dropdown.           |
|      `multiple`      |                    `bool`                     |     Whether multiple selection is enabled.      |
|     `widget_id`      |                `Optional[str]`                |            Unique widget identifier.            |

### classes

Initial list of `ObjClass` instances.

**type:** `Union[List[ObjClass], ObjClassCollection]`

**default value:** `[]`

```python
cat_obj_class = sly.ObjClass("cat", sly.Rectangle)
dog_obj_class = sly.ObjClass("dog", sly.Rectangle)
horse_obj_class = sly.ObjClass("horse", sly.Rectangle)
sheep_obj_class = sly.ObjClass("sheep", sly.Rectangle)

obj_classes = [cat_obj_class, dog_obj_class, horse_obj_class, sheep_obj_class]

select_class = SelectClass(classes=obj_classes)
```

### filterable

Enable or disable filtering/searching functionality in the dropdown.

**type:** `Optional[bool]`

**default value:** `True`

```python
select_class = SelectClass(classes=obj_classes, filterable=True)
```

### placeholder

Set placeholder text shown when no class is selected.

**type:** `Optional[str]`

**default value:** `"Select class"`

```python
select_class = SelectClass(classes=obj_classes, placeholder="Select a class")
```

### multiple

Enable multiple class selection.

**type:** `bool`

**default value:** `False`

```python
select_class = SelectClass(obj_classes, multiple=True)
```

### show\_add\_new\_class

Show or hide the "Add New Class" button that allows users to create new classes directly from the widget.

**type:** `Optional[bool]`

**default value:** `True`

```python
select_class = SelectClass(classes=obj_classes, show_add_new_class=True)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765891980532.png" alt=""><figcaption></figcaption></figure>

### size

Set the size of the widget.

**type:** `Optional[Literal["large", "small", "mini"]]`

**default value:** `None`

```python
select_class = SelectClass(classes=obj_classes, size="small")
```

### widget\_id

Unique widget identifier.

**type:** `Optional[str]`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                     |
| :--------------------: | --------------------------------------------------------------- |
| `get_selected_class()` | Get the currently selected ObjClass object(s).                  |
|     `get_value()`      | Get the currently selected class name(s).                       |
|     `set_value()`      | Sets the selected class by name or list of names.               |
|  `get_all_classes()`   | Returns a copy of all available classes.                        |
|        `set()`         | Updates the list of available classes and refreshes the widget. |
|    `@value_changed`    | Decorator for value change event.                               |
|    `@class_created`    | Decorator for new class creation event.                         |

## Mini App Example

You can find this example in our GitHub repository:

[ui-widgets-demos/selection/026\_select\_class/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/026\_select\_class/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, NotificationBox, SelectClass
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create object classes

```python
cat_obj_class = sly.ObjClass("cat", sly.Rectangle)
dog_obj_class = sly.ObjClass("dog", sly.Rectangle)
horse_obj_class = sly.ObjClass("horse", sly.Rectangle)
sheep_obj_class = sly.ObjClass("sheep", sly.Rectangle)

obj_classes = [
    cat_obj_class,
    dog_obj_class,
    horse_obj_class,
    sheep_obj_class,
]
```

### Initialize `SelectClass` widget

```python
select_class = SelectClass(
    classes=obj_classes,
    multiple=True,
    show_add_new_class=True
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
notification_box = NotificationBox()

card = Card(
    title="Select Class",
    content=Container(widgets=[select_class, notification_box]),
)

layout = Container(widgets=[card])

app = sly.Application(layout=layout)
```

### Handle class selection changes

```python
@select_class.value_changed
def on_class_selected(class_name):
    selected_class = select_class.get_selected_class()
```

### Handle new class creation

```python
@select_class.class_created
def on_class_created(new_class: sly.ObjClass):
    notification_box.set(
        title="New class created",
        description=f"You have created a new class: {new_class.name}",
    )
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765892006355.png" alt=""><figcaption></figcaption></figure>
