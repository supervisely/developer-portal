# Text Area

## Introduction

In this tutorial you will learn how to use `TextArea` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/textarea)

## Function signature

```python
TextArea(value=None, placeholder="Please input", rows=2, autosize=True, readonly=False, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/221415130-d049bea9-e49b-4e52-ad98-785acdb96a98.gif)

## Parameters

|  Parameters   |  Type  |                               Description                               |
| :-----------: | :----: | :---------------------------------------------------------------------: |
|    `value`    | `str`  |                              Binding value                              |
| `placeholder` | `str`  | Specifies a short hint that describes the expected value of a text area |
|    `rows`     | `int`  |          Specifies the visible number of lines in a text area           |
|  `autosize`   | `bool` |        Specifies that a text area should automatically get focus        |
|  `readonly`   | `bool` |             Specifies that a text area should be read-only              |
|  `widget_id`  | `str`  |                            Id of the widget                             |

### value

Binding value.

**type:** `str`

**default value:** `None`

```python
input_value = "Some text " * 100
text_area = TextArea(value=input_value)
```

![value](https://user-images.githubusercontent.com/120389559/221416713-16195fef-6c59-46e2-adc0-61b535f5e136.gif)

### placeholder

Specifies a short hint that describes the expected value of a text area.

**type:** `str`

**default value:** `"Please input"`

```python
text_area = TextArea(placeholder="Placeholder data")
```

![placeholder](https://user-images.githubusercontent.com/120389559/221416855-aada5788-eb18-4c97-9a83-048187598ed3.png)

### rows

Specifies the visible number of lines in a text area.

**type:** `int`

**default value:** `2`

```python
input_value = "Some text " * 100
text_area = TextArea(value=input_value, rows=100)
```

![rows](https://user-images.githubusercontent.com/120389559/221416990-f787ba83-6fbd-4e74-a413-050834b28cda.png)

### autosize

Specifies that a text area should automatically get focus.

**type:** `bool`

**default value:** `True`

```python
text_area = TextArea(rows=10, autosize=False)
```

![autosize](https://user-images.githubusercontent.com/120389559/221417225-88043c32-f0ee-4f00-938f-5003690a06b1.png)

### readonly

Specifies that a text area should be read-only.

**type:** `bool`

**default value:** `false`

```python
text_area = TextArea(value="some text", readonly=True)
```

![readonly](https://user-images.githubusercontent.com/120389559/221417587-2430c32b-604a-4bcb-9abd-31eeb95bf0fe.gif)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods  | Description                            |
| :---------------------: | -------------------------------------- |
| `set_value(value: str)` | Set value data.                        |
|      `get_value()`      | Returns input value data.              |
|     `is_readonly()`     | Check `TextArea` is `readonly` or not. |
|   `enable_readonly()`   | Set `readonly` == `True`.              |
|  `disable_readonly()`   | Set `readonly` == `False`.             |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/060_textarea/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/060_textarea/src/main.py)

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
    widgets=[button_random_text, button_clean_input], direction="horizontal"
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

![layout](https://user-images.githubusercontent.com/120389559/221420916-350918f4-223c-4a6b-82cb-2fb2fbbfe5ad.gif)
