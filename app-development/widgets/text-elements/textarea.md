# TextArea

## Introduction

**`TextArea`** widget in Supervisely is a widget that allows users to enter and edit multiple lines of text. The widget provides a large text input area that can be customized with various options, such as placeholder text, autosize or read-only properties, and default value. `TextArea` widget is often used for collecting longer form input from users, such as descriptions or comments.

## Function signature

```python
TextArea(
    value=None,
    placeholder="Please input",
    rows=2,
    autosize=True,
    readonly=False,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223040758-1a68dd01-56fa-4269-8d50-fc5036ea7883.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters  |  Type  |                               Description                               |
| :-----------: | :----: | :---------------------------------------------------------------------: |
|    `value`    |  `str` |                            Widgets text value                           |
| `placeholder` |  `str` | Specifies a short hint that describes the expected value of a text area |
|     `rows`    |  `int` |           Specifies the visible number of lines in a text area          |
|   `autosize`  | `bool` |        Specifies that a text area should automatically get focus        |
|   `readonly`  | `bool` |              Specifies that a text area should be read-only             |
|  `widget_id`  |  `str` |                             ID of the widget                            |

### value

Widgets text value

**type:** `str`

**default value:** `None`

```python
input_value = "Some text " * 100
text_area = TextArea(value=input_value)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223040715-985f1cb2-fe65-4455-b480-129959ab57f5.gif" alt=""><figcaption></figcaption></figure>

### placeholder

Specifies a short hint that describes the expected value of a text area.

**type:** `str`

**default value:** `"Please input"`

```python
text_area = TextArea(placeholder="Placeholder data")
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221416855-aada5788-eb18-4c97-9a83-048187598ed3.png" alt=""><figcaption></figcaption></figure>

### rows

Specifies the visible number of lines in a text area.

**type:** `int`

**default value:** `2`

```python
input_value = "Some text " * 100
text_area = TextArea(value=input_value, rows=100)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221416990-f787ba83-6fbd-4e74-a413-050834b28cda.png" alt=""><figcaption></figcaption></figure>

### autosize

Specifies that a text area should automatically get focus.

**type:** `bool`

**default value:** `True`

```python
text_area = TextArea(rows=10, autosize=False)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221417225-88043c32-f0ee-4f00-938f-5003690a06b1.png" alt=""><figcaption></figcaption></figure>

### readonly

Specifies that a text area should be read-only.

**type:** `bool`

**default value:** `false`

```python
text_area = TextArea(value="some text", readonly=True)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221417587-2430c32b-604a-4bcb-9abd-31eeb95bf0fe.gif" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|  Attributes and Methods | Description                            |
| :---------------------: | -------------------------------------- |
| `set_value(value: str)` | Set value data.                        |
|      `get_value()`      | Returns input value data.              |
|     `is_readonly()`     | Check `TextArea` is `readonly` or not. |
|   `enable_readonly()`   | Set `readonly` == `True`.              |
|   `disable_readonly()`  | Set `readonly` == `False`.             |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/text elements/002\_textarea/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text%20elements/002\_textarea/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, TextArea, Button
from dotenv import load_dotenv
import random
import string
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Button` widgets we will use

```python
button_random_text = Button(text="Generate random text")
button_clean_input = Button(text="Clean input")
buttons_container = Container(
    widgets=[button_random_text, button_clean_input],
    direction="horizontal",
)
```

### Initialize `TextArea` widget

```python
text_area = TextArea()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="TextArea",
    content=Container(widgets=[text_area, buttons_container]),
)

layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Add functions to control widgets from python code

```python
@button_random_text.click
def random_text():
    random_text = "".join(
        random.choice(string.ascii_uppercase + string.digits) for _ in range(1000)
    )
    text_area.set_value(value=random_text)


@button_clean_input.click
def clear():
    text_area.set_value(value="")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223043298-a13fc216-6d1b-4330-8325-dceed63df23d.gif" alt=""><figcaption></figcaption></figure>
