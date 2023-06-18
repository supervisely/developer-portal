# Copy to Clipboard

## Introduction

**`CopyToClipboard`** widget allows you to wrap your widgets (`Editor`, `Text`, `TextArea`, or `Input`) and `str` text with a copy button. This enables you to easily obtain the value of the wrapped content and copy it to your clipboard.

## Function signature

```python
CopyToClipboard(
    content="", 
    widget_id=None
)
```

![default](https://github.com/supervisely/developer-portal/assets/78355358/473d4b89-11c1-4e10-99be-027b4d1f78d7)

## Parameters

|  Parameters |                     Type                    |        Description        |
| :---------: | :-----------------------------------------: | :-----------------------: |
|  `content`  | `Union[Editor, Text, TextArea, Input, str]` | `CopyToClipboard` content |
| `widget_id` |                    `str`                    |      Id of the widget     |

### content

Determine input `CopyToClipboard` content.

**type:** `Union[Editor, Text, TextArea, Input, str]`

**default value:** `""`

```python
copy_to_clipboard = CopyToClipboard(content="Some text to copy")
```

![content](https://github.com/supervisely/developer-portal/assets/78355358/983984a7-3bdd-4567-9c40-987653c01065)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                   |
| :--------------------: | --------------------------------------------- |
|     `get_content()`    | Return wrapped content (i.e. widget or `str`) |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/text elements/004\_copy\_to\_clipboard/src/main.py](https:/github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text%20elements/004\_copy\_to\_clipboard/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Card,
    Container,
    CopyToClipboard,
    Editor,
    Text,
    TextArea,
    Input,
)
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize different inputs

```python
editor = Editor('{ "Editor": 42 }', show_line_numbers=True)
input = Input(value="Input", size="large")
text = Text(text="Text", status="success")
text_area = TextArea(value="TextArea")
string = "Some string"

copytoclipboard1 = CopyToClipboard(content=editor)
copytoclipboard2 = CopyToClipboard(content=input)
copytoclipboard3 = CopyToClipboard(content=text)
copytoclipboard4 = CopyToClipboard(content=text_area)
copytoclipboard5 = CopyToClipboard(content=string)
```

### Create app layout

Prepare a layout for an app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Copy To Clipboard",
    content=Container(
        [
            copytoclipboard1,
            copytoclipboard2,
            copytoclipboard3,
            copytoclipboard4,
            copytoclipboard5,
        ]
    ),
)
layout = Container(widgets=[card], direction="vertical")
```

### Create an app using a layout

Create an app object with a layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://github.com/supervisely/developer-portal/assets/78355358/77c72255-e912-4cba-9fb3-94f695d233e0)
