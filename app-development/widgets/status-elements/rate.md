# Rate

## Introduction

**`Rate`** widget in Supervisely that allows users to provide a rating using a graphical interface. It provides a customizable rating scale and supports features like disabling the widget, allowing half ratings, and displaying text labels.

## Function signature

```python
Rate(
    value=None,
    max=5,
    disabled=False,
    allow_half=False,
    texts=[],
    show_text=False,
    text_color="#1F2D3D",
    text_template="",
    colors=["#F7BA2A", "#F7BA2A", "#F7BA2A"],
    void_color="#C6D1DE",
    disabled_void_color="#EFF2F7",
    widget_id=None,
)
```

![rate_default](https://user-images.githubusercontent.com/120389559/225903196-c2d60aa6-529a-4874-aac0-41571d52135c.gif)

## Parameters

|      Parameters       |        Type         |                  Description                  |
| :-------------------: | :-----------------: | :-------------------------------------------: |
|        `value`        | `Union[int, float]` |                 Rating score                  |
|         `max`         |        `int`        |               Max rating score                |
|      `disabled`       |       `bool`        |          Whether `Rate` is read-only          |
|     `allow_half`      |       `bool`        |     Whether picking half start is allowed     |
|        `texts`        |     `List[str]`     |                  Text array                   |
|      `show_text`      |       `bool`        |           Whether to display texts            |
|     `text_color`      |        `str`        |                Color of texts                 |
|    `text_template`    |        `str`        | Text template when the component is read-only |
|       `colors`        |     `List[str]`     |             Color array for icons             |
|     `void_color`      |        `str`        |           Color of unselected icons           |
| `disabled_void_color` |        `str`        |      Color of unselected read-only icons      |
|      `widget_id`      |        `str`        |               ID of the widget                |

### max

Determine max rating score.

**type:** `int`

**default value:** `5`

```python
rate = Rate(max=15)
```

![max](https://user-images.githubusercontent.com/120389559/225904319-6b017a51-456d-4c12-bff6-b8467bc94630.png)

### colors

Determine color array for icons. It should have 3 elements, each of which corresponds with a score level

**type:** `List[str]`

**default value:** `["#F7BA2A", "#F7BA2A", "#F7BA2A"]`

```python
rate = Rate(colors=["#1414E4", "#2DE414", "#F7BA2A"])
```

![colors](https://user-images.githubusercontent.com/120389559/225905073-7c529057-7e7b-4617-8f2d-de17ea9252de.gif)

### disabled

Determine whether `Rate` is read-only.

**type:** `bool`

**default value:** `False`

```python
rate = Rate(disabled=True)
```

![disabled](https://user-images.githubusercontent.com/120389559/225905381-5b5455f9-b45e-4b0f-9b44-511445431981.png)

### allow_half

Determine whether picking half start is allowed.

**type:** `bool`

**default value:** `False`

```python
rate = Rate(allow_half=True)
```

![allow_half](https://user-images.githubusercontent.com/120389559/225905792-bc5fa368-abb0-41f7-b40a-938b7e5f08a5.gif)

### texts

Determine text array for each star. Available if `show_text` is `True`

**type:** `List[str]`

**default value:** `[]`

### show_text

Determine whether to display texts.

**type:** `bool`

**default value:** `False`

```python
infos = ["oops", "disappointed", "normal", "good", "great"]
rate = Rate(texts=infos, show_text=True)
```

![texts](https://user-images.githubusercontent.com/120389559/225906480-ce45b8ab-6af4-4444-b279-68691a6e98cd.gif)

### text_color

Determine color of texts.

**type:** `str`

**default value:** `"#1F2D3D"`

```python
infos = ["oops", "disappointed", "normal", "good", "great"]
rate = Rate(texts=infos, show_text=True, text_color="#E414BB")
```

![text_color](https://user-images.githubusercontent.com/120389559/225906932-fcb00268-f55a-4762-bf8e-2fe254609f01.png)

### void_color

Determine color of unselected icons.

**type:** `str`

**default value:** `"#C6D1DE"`

```python
rate = Rate(void_color="#1459E4")
```

![void_color](https://user-images.githubusercontent.com/120389559/225907283-ab4c02a6-5acb-4538-8f84-b50dd5bfe6a4.png)

### disabled_void_color

Determine color of unselected read-only icons.

**type:** `str`

**default value:** `"#C6D1DE"`

```python
rate = Rate(disabled_void_color="#5D6D7E", disabled=True)
```

![disabled_void_color](https://user-images.githubusercontent.com/120389559/225908159-b285ef4a-910a-410b-b3ca-2b970ae7d408.png)

## Methods and attributes

|        Attributes and Methods         | Description                                                 |
| :-----------------------------------: | ----------------------------------------------------------- |
|             `is_disabled`             | Property return `True` if `Rate` is read-only.              |
|             `get_value()`             | Get `Rate` score value.                                     |
| `set_value(value: Union[int, float])` | Set `Rate` score value.                                     |
|           `get_max_value()`           | Get max rating score.                                       |
|      `set_max_value(value: int)`      | Set max rating score.                                       |
|            `get_colors()`             | Get color array for icons.                                  |
|            `set_colors()`             | Set color array for icons.                                  |
|              `disable()`              | Enable rate`s read-only property.                           |
|              `enable()`               | Disable rate`s read-only property.                          |
|       `allow_half_precision()`        | Enable picking half star.                                   |
|      `disallow_half_precision()`      | Disable picking half star.                                  |
|             `get_texts()`             | Return text array for each star.                            |
|     `set_texts(value: List[str])`     | Set text for each star.                                     |
|             `show_text()`             | Enable displaying texts.                                    |
|             `hide_text()`             | Disable displaying texts.                                   |
|          `get_text_color()`           | Get color of texts.                                         |
|     `set_text_color(value: str)`      | Set color of texts.                                         |
|          `get_void_color()`           | Get color of unselected icons.                              |
|     `set_void_color(value: str)`      | Set color of unselected icons.                              |
|      `get_disabled_void_color()`      | Get color of unselected read-only icons.                    |
| `set_disabled_void_color(value: str)` | Set color of unselected read-only icons.                    |
|         `value_changed(func)`         | Decorator function is handled when input `Rate` is changed. |

## Mini App Example

You can find this examples in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/status-elements/008\_rate/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/status%20elements/008\_rate/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Field, Input, InputNumber, Rate, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

**In this guide, let's look at 3 examples of using the widget `Rate` in the one app.**

### Example 1

**Initialize `Rate` widget**

```python
infos = ["oops", "disappointed", "normal", "good", "great"]
colors = ["#656565", "#F7BA2A", "#F6568E"]
rate = Rate(
    text_color="#656565",
    disabled=True,
    value=4,
    show_text=True,
    colors=colors,
    max=5,
    text_template="points",
)
```

**Prepare `InputNumber`, `Button` and `Card` widgets we will use in this example**

```python
btn = Button("Enable / Disable")

card = Card("Disabled rate", content=Container([rate, btn]))
```

**Add functions to control widgets from python code**

```python
@btn.click
def enable_btn_click():
    rate.enable() if rate.is_disabled else rate.disable()
```

**Create app**

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Container(widgets=[card])
```

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<p align="center">
  <img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/2a7ea6bf-e5d2-404f-ae9f-0e67bf606306" alt="layout" />
</p>

### Example 2

**Initialize `Rate` widget**

```python
colors = ["#656565", "#F7BA2A", "#F6568E"]
rate = Rate(
    text_color="#656565",
    allow_half=True,
    value=4.2,
    show_text=True,
    colors=colors,
)
```

**Prepare `InputNumber`, `Button` and `Card` widgets we will use in this example**

```python
btn = Button("Set value")
input_num = InputNumber(value=rate.get_value(), min=0, max=5, step=0.1)

card = Card("Set rating", content=Container([rate, input_num, btn]))
```

**Add functions to control widgets from python code**

```python
@btn.click
def set_value_btn_click():
    new_value = input_num.get_value()
    rate.set_value(new_value)
```

**Create app**

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Container(widgets=[card])
```

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<p align="center">
  <img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/0f9db62b-f3b3-4df4-bc43-df113e51fbeb" alt="layout" />
</p>

### Example 3

**Initialize `Rate` widget**

```python
infos = ["oops", "disappointed", "normal", "good", "great"]
colors = ["#656565", "#F7BA2A", "#F6568E"]
rate = Rate(
    texts=infos,
    text_color="#656565",
    show_text=True,
    colors=colors,
    void_color="#F6568E",
)
```

**Prepare `Input`, `Field`, `Text`, `Button` and `Card` widgets we will use in this example**

```python
text_input = Input()
text_field = Field(
    content=text_input,
    title="Enter texts for rate",
    description="Enter 5 string values separated by commas",
)
text = Text(status="text")
btn = Button("Set texts")


card = Card(
    "Rate",
    content=Container([rate, text, text_field, btn]),
)
```

**Add functions to control widgets from python code**

```python
@rate.value_changed
def rate_changed(value):
    text.text = f"Rate value: {value} stars"


@btn.click
def set_texts_btn_click():
    text = text_input.get_value()
    if len(text.split()) != 5:
        return
    rate.set_texts(text.split(", "))
```

**Create app**

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Container(widgets=[card])
```

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<p align="center">
  <img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/e1660e25-7051-445c-bd36-f08c86a1080a" alt="layout" />
</p>
