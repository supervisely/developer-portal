# Card

## Introduction

**`Card`** widget in Supervisely is a simple widget that can be used to display information or content in a compact format. 
It can be controlled by setting `loading`/`lock` properties and can be collapsed to save space. It provides a straightforward and easy-to-use solution for displaying information clearly and concisely.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/card)

## Function signature

```python
card = Card(content=Text("some text"))
```

![card-def](https://user-images.githubusercontent.com/79905215/220175796-e2b904d2-9dee-4243-bd73-2aa78e592954.png)

## Parameters

|     Parameters      |   Type   |                         Description                         |
| :-----------------: | :------: | :---------------------------------------------------------: |
|       `title`       |  `str`   |                      Card widget title                      |
|    `description`    |  `str`   |              Description text for card widget               |
|    `collapsable`    |  `bool`  | Enable `collapsable` property to allow minimize card widget |
|      `content`      | `Widget` |               Widget to place in Card widget                |
| `content_top_right` | `Widget` |     Widget to place in top right corner of Card widget      |
|   `lock_message`    |  `str`   |         Message to display when card will be locked         |
|     `widget_id`     |  `str`   |                          Widget ID                          |

### title

Card widget title

**type:** `str`

**default** `None`

```python
card = Card(title="Card title")
```

![card-title](https://user-images.githubusercontent.com/79905215/220181030-f2da22e8-0fa5-4383-ae38-7977a4969750.png)

### description

Description text for card widget

**type:** `str`

**default** `None`

```python
card = Card(description="card description")
```

![card-description](https://user-images.githubusercontent.com/79905215/220180856-0640f01f-71ef-422e-a864-76f7abbbe98e.png)

### collapsable

Enable `collapsable` property to allow minimize card widget

**type:** `bool`

**default** `False`

```python
card = Card(
    title="Card title",
    description="card description",
    content=Text("some text"),
    collapsable=True
)
```

![card-collapse](https://user-images.githubusercontent.com/79905215/220180426-9533f12c-87b0-445b-9456-d2cc37cc8ebc.gif)

### content

Widget to place in Card widget

**type:** `Widget`

**default** `None`

```python
card = Card(
    title="Card title",
    description="card description",
    content=Container(widgets=[Text("some text"), Button("Button")])
)
```

![card-content](https://user-images.githubusercontent.com/79905215/220179379-df3e2b0c-a85f-4e7f-b9c2-ee1c52c5a7ab.png)

### content_top_right

Widget to place in top right corner of Card widget

**type:** `Widget`

**default** `None`

```python
card = Card(
    title="Card title",
    description="card description",
    content=Text("some text"),
    content_top_right=Button("Button"),
)
```

![card-top-right](https://user-images.githubusercontent.com/79905215/220179093-c6f870f6-8c5f-4cb2-ac08-28a0d51efd63.png)

### lock_message

Message to display when card will be locked

**type:** `str`

**default** `"Card content is locked"`

```python
card = Card(content=Text("some text"), lock_message='Press "UNLOCK" button to unlock the card.')
```

![card-lock-message](https://user-images.githubusercontent.com/79905215/220178542-f589a5f4-5ffc-437d-b0ed-5863a6d8d64b.png)

### widget_id

Widget ID

**type:** `str`

**default** `None`

## Methods and attributes

| Attributes and Methods | Description                               |
| :--------------------: | ----------------------------------------- |
|      `collapse()`      | Minimize card widget.                     |
|     `uncollapse()`     | Expand card widget.                       |
|        `lock()`        | Lock card widget and show message.        |
|       `unlock()`       | Unlock card widget and hide lock message. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/035_card/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/035_card/src/main.py)

### Import libraries

```python
import os
import random

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Image
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare widgets we will use in `Card` widget

Buttons to enable loading property, lock and collapse `Card` widget:

```python
loading_btn = Button("Loading True")
lock_btn = Button("Lock")
collapse_btn = Button("Collapse")
```

Buttons to unlock, uncollapse and disable loading property of `Card` widget:

```python
unlock_btn = Button("Unlock", button_type="success")
uncollapse_btn = Button("Uncollapse", button_type="success")
refresh_btn = Button("Loading False", button_type="success")
```

Use `Container` widget to join `Button` widgets in groups.

```python
btn_container_1 = Container(widgets=[loading_btn, refresh_btn])
btn_container_2 = Container(widgets=[lock_btn, unlock_btn])
btn_container_3 = Container(widgets=[collapse_btn, uncollapse_btn])

containers = Container(
    widgets=[btn_container_1, btn_container_2, btn_container_3],
    direction="horizontal",
)
```

Prepare widgets to display some image.

```python
preview_btn = Button("Preview")
image = Image()
```

### Initialize `Card` widgets

One `Card` widget for buttons

```python
buttons_card = Card(content=containers)
```

Another `Card` widget for previewing images.

```python
image_card = Card(
    title="Card",
    description="Card Description",
    content=image,
    collapsable=True,
    content_top_right=preview_btn,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Container(
    widgets=[buttons_card, image_card],
    direction="horizontal",
    fractions=[1, 1],
)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control widgets from python code

```python
@unlock_btn.click
def lock_card():
    image_card.unlock()


@uncollapse_btn.click
def lock_card():
    image_card.uncollapse()


@refresh_btn.click
def lock_card():
    image_card.loading = False


@lock_btn.click
def lock_card():
    image_card.lock()


@collapse_btn.click
def lock_card():
    image_card.collapse()


@loading_btn.click
def lock_card():
    image_card.loading = True


@preview_btn.click
def load_preview_image():
    image_card.loading = True
    image.set(image_info.full_storage_url)
    image_card.loading = False
```

![card-app](https://user-images.githubusercontent.com/79905215/220270565-861b5a71-238e-4a30-8152-1df5811d8e6c.gif)
