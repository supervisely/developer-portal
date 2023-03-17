# Button

## Introduction

**`Button`** widget in Supervisely is a user interface element that allows users to create clickable buttons in the appliactions. These buttons can be customized with text, size, colors, and icons, and can be used to trigger specific actions or workflows. The Button widget is a versatile tool that can help users automate repetitive tasks or streamline their workflows, making it a valuable addition to any project dashboard.

## Function signature

```python
Button(
    text="Button",
    button_type="primary",
    button_size=None,
    plain=False,
    show_loading=True,
    icon=None,
    icon_gap=5,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222425389-2eaf1f4a-0bc6-4593-bee1-0698df506775.png" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters   |                                 Type                                 |                                     Description                                    |
| :------------: | :------------------------------------------------------------------: | :--------------------------------------------------------------------------------: |
|     `text`     |                                 `str`                                |                     Determine text that displayed on the button                    |
|  `button_type` | `Literal["primary", "info", "warning", "danger", "success", "text"]` |                                     Button type                                    |
|  `button_size` |               `Literal["mini", "small", "large", None]`              |                                     Button size                                    |
|     `plain`    |                                `bool`                                |                     Determine whether button is a plain button                     |
| `show_loading` |                                `bool`                                |        If `True` display loading animation when `loading` property is `True`       |
|     `icon`     |                                 `str`                                | Button icon, accepts an icon name of icon from Material Design Iconic Font library |
|   `icon_gap`   |                                 `int`                                |                          Gap between icon and button text                          |
|   `widget_id`  |                                 `str`                                |                                  ID of the widget                                  |

### text

Determine text that will be displayed on the button.

**type:** `str`

**default value:** `Button`

```python
button = Button(text="My text on the button")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222425551-f4ba5a79-4fc6-48b9-96ac-40ac01b6c655.png" alt=""><figcaption></figcaption></figure>

### button\_type

Button type.

**type:** `Literal["primary", "info", "warning", "danger", "success", "text"]`

**default value:** `"primary"`

```python
button_primary = Button(text="primary", button_type="primary")
button_info = Button(text="info", button_type="info")
button_warning = Button(text="warning", button_type="warning")
button_danger = Button(text="danger", button_type="danger")
button_success = Button(text="success", button_type="success")
button_text = Button(text="text", button_type="text")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222425939-dc3dab0a-4944-4ccb-9423-0914810eb0dd.png" alt=""><figcaption></figcaption></figure>

### button\_size

Button size.

**type:** `Literal["mini", "small", "large", None]`

**default value:** `None`

```python
button_mini = Button(text="mini", button_size="mini")
button_small = Button(text="small", button_size="small")
button_large = Button(text="large", button_size="large")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222426825-c1c48184-40fd-4766-8ec0-f49c0a466230.png" alt=""><figcaption></figcaption></figure>

### plain

Determine whether button is a plain button.

**type:** `bool`

**default value:** `False`

```python
button_plain = Button(plain=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222426997-fd108303-a4dd-4529-bbcf-e0a3431b7c5e.png" alt=""><figcaption></figcaption></figure>

### show\_loading

Determine whether button is loading.

**type:** `bool`

**default value:** `False`

```python
button_1 = Button(show_loading=False)

button_2 = Button(show_loading=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222428715-422d966b-9bc4-40d4-a59f-29351651869b.gif" alt=""><figcaption></figcaption></figure>

### icon

Button icon, accepts an icon name of icon component. Icons can be found at [zavoloklom.github.io](http://zavoloklom.github.io/material-design-iconic-font/icons.html).

Open any icon at [zavoloklom.github.io](http://zavoloklom.github.io/material-design-iconic-font/icons.html) and copy it's name (see example below).

<figure><img src="https://user-images.githubusercontent.com/79905215/222429824-54cf5044-6083-453b-964a-438469824e63.png" alt=""><figcaption></figcaption></figure>

**type:** `str`

**default value:** `None`

```python
button_icon = Button(text="icon", icon="zmdi zmdi-play")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222430124-297575bb-4031-487d-bf81-dc0af58c1fe7.png" alt=""><figcaption></figcaption></figure>

### icon\_gap

Gap between icon and button text.

**type:** `int`

**default value:** `5`

```python
button_icon_gap_5 = Button(text="icon gap 5", icon="zmdi zmdi-play", icon_gap=5)
button_icon_gap_50 = Button(text="icon gap 50", icon="zmdi zmdi-play", icon_gap=50)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222430331-6534e93f-0978-482c-8e96-8b49987e898a.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/controls/001\_button/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/001\_button/src/main.py)

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Text
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize `Button` and `Text` widgets

```python
button_add = Button(text="Add", icon="zmdi zmdi-plus-1")
button_subtract = Button(text="Subtract", icon="zmdi zmdi-neg-1")
text_num = Text(text="0", status="text")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place 3 widgets that we've just created in the `Container` widget. Place order in the `Container` is important, we want buttons to be displayed above `Text` widget.

```python
card = Card(
    "Button",
    description="👉 Click on the button to add or subtract 1",
    content=Container([button_add, button_subtract, text_num]),
)
layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready. It's time to handle button clicks.

### Handle button clicks

Use the decorator as shown below to handle button click. We have 2 buttons, one for adding and another one for subtracting

```python
@button_add.click
def add():
    text_num.text = str(int(text_num.text) + 1)

@button_subtract.click
def subtract():
    text_num.text = str(int(text_num.text) - 1)
```

<figure><img src="https://user-images.githubusercontent.com/48913536/202197687-5b2b4cdc-66ef-4d88-b47e-48a1556bd56e.gif" alt=""><figcaption></figcaption></figure>
