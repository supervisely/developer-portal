# SelectClass

## Introduction

**`SelectClass`** widget is a compact dropdown menu that allows users to select an object class (or multiple classes) from a list of available classes. Each item displays the class name, its color and its geometry type (shape). The widget can also let users create a new class on the fly via the "Add new class" option.

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

## Parameters

|      Parameters      |                              Type                              |                  Description                  |
| :------------------: | :------------------------------------------------------------: | :-------------------------------------------: |
|       `classes`      | `Union[List[ObjClass], ObjClassCollection]`                    |        Initial list of object classes         |
|     `filterable`     |                            `bool`                              |    Enable search/filter inside the dropdown   |
|     `placeholder`    |                            `str`                               |     Placeholder text when nothing selected    |
| `show_add_new_class` |                            `bool`                              |    Show "Add new class" option in the list    |
|        `size`        |          `Literal["large", "small", "mini", None]`             |                 Selector size                 |
|      `multiple`      |                            `bool`                              |          Enable multiple selection            |
|      `widget_id`     |                            `str`                               |               ID of the widget                |

### classes

List of `ObjClass` instances (or an `ObjClassCollection`) to populate the dropdown.

**type:** `Union[List[ObjClass], ObjClassCollection]`

**default value:** `[]`

```python
class_car = sly.ObjClass("car", sly.Rectangle, color=[255, 0, 0])
class_person = sly.ObjClass("person", sly.Polygon, color=[0, 255, 0])

select_class = SelectClass(classes=[class_car, class_person])
```

### filterable

Enable search/filter functionality in the dropdown.

**type:** `bool`

**default value:** `True`

```python
select_class = SelectClass(classes=[class_car, class_person], filterable=True)
```

### placeholder

Placeholder text shown when no class is selected.

**type:** `str`

**default value:** `"Select class"`

```python
select_class = SelectClass(classes=[class_car, class_person], placeholder="Choose a class")
```

### show\_add\_new\_class

Show the "Add new class" option at the end of the list, which lets the user create a new class directly from the widget.

**type:** `bool`

**default value:** `True`

```python
select_class = SelectClass(classes=[class_car, class_person], show_add_new_class=True)
```

### size

Size of the select dropdown.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_class = SelectClass(classes=[class_car, class_person], size="small")
```

### multiple

Enable selection of multiple classes at once.

**type:** `bool`

**default value:** `False`

```python
select_class = SelectClass(classes=[class_car, class_person], multiple=True)
```

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|     Attributes and Methods    | Description                                                                                          |
| :---------------------------: | ---------------------------------------------------------------------------------------------------- |
|         `get_value()`         | Return the currently selected class name(s) (`str`, `List[str]` if `multiple=True`, or `None`).      |
|     `get_selected_class()`    | Return the currently selected `ObjClass` object(s) (or `List[ObjClass]` if `multiple=True`).         |
|     `set_value(class_name)`   | Set the selected class by name.                                                                      |
|      `get_all_classes()`      | Return a copy of all available `ObjClass` objects.                                                   |
|        `set(classes)`         | Update the list of available classes.                                                                |
|       `@value_changed`        | Decorator; the handler receives the selected `ObjClass` (or list) when the selection changes.        |
|       `@class_created`        | Decorator; the handler receives the newly created `ObjClass` when a new class is created via the UI. |

## Mini App Example

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Card, Container, SelectClass
```

### Prepare object classes

```python
class_car = sly.ObjClass("car", sly.Rectangle, color=[255, 0, 0])
class_person = sly.ObjClass("person", sly.Polygon, color=[0, 255, 0])
```

### Initialize `SelectClass` widget

```python
select_class = SelectClass(
    classes=[class_car, class_person],
    filterable=True,
    show_add_new_class=True,
)
```

### Handle events

```python
@select_class.value_changed
def on_class_selected(selected_class):
    print(f"Selected class: {selected_class.name}")


@select_class.class_created
def on_class_created(new_class: sly.ObjClass):
    print(f"New class created: {new_class.name}")
```

### Create app layout

Prepare a layout for the app using a `Card` widget with the `content` parameter and place the widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Class",
    content=Container(widgets=[select_class]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```
