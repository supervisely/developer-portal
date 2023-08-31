# Icons

## Introduction

In this tutorial you will learn how to use `Icons` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/icons)

## Function signature

```python
Icons(class_name=None, color=None, bg_color=None, rounded=False, image_url=None, widget_id=None)
```

![icons_default](https://user-images.githubusercontent.com/120389559/225037687-cd58165f-6464-418f-9db6-229d8b113f4e.png)

## Parameters

|  Parameters  |  Type  |                Description                |
| :----------: | :----: | :---------------------------------------: |
| `class_name` | `str`  | Icon class name (e.g "zmdi zmdi-image-o") |
|   `color`    | `str`  |                Icon color                 |
|  `bg_color`  | `str`  |           Icon background color           |
|  `rounded`   | `bool` |                Round icon                 |
| `image_url`  | `str`  |              Icon image url               |
| `widget_id`  | `str`  |             Id of the widget              |

### class_name

Determines icon class name. Icons can be found at [zavoloklom.github.io](http://zavoloklom.github.io/material-design-iconic-font/icons.html).

**type:** `str`

**default value:** `None`

```python
icon = Icons(class_name="zmdi zmdi-bike")
```

![class_name](https://user-images.githubusercontent.com/120389559/225039036-991b1dd3-c348-4145-bf54-74f49187b183.png)

### color

Determines icon color.

**type:** `str`

**default value:** `None`

```python
icon = Icons(class_name="zmdi zmdi-bike", color="#000000")
```

![color](https://user-images.githubusercontent.com/120389559/225039748-2fdaa29e-11b0-4f72-a8b7-e2209a323e74.png)

### bg_color

Determines icon background color.

**type:** `str`

**default value:** `None`

```python
icon = Icons(class_name="zmdi zmdi-bike", bg_color="#000000")
```

![bg_color](https://user-images.githubusercontent.com/120389559/225040501-777765a3-43a7-4988-a87e-9ce6fa91a397.png)

### rounded

Round icon.

**type:** `bool`

**default value:** `False`

```python
icon = Icons(class_name="zmdi zmdi-bike", rounded=True)
```

![rounded](https://user-images.githubusercontent.com/120389559/225041354-aacce923-06df-4240-af25-9ee47588001f.png)

### image_url

Icon image url.

**type:** `str`

**default value:** `None`

```python
icon = Icons(image_url="https://i.imgur.com/0E8d8bB.png")
```

![image_url](https://user-images.githubusercontent.com/120389559/225041975-0c8ab153-93c6-4092-9c2a-b0271ae78893.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods    | Description                      |
| :--------------------------: | -------------------------------- |
| `set_class_name(value: str)` | Set `Icons` class name.          |
|      `get_class_name()`      | Return `Icons` class name.       |
|   `set_color(value: str)`    | Set `Icons` color.               |
|        `get_color()`         | Return `Icons` color.            |
|  `set_bg_color(value: str)`  | Set `Icons` background color.    |
|       `get_bg_color()`       | Return `Icons` background color. |
|       `set_rounded()`        | Set `Icons` round.               |
|       `set_standart()`       | Set `Icons` standart.            |
|        `get_rouded()`        | Check `Icons` round or not.      |
| `set_image_url(value: str)`  | Set `Icons` image url.           |
|      `get_image_url()`       | Return `Icons` image url.        |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/008_icons/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/008_icons/src/main.py)

### Import libraries

```python
import os

from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Button, Card, Container, Icons
```

### Initialize `Button` we will use

```python
button_url = Button("Set url")
button_round = Button("Round icon")
button_standart = Button("Standart icon")
button_add_bg = Button("Add bg")
button_start = Button("Start icon")
btn_container = Container(
    [button_round, button_standart, button_add_bg, button_url, button_start], direction="horizontal"
)
```

### Initialize `Icons` widget

```python
icon_name = "zmdi zmdi-car-taxi"
icon = Icons(class_name=icon_name)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Icons",
    content=Container([icon, btn_container]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widget from python code

```python
@button_url.click
def set_url():
    icon.set_image_url("https://i.imgur.com/0E8d8bB.png")


@button_round.click
def set_round():
    icon.set_rounded()


@button_standart.click
def set_standart():
    icon.set_standart()


@button_add_bg.click
def set_bg():
    icon.set_bg_color("#000000")


@button_start.click
def start():
    icon.set_image_url("")
    icon.set_standart()
    icon.set_bg_color("")
    icon.set_class_name(icon_name)
```

![layout](https://user-images.githubusercontent.com/120389559/225051826-93bb6e64-2cd8-4184-9fab-8441baec10af.gif)
