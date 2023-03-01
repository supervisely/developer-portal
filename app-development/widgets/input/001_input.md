# Input

## Introduction

This Supervisely widget allows you to create input fields for text. It is a useful widget for applications that require users to enter text, such as project name, dataset name or path to folder or archive with data.

The `Input` widget also allows you to set default text to be displayed in the input field, set text placeholder, set input field to be `readonly`, and set a minimum and maximum length for the input.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/input)

## Function signature

```python
Input(value="", minlength=0, maxlength=1000, placeholder="", size=None, readonly=False, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/217822528-c77c3391-8baf-4bab-8480-d38f0071278b.png)

## Parameters

|  Parameters   |                   Type                    |           Description           |
| :-----------: | :---------------------------------------: | :-----------------------------: |
|    `value`    |                   `str`                   |          Binding value          |
|  `minlength`  |                   `int`                   |    Minimum input text length    |
|  `maxlength`  |                   `int`                   |    Maximum input text length    |
| `placeholder` |                   `str`                   |      Placeholder of input       |
|    `size`     | `Literal["mini", "small", "large", None]` |          Size of input          |
|  `readonly`   |                  `bool`                   | Same as readonlyin native input |
|  `widget_id`  |                   `str`                   |        Id of the widget         |

### value

Binding value.

**type:** `str`

**default value:** ""

```python
input = Input(value="Start input value")
```

![value](https://user-images.githubusercontent.com/120389559/217822714-62e392c0-6cfe-4b81-af72-b47a91470a14.png)

### minlength

Minimum input text length.

**type:** `int`

**default value:** `0`

```python
input = Input(minlength=5)
```

### maxlength

Maximum input text length.

**type:** `int`

**default value:** `1000`

```python
input = Input(maxlength=500)
```

### placeholder

Placeholder of input.

**type:** `str`

**default value:** ""

```python
input = Input(placeholder="Please input")
```

![placeholder](https://user-images.githubusercontent.com/120389559/217822992-b29d20f9-5a87-43f1-852d-90bba98151a6.png)

### size

Size of input.

**type:** `Literal["mini", "small", "large", None]`

**default value:** `None`

```python
input = Input()
input_mini = Input(size="mini", placeholder="input mini")
input_small = Input(size="small", placeholder="input small")
input_large = Input(size="large", placeholder="input large")
```

![size](https://user-images.githubusercontent.com/120389559/218092559-c423e7f8-f7cb-4a17-8975-920c2f33a5f8.png)

### readonly

Same as readonly in native input.

**type:** `bool`

**default value:** `false`

```python
input = Input(readonly=True)
```

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                       |
| :--------------------: | ------------------------------------------------- |
|    `is_readonly()`     | Return `True` if input is readonly, else `False`. |
|     `set_value()`      | Set input value.                                  |
|     `get_value()`      | Get input value.                                  |
|  `enable_readonly()`   | Enable input`s readonly property.                 |
|  `disable_readonly()`  | Disable input`s readonly property.                |
|   `value_changed()`    | Handled when input value is changed.              |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/004_input/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/004_input/src/main.py)

### Import libraries

```python
import os
from random import choice

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Input
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Input` widget

```python
input_text = Input(placeholder="Please input")
```

### Create buttons to control `Input` widget values.

```python
button_random_planet = Button(text="Random planet name")
button_clean_input = Button(text="Clean input")
button_set_readonly = Button(text="Set readonly")

buttons_container = Container(
    widgets=[
        button_random_planet,
        button_clean_input,
        button_set_readonly,
    ],
    direction="horizontal",
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Input",
    content=Container(widgets=[input_text, buttons_container]),
)

layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widget from python code

```python
@button_random_planet.click
def random_planet():
    input_text.set_value(
        choice(
            [
                "Mercury",
                "Venus",
                "Earth",
                "Mars",
                "Jupiter",
                "Saturn",
                "Uranus",
                "Neptune",
            ]
        )
    )

@button_clean_input.click
def random_word():
    input_text.set_value("")


@button_set_readonly.click
def set_readonly():
    if input_text.is_readonly():
        input_text.disable_readonly()
        print("Readonly: Disabled")
    else:
        input_text.enable_readonly()
        print("Readonly: Enabled")
```

![layout](https://user-images.githubusercontent.com/120389559/218093184-15b038b9-6b26-4ce1-bc24-64905e4e5c52.gif)
