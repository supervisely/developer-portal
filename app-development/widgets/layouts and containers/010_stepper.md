# Stepper

## Introduction

In this tutorial you will learn how to use `Stepper` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/stepper)

## Function signature

```python
Stepper(titles=[], widgets=[], active_step=1, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/221410742-e51459ab-cfaf-469a-a9d9-392d6edf8800.png)

## Parameters

|  Parameters   |  Type  |          Description          |
| :-----------: | :----: | :---------------------------: |
|   `titles`    | `list` |        Widgets titles         |
|   `widgets`   | `list` | Widgets provided in `Stepper` |
| `active_step` | `int`  |        Set active step        |
|  `widget_id`  | `str`  |       Id of the widget        |

### titles

Determine widgets titles.

**type:** `list`

**default value:** `[]`

```python
text_info = Text(text="My info text", status="info")
card_info = Card(title="Info text", content=text_info)
stepper = Stepper(titles=["Title_1"], widgets=[card_info])
```

![titles](https://user-images.githubusercontent.com/120389559/221411042-67dbd904-411c-4ec6-9b86-dee7319702d6.png)

### widgets

Determine widgets provided in `Stepper`.

**type:** `list`

**default value:** `[]`

```python
text_info = Text(text="My info text", status="info")
text_success = Text(text="My success text", status="success")

card_info = Card(title="Info text", content=text_info)
card_success = Card(title="Success text", content=text_success)

stepper = Stepper(widgets=[card_info, card_success])
```

![widgets](https://user-images.githubusercontent.com/120389559/221411166-a29b6df7-30df-45d7-ab8f-7df3e8d9861d.png)

### active_step

Set active step.

**type:** `int`

**default value:** `1`

```python
text_info = Text(text="My info text", status="info")
text_success = Text(text="My success text", status="success")

card_info = Card(title="Info text", content=text_info)
card_success = Card(title="Success text", content=text_success)

stepper = Stepper(widgets=[card_info, card_success], active_step=2)
```

![active_step](https://user-images.githubusercontent.com/120389559/221411303-13342d2b-a8bd-48fe-8e56-898ed851c639.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods     | Description                |
| :---------------------------: | -------------------------- |
| `set_active_step(value: int)` | Set active step value.     |
|      `get_active_step()`      | Returns active step value. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/059_stepper/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/059_stepper/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, Stepper, Text, Button
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Card` widgets we will use

```python
text_info = Text(text="My info text", status="info")
text_success = Text(text="My success text", status="success")
text_warning = Text(text="My warning text", status="warning")

card_info = Card(title="Info text", content=text_info)
card_success = Card(title="Success text", content=text_success)
card_warning = Card(title="Warning text", content=text_warning)
```

### Initialize `Stepper` widget

```python
stepper = Stepper(
    widgets=[card_info, card_success, card_warning],
    titles=["Text step", "Success step", "Warning step"],
)

card = Card(
    title="Stepper",
    content=stepper,
)
```

### Initialize `Button` widgets we will use to increase and decrease `Stepper` steps

```python
button_increase = Button(text="Increase step")
button_decrease = Button(text="Decrease step")

buttons_container = Container(widgets=[button_increase, button_decrease])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
buttons_card = Card(content=buttons_container)
layout = Container(widgets=[card, buttons_card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Add functions to control widgets from python code

```python
@button_increase.click
def click_button():
    curr_step = stepper.get_active_step()
    curr_step += 1
    stepper.set_active_step(curr_step)


@button_decrease.click
def click_button():
    curr_step = stepper.get_active_step()
    curr_step -= 1
    stepper.set_active_step(curr_step)
```

![layout](https://user-images.githubusercontent.com/120389559/221412710-4ab1a750-3042-4a1f-9cf7-ac278f9a08c4.gif)
