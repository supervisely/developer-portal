# Destination Project

## Introduction

**`Badge`** widget in Supervisely is a versatile tool for displaying notifications or counts on elements such as buttons, text. With customizable types (number, text, or dot), visibility, the Badge can be easily used in Supervisely apps for a seamless user experience.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/misc/badge)

## Function signature

```python
Badge(
    value: Union[int, str, float] = None,
    widget: Optional[Widget] = None,
    max: int = None,
    is_dot: bool = False,
    hidden: bool = False,
    widget_id: str = None,
)
```

![default](https://user-images.githubusercontent.com/79905215/227223981-643400c7-0c16-4c5e-acc9-d2c2214162d1.png)

## Parameters

| Parameters  |           Type           |                             Description                             |
| :---------: | :----------------------: | :-----------------------------------------------------------------: |
|   `value`   | `Union[int, str, float]` |                     Badge widget content value                      |
|  `widget`   |    `Optional[Widget]`    |               Determine a widget to content in badge                |
|    `max`    |          `int`           | Determine max value of badge content. Value type has to be a number |
|  `is_dot`   |          `bool`          |             Specifies that badge is displayed as a dot              |
|  `hidden`   |          `bool`          |               Specifies that a badge widget is hidden               |
| `widget_id` |          `str`           |                          ID of the widget                           |

#### value

Badge widget content value

**type:** `Union[int, str, float]`

**default value:** `None`

```python
badge = Badge("value")
```

![value](https://user-images.githubusercontent.com/79905215/227200691-47e12a73-bfb0-4808-ac5d-d45d6730cdd6.png)

#### widget

Determine a widget to content in badge

**type:** `Optional[Widget]`

**default value:** `None`

```python
button = Button()
badge = Badge(widget=button)
```

![widget](https://user-images.githubusercontent.com/79905215/227201103-29ea0ca5-f313-461b-aa22-7787863ae839.png)

#### max

Determine max value of badge content. Value type has to be a number

**type:** `int`

**default value:** `None`

```python
button = Button()
badge = Badge(
    widget=button,
    value=10,
    max=5,
)
```

![max](https://user-images.githubusercontent.com/79905215/227208678-1c30b7a2-d725-4af6-aa31-724747e86188.png)

#### is_dot

Specifies that badge is displayed as a dot

**type:** `bool`

**default value:** `False`

```python
button = Button()
badge = Badge(
    widget=button,
    is_dot=True,
)
```

![isdot](https://user-images.githubusercontent.com/79905215/227210594-8ac4d3bf-5880-42da-92ca-5d0df7d4d3f2.png)

#### hidden

Specifies that a badge widget is hidden

**type:** `bool`

**default value:** `False`

```python
button = Button()
badge = Badge(
    widget=button,
    hidden=True,
)
```

![hidden](https://user-images.githubusercontent.com/79905215/227217503-976e358a-f383-4ac5-bed8-8508a61c84b5.png)

#### widget_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

|           Attributes and Methods           | Description                          |
| :----------------------------------------: | ------------------------------------ |
| `set_value(value: Union[str, int, float])` | Set badge value                      |
|               `get_value()`                | Get badge value                      |
|                 `clear()`                  | Clear badge value                    |
|               `hide_badge()`               | Hide badge on widget                 |
|               `show_badge()`               | Show badge on widget                 |
|           `toggle_visibility()`            | Toggle visibility of badge on widget |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/misc/badge/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/misc/badge/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Badge, Button, Checkbox, Container, Card, Input, InputNumber
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `Button`, `Checkbox`, `Input`, `InputNumber` widgets to contain in `Badge` widgets

```python
button_1 = Button("Hide badge")
text_input = Input(placeholder="Enter value")

button_2 = Button("Hide badge")
number_input = InputNumber(min=1, max=100)

button_3 = Button("Toggle visibility")
checkbox = Checkbox("Show badge")
```

### Create `Badge` widgets we will use in UI for demo

```python
text_badge = Badge(widget=button_1, value="new")

number_badge = Badge(widget=button_2, value=1, max=10)

dot_badge = Badge(widget=button_3, is_dot=True)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
text_container = Container([text_badge, text_input])

number_container = Container([number_badge, number_input])

dot_container = Container([dot_badge, checkbox])

container = Container(
    widgets=[text_container, number_container, dot_container],
    direction="horizontal",
)
card = Card(content=container)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from code

```python
@text_input.value_changed
def set_badge_text(value):
    text_badge.set_value(value)


@number_input.value_changed
def set_badge_number(value):
    number_badge.set_value(value)


@checkbox.value_changed
def change_badge_visibility(value):
    dot_badge.toggle_visibility()


@button_1.click
def change_badge_visibility():
    text_badge.hide_badge()


@button_2.click
def change_badge_visibility():
    number_badge.hide_badge()


@button_3.click
def change_badge_visibility():
    dot_badge.toggle_visibility()
```

<p align="center">
    <img src="https://user-images.githubusercontent.com/79905215/227223140-8c7d481a-40c5-4b34-ace7-10f7d5147881.gif" alt="layout">
</p>
