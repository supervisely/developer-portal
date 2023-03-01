# Switch

## Introduction

In this tutorial you will learn how to use `Switch` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/switch)

## Function signature

```python
Switch(
    switched=False,
    width=58,
    on_text="ON",
    off_text="OFF",
    on_color=None,
    off_color=None,
    on_content=None,
    off_content=None,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/219039067-9798a2ba-2bbc-42f7-94b1-dabb4e6da37b.gif)

## Parameters

|  Parameters   |   Type   |                       Description                       |
| :-----------: | :------: | :-----------------------------------------------------: |
|  `switched`   |  `bool`  |             Determine `Switch` is ON or OFF             |
|    `width`    |  `int`   |                    Width of `Switch`                    |
|   `on_text`   |  `str`   |        Text displayed when `Switch` in ON state         |
|  `off_text`   |  `str`   |        Text displayed when `Switch` in OFF state        |
|  `on_color`   |  `str`   |       Background color when `Switch` in ON state        |
|  `off_color`  |  `str`   |       Background color when `Switch` in OFF state       |
| `on_content`  | `Widget` | Determine active `Widget` when `Switch` is in ON state  |
| `off_content` | `Widget` | Determine active `Widget` when `Switch` is in OFF state |
|  `widget_id`  |  `str`   |                    Id of the widget                     |

### switched

Determine `Switch` is ON or OFF.

**type:** `bool`

**default value:** `false`

```python
switch = Switch(switched=True)
```

![switched](https://user-images.githubusercontent.com/120389559/219045048-d914e087-21b7-4166-9c31-06eb146a2031.png)

### width

Determine `Switch` width.

**type:** `int`

**default value:** `58`

```python
switch = Switch(width=300)
```

![width](https://user-images.githubusercontent.com/120389559/219045541-a6ee235e-ebd3-4aea-9ef8-d851aa4219c6.gif)

### on_text

Determine text displayed when `Switch` in ON state.

**type:** `str`

**default value:** `ON`

### off_text

Determine text displayed when `Switch` in OFF state.

**type:** `str`

**default value:** `OFF`

```python
switch = Switch(on_text="on example", off_text="off example", width=140)
```

![on_off_text](https://user-images.githubusercontent.com/120389559/219046819-c0b57c41-2829-46d5-933e-772a203ea75f.gif)

### on_color

Determine background color when `Switch` in ON state.

**type:** `str`

**default value:** `None`

### off_color

Determine background color when `Switch` in OFF state.

**type:** `str`

**default value:** `None`

```python
switch = Switch(on_color="#FF0000", off_color="#7F00FF")
```

![on_off_color](https://user-images.githubusercontent.com/120389559/219048879-b58edb2c-7074-488d-bcb6-9713480a3fc6.gif)

### on_content

Determine active `Widget` when `Switch` in ON state.

**type:** `Widget`

**default value:** `None`

### off_content

Determine active `Widget` when `Switch` in OFF state.

**type:** `Widget`

**default value:** `None`

```python
switch = Switch(on_content=Text("ON Conent"), off_content=Text("OFF content"))
switch_one_of = OneOf(switch)
```

![on_off_content](https://user-images.githubusercontent.com/120389559/219050482-9b385f5d-b9e1-42a6-922e-c24c0673375d.gif)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|   Attributes and Methods    | Description                            |
| :-------------------------: | -------------------------------------- |
|       `is_switched()`       | Check `Switch` state.                  |
|           `on()`            | Set `Switch` in ON state.              |
|           `off()`           | Set `Switch` in OFF state.             |
|        `get_width()`        | Return `Switch` width.                 |
|       `get_on_text()`       | Return `Switch` ON text.               |
|       `set_on_text()`       | Set `Switch` ON text.                  |
|      `get_off_text()`       | Return `Switch` OFF text.              |
| `set_off_text(value: str)`  | Set `Switch` OFF text.                 |
|      `get_on_color()`       | Return `Switch` ON color.              |
| `set_on_color(value: str)`  | Set `Switch` ON color.                 |
|      `get_off_color()`      | Return `Switch` OFF color.             |
| `set_off_color(value: str)` | Set `Switch` OFF color.                |
|      `value_changed()`      | Handled when `Switch` value is change. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/048_switch/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/048_switch/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Container, Switch, OneOf, Text, Card
```

### Initialize `Switch` widget

```python
switch = Switch(on_content=Text("ON Conent"), off_content=Text("OFF content"))
```

### Initialize `OneOf` widget to switch between `Cards`

```python
switch_one_of = OneOf(switch)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
layout = Container(widgets=[Card(title="Switch", content=switch), Card(title="OneOf", content=switch_one_of)])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/219058398-19aea1df-d3c0-4bc9-ac9b-163c47bdd2b0.gif)
