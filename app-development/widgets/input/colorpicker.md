# ColorPicker

## ColorPicker

## Introduction

**`ColorPicker`** is a color selector supporting multiple color formats.

## Function signature

```python
ColorPicker(
    show_alpha=False,
    color_format="hex",
    widget_id=None,
)
```

![color\_picker\_default](https://user-images.githubusercontent.com/120389559/225931304-f021f9fb-2e38-4c40-b8cd-a0aab88027eb.gif)

## Parameters

|   Parameters   |                  Type                 |             Description             |
| :------------: | :-----------------------------------: | :---------------------------------: |
|  `show_alpha`  |                 `bool`                | Whether to display the alpha slider |
| `color_format` | `Literal["hex", "hsl", "hsv", "rgb"]` |             Color format            |
|    `compact`   |                 `bool`                |         Make widget compact         |
|   `widget_id`  |                 `str`                 |           ID of the widget          |

### show\_alpha

Determine whether to display the alpha slider.

**type:** `bool`

**default value:** `False`

```python
color_picker = ColorPicker(show_alpha=True)
```

![show\_alpha](https://user-images.githubusercontent.com/120389559/225931910-07a8cb48-bf44-4cfb-bdb5-1f5dfcc898c1.gif)

### color\_format

Determine color format(hsl, hsv, hex, rgb).

**type:** `Literal["hex", "hsl", "hsv", "rgb"]`

**default value:** `hex`

```python
color_picker = ColorPicker(color_format="hsl")
```

![color\_format](https://user-images.githubusercontent.com/120389559/225932593-d332ad51-b3dd-4a20-96ab-25cf26918095.gif)

### compact

Make widget compact.

**type:** `bool`

**default value:** `False`

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/c5fd68b1-5dd2-45bf-8da7-4d07da947b01)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|   Attributes and Methods  | Description                                |
| :-----------------------: | ------------------------------------------ |
|       `get_value()`       | Return `ColorPicker` current color         |
|       `set_value()`       | Set `ColorPicker` color                    |
| `is_show_alpha_enabled()` | Check if `show_alpha` is `True` or `False` |
|   `enable_show_alpha()`   | Enable `show_alpha`                        |
|   `disable_show_alpha()`  | Disable `show_alpha`                       |
|    `get_color_format()`   | Return `ColorPicker` color format          |
|    `set_color_format()`   | Set `ColorPicker` color format             |
|     `value_changed()`     | Handle color change event                  |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/input/007\_color\_picker/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/input/007\_color\_picker/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, ColorPicker, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize ColorPicker and Text widgets

```python
color_picker = ColorPicker(show_alpha=False, color_format="hex", compact=False)
color_info = Text("Current color: #20A0FF", "info")
text = Text("Hello, World!", color="#20A0FF", font_size=22)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Card(
    "Color Picker",
    content=Container([color_picker, color_info, text]),
)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/a8b5e878-3f5c-4e7c-9f2c-e2a4b57c5a9b)
