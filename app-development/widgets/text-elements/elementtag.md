# ElementTag

## Introduction

**`ElementTag`** widget in Supervisely is a widget that allows users to display [elements tag](https://element.eleme.io/1.4/#/en-US/component/tag) in the UI.

### Function signature

```python
ElementTag(text="", type=None, hit=False, color="", widget_id=None)
```

## Parameters

|  Parameters |                                Type                                |                  Description                  |
| :---------: | :----------------------------------------------------------------: | :-------------------------------------------: |
|    `text`   |                                `str`                               |               `ElementTag` text               |
|    `type`   | `Literal["primary", "gray", "success", "warning", "danger", None]` |               `ElementTag` theme              |
|    `hit`    |                               `bool`                               | Whether `ElementTag` has a highlighted border |
|   `color`   |                                `str`                               |      Background color of the `ElementTag`     |
| `widget_id` |                                `str`                               |                ID of the widget               |

### text

Determine `ElementTag` text.

**type:** `str`

```python
el_tag = ElementTag(text="Tag example")
```

![](https://user-images.githubusercontent.com/120389559/226908793-2a620b84-0b72-4231-8639-ce1f3a458f89.png)

### type

Determine `ElementTag` theme.

**type:** `Literal["primary", "gray", "success", "warning", "danger", None]`

**default value:** `None`

```python
el_tag = ElementTag(text="Tag example", type="success")
```

![](https://user-images.githubusercontent.com/120389559/226909285-8ad976b9-e16a-4ebc-b0f4-f76e9b7fb7c2.png)

### hit

Determine whether `ElementTag` has a highlighted border.

**type:** `bool`

**default value:** `False`

```python
el_tag = ElementTag(text="Tag example", hit=True, type="success")
```

![](https://user-images.githubusercontent.com/120389559/226909880-f21382df-01de-42fc-9c0d-81dad8522e7c.png)

### color

Determine background color of the `ElementTag`.

**type:** `str`

**default value:** `""`

```python
el_tag = ElementTag(text="Tag example", color="#E414D7", type="success")
```

![](https://user-images.githubusercontent.com/120389559/226910422-d0dc98ec-40da-4c11-adb3-69feff45e57a.png)

### widget\_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

|  Attributes and Methods | Description                   |
| :---------------------: | ----------------------------- |
|  `set_text(value: str)` | Set `ElementTag` text.        |
|       `get_text()`      | Return `ElementTag` text.     |
|       `set_type()`      | Set `ElementTag` type.        |
|       `get_type()`      | Return `ElementTag` type.     |
|       `set_hit()`       | Set `ElementTag` `hit` value. |
|       `get_hit()`       | Get `hit` value.              |
|      `get_color()`      | Return `ElementTag` color.    |
| `set_color(value: str)` | Set `ElementTag` color.       |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/text elements/008\_element\_tag/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text%20elements/008\_element\_tag/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, ElementTag
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `ElementTag` widgets

```python
el_tag = ElementTag(text="Tag")

all_tag_types = [el_tag]
for tag_type in ["primary", "gray", "success", "warning", "danger"]:
    curr_tag = ElementTag(text=f"Tag {tag_type}", type=tag_type)
    all_tag_types.append(curr_tag)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    "ElementTag",
    content=Container(widgets=all_tag_types, direction="horizontal"),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![](https://user-images.githubusercontent.com/120389559/226914574-394c3629-4816-42c8-8a5a-82aec34239ad.png)
