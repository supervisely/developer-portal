# Docstring

## Introduction

In this tutorial, you will learn how to use the `Docstring` widget in the Supervisely app. The `Docstring` widget allows you to display a given docstring as HTML-formatted content.

[Read this tutorial in the developer portal.](https://developer.supervisely.com/app-development/apps-with-gui/docstring)

## Function signature

```python
Docstring(
    content="",
    is_html=False,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/224303776-85b03aec-bfdd-45e8-be9c-4cbc598fdfa0.png)

## Parameters

| Parameters  |  Type  |                     Description                     |
| :---------: | :----: | :-------------------------------------------------: |
|  `content`  | `str`  |                   `HTML` content                    |
|  `is_html`  | `bool` | Determine if the given content has an `HTML` format |
| `widget_id` | `str`  |                  ID of the widget                   |

### content

Determine input `HTML` content.

**type:** `str`

**default value:** `""`

```python
docstring = Docstring(content="Some content")
```

![content](https://user-images.githubusercontent.com/120389559/224304307-2e222fd3-0430-4b1b-abec-6e8996df64c5.png)

### is_html

Determine if the given content has an `HTML` format.

**type:** `bool`

**default value:** `False`

```python
docstring = Docstring(content="<h2>Some title</h2><p>some text</p>", is_html=True)
```

![is_html](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/075be32a-8db7-45bb-bbe4-e800268f5391)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|           Attributes and Methods           | Description                |
| :----------------------------------------: | -------------------------- |
| `set_content(content: str, is_html: bool)` | Set `Docstring` content    |
|              `get_content()`               | Return `Docstring` content |
|                 `clear()`                  | Clear `Docstring` content  |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/misc/docstring/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/misc/docstring/src/main.py)

### Import libraries

```python
import os

from dotenv import load_dotenv

import supervisely as sly
from supervisely.api.image_api import ImageApi  # we will use ImageApi class docstring as examples
from supervisely.app.widgets import Card, Container, Docstring
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare contents for `Docstring` widget

**You can use 2 types of content:**

- raw docstring as a `str` type

```python
docstring_example_1 = ImageApi.__doc__
```

- html format string

```py
docstring_example_2 = """<html>
<head>
<style>
table, th, td { border: 1px solid black; border-collapse: collapse; }
th, td { padding: 5px; }
th { text-align: left; }
</style>
</head>
<body>

<table>
  <tr> <th>Class name</th> <th>Images count</th> <th>Labels count</th> </tr>
  <tr> <td>Lemon</td> <td>6</td> <td>6</td> </tr>
  <tr> <td>Kiwi</td> <td>6</td> <td>20</td> </tr>
</table>
</body>
</html>"""
```

### Initialize `Docstring` widgets and set content types and values

```python
docstring_1 = Docstring(content=docstring_example_1)

docstring_2 = Docstring()
docstring_2.set_content(content=docstring_example_2, is_html=True)
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card_1 = Card(title="Docstring example 1", content=docstring_1)
card_2 = Card(title="Docstring example 2", content=docstring_2)

layout = Container(widgets=[card_1, card_2])
```

### Create app using layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/12865031-2274-426c-98fa-48bf21de5941)
