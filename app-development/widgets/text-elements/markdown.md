# Markdown

## Introduction

The `Markdown` widget allows you to easily add and style your text using Markdown syntax. By using Markdown, you can add headings, lists, links, images, and more to enhance the presentation of your text content.

## Function signature

```python
Markdown(
    content="",
    height=300,
    widget_id=None,
)
```

![Markdown preview](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/08e3e736-690e-4a9a-bb41-fe44061b5a73)

## Parameters

| Parameters  |                 Type                 |       Description       |
| :---------: | :----------------------------------: | :---------------------: |
|  `content`  |                `str`                 | `Markdown` content text |
|  `height`   | `Union[int, Literal["fit-content"]]` |     `Widget` height     |
| `widget_id` |                `str`                 |    ID of the widget     |

### content

Determine input `Markdown` content.

**type:** `str`

**default value:** `""`

```python
markdown = Markdown(content="Some content")
```

![content](https://user-images.githubusercontent.com/120389559/224316855-4dd7d72a-3818-44f5-bc74-7a83ac1a82ab.png)

### height

Determine `Widget` height.

**type:** `int`

**default value:** `300`

```python
markdown = Markdown(content="Some content", height=30)
```

![height](https://user-images.githubusercontent.com/120389559/224317474-e94ef7c0-39f2-42db-b7d6-5a105d84e11b.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|   Attributes and Methods    | Description               |
| :-------------------------: | ------------------------- |
| `set_content(content: str)` | Set `Markdown` data.      |
|       `get_content()`       | Return `Markdown` data.   |
|       `get_height()`        | Return `Markdown` height. |
|  `set_height(height: int)`  | Set `Markdown` height.    |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/text-elements/005_markdown/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text%20elements/005\_markdown/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Flexbox, Markdown, Button
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Button` widgets, we will use

```python
button_text = Button(text="Set md text")
button_clean = Button(text="Clean md")
button_readme = Button(text="Set md readme")
buttons_container = Flexbox(widgets=[button_text, button_clean, button_readme])
```

### Initialize `Markdown` widget

```python
md_path = "path/to/README.md"
md = ""
with open(md_path, "r") as f:
    md = f.read()

markdown = Markdown(height=400)
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(title="Markdown", content=Container([markdown, buttons_container]))
layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

### Handle button clicks

```python
@button_text.click
def simple_content():
    markdown.set_content("### Title \n *some markdown text*")


@button_clean.click
def clear_content():
    markdown.set_content("")


@button_readme.click
def readme_content():
    markdown.set_content(md)
```

![layout](https://user-images.githubusercontent.com/120389559/224319059-a601a2a4-fc67-4551-bf22-df3b621f9260.gif)
