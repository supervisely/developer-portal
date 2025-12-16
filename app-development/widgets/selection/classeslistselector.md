# Classes List Selector

## Introduction

**`Classes List Selector`** widget allows users to view a list of object classes and change it dynamically using widget methods. This widget can be used for visualizing and selecting classes in Supervisely.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/apps-with-gui/classes-list-selector)

## Function signature

```python
classes_list_selector = ClassesListSelector(
    classes=obj_classes,
    multiple=True,
    empty_notification=None,
    widget_id=None
)

```

{% tabs %}
{% tab title="Default with classes" %}

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/dad85bb2-aa50-49e2-b4f4-ee567d7584b0" alt=""><figcaption><p>Default with classes</p></figcaption></figure>
{% endtab %}

{% tab title="Default without classes" %}

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/cfc129dc-7a29-4aa1-a171-f0579547c78e" alt=""><figcaption><p>Default without classes</p></figcaption></figure>
{% endtab %}
{% endtabs %}

## Parameters

|      Parameters      |                    Type                     |                               Description                               |
| :------------------: | :-----------------------------------------: | :---------------------------------------------------------------------: |
|      `classes`       | `Union[List[ObjClass], ObjClassCollection]` |      Supervisely object class collection or list of object classes      |
|      `multiple`      |                   `bool`                    |                    Enable multiple classes selection                    |
| `empty_notification` |              `NotificationBox`              | Notification that will be displayed when there are no classes in widget |
|     `widget_id`      |                    `str`                    |                            ID of the widget                             |
| `allow_new_classes`  |                   `bool`                    |                  Allow adding new classes dynamically                   |

### classes

List of `ObjClass` objects or Supervisely object class collection (`ObjClassCollection`).

**type:** `Union[List[ObjClass], ObjClassCollection]`

```python
classes_list_selector = ClassesListSelector(
    classes=obj_classes
)
```

### multiple

If `True` then multiple classes can be selected. Otherwise, only one class can be selected.

**type:** `bool`

**default** `False`

```python
classes_list_selector = ClassesListSelector(
    classes=obj_classes,
    multiple=True
)
```

{% tabs %}
{% tab title="Multiple Enabled" %}

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/008b2ca2-6077-47ae-9491-167646c23a32" alt=""><figcaption><p>Multiple Enabled</p></figcaption></figure>
{% endtab %}

{% tab title="Multiple disabled" %}

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/9e94991c-4d30-4e0a-8ca0-aecbbc7f5c73" alt=""><figcaption><p>Multiple disabled</p></figcaption></figure>
{% endtab %}
{% endtabs %}

### empty_notification

Notification that will be displayed when there are no classes in widget

**type:** `NotificationBox`

**default** `None`

```python
notification_box = NotificationBox(title="No classes", description="Provide classes to the widget.")
classes_list_selector = ClassesListSelector(
    multiple=True,
    empty_notification=notification_box
)
```

![custom-no-classes](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/3987b67e-be8d-4762-b149-6cf73db6a869)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

### allow_new_classes

Allow adding new classes dynamically.

**type:** `bool`

**default value:** `False`

## Methods and attributes

|  Attributes and Methods  | Description                                   |
| :----------------------: | --------------------------------------------- |
|         `set()`          | Set classes to widget.                        |
| `get_selected_classes()` | Return list of selected classes.              |
|      `select_all()`      | Select all classes.                           |
|     `deselect_all()`     | Deselect all classes.                         |
|        `select()`        | Select classes by names.                      |
|       `deselect()`       | Deselect classes by names.                    |
|     `set_multiple()`     | Set multiple flag.                            |
|   `get_all_classes()`    | Return list of all classes.                   |
|   `@selection_changed`   | Callback triggers when selection is changed.  |
|     `@class_created`     | Decorator to handle new class creation event. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/010_object_classes_list/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/010_object_classes_list/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    ClassesListSelector,
    NotificationBox,
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

### Create list of object classes

```python
cat_obj_class = sly.ObjClass("cat", sly.Rectangle)
dog_obj_class = sly.ObjClass("dog", sly.Rectangle)
horse_obj_class = sly.ObjClass("horse", sly.Rectangle)
sheep_obj_class = sly.ObjClass("sheep", sly.Rectangle)
cow_obj_class = sly.ObjClass("cow", sly.Rectangle)
elephant_obj_class = sly.ObjClass("elephant", sly.Rectangle)
bear_obj_class = sly.ObjClass("bear", sly.Rectangle)
zebra_obj_class = sly.ObjClass("zebra", sly.Rectangle)
obj_classes = [
    cat_obj_class,
    dog_obj_class,
    horse_obj_class,
    sheep_obj_class,
    cow_obj_class,
    elephant_obj_class,
    bear_obj_class,
    zebra_obj_class,
]
```

### Initialize `ClassesListSelector` widget with class creation enabled

```python
notification_box = NotificationBox(title="No classes", description="Provide classes to the widget.")
classes_list_selector = ClassesListSelector(
    obj_classes, multiple=True, empty_notification=notification_box, allow_new_classes=True
)

selected_classes_cnt = Text(f"Selected classes: 0 / {len(obj_classes)}")
info_text = Text("You can create new classes using the Add new class button", status="info")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created into the `Container` widget.

```python
container = Container(
    widgets=[
        info_text,
        selected_classes_cnt,
        classes_list_selector,
    ]
)

card = Card(
    title="Classes List Selector",
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
# Update counters helper
def update_counters():
    all_classes = classes_list_selector.get_all_classes()
    selected = classes_list_selector.get_selected_classes()
    selected_classes_cnt.set(f"Selected classes: {len(selected)} / {len(all_classes)}", "text")


@classes_list_selector.selection_changed
def selection_changed(classes):
    update_counters()


@classes_list_selector.class_created
def on_class_created(new_class):
    info_text.set(f"New class created: '{new_class.name}' ({new_class.geometry_type.name()})", "success")
    update_counters()
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/5cd483ab-c620-4af3-8158-2a07b0e87e69)
