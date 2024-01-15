# IFrame

## Introduction

**`IFrame`** widget in Supervisely allows to render `.html` template in the app GUI. This widget is useful for displaying custom html content within an app.

## Function signature

```python
IFrame(
    path_to_html="static/template.html",
    height="500px",
    width="700px",
    widget_id=None,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/fdfaf53a-36a9-4306-9a93-125f11578814)

## Parameters

|   Parameters   |        Type       |                             Description                            |
| :------------: | :---------------: | :----------------------------------------------------------------: |
| `path_to_html` |       `str`       | Path to .html file inside static dir, must not include forward "/" |
|    `height`    | `Union[str, int]` |                     Height of the widget in px                     |
|     `width`    | `Union[str, int]` |                      Width of the widget in px                     |
|   `widget_id`  |       `str`       |                          ID of the widget                          |

### path\_to\_html

Path to .html file inside static dir, must not include forward "/"

**type:** `str`

```python
IFrame(path_to_html="static/template.html")
```

Here is an example of `template.html` file used on the screenshot below:

```html
<div><p>Hello, World</p></div>
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/6793dd09-aa9a-4f08-81dd-b8a5fcc8e6b6)

### height

Height of the widget in px.

**type:** `str`

**default value:** `None`

```python
IFrame(
    path_to_html="static/template.html",
    height="300px"
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/0195cc11-ebcf-4b60-bb25-577efb0f46f5)

### width

Width of the widget in px.

**type:** `str`

**default value:** `None`

```python
IFrame(
    path_to_html="static/template.html",
    width="1000px"
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/0bf9244c-b278-44c6-a4d4-4bfd889a7920)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods   | Description                        |
| :-------------------------: | ---------------------------------- |
|           `set()`           | Set html template to widget.       |
| `set_plot_size(value: str)` | Set widget height and width in px. |
|         `clean_up()`        | Clean html template.               |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/layouts-and-containers/017\_iframe/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/017\_iframe/src/main.py)

### Import libraries

```python
import os
from pathlib import Path
import supervisely as sly
from supervisely.app.widgets import Card, Container, IFrame, Button
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Set path to `static` directory

```python
static_dir = Path("layouts and containers/017_iframe/static")
```

### Initialize `Input` widget

```python
iframe = IFrame("static/index_1.html", height="300px", width="300px")
```

### Create buttons to switch between html templates in `IFrame` widget

```python
button_template_1 = Button("Template 1")
button_template_2 = Button("Template 2")
button_template_3 = Button("Template 3")

button_container = Container(
    widgets=[button_template_1, button_template_2, button_template_3], direction="horizontal"
)
```

### Create app layout

```python
container = Container(widgets=[iframe, button_container])

layout = Card(
    title="IFrame",
    content=container,
)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout, static_dir=static_dir)
```

### Add functions to switch html templates in `IFrame` widget from python code

```python
@button_template_1.click
def set_template_1():
    iframe.set("static/index_1.html", height="300px", width="300px")


@button_template_2.click
def set_template_2():
    iframe.set("static/index_2.html", height="500px", width="500px")


@button_template_3.click
def set_template_3():
    iframe.set("static/index_3.html", height="200px", width="1000px")
```

![miniapp-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/45295f0b-9675-4d60-ad2f-d1890976f4af)
