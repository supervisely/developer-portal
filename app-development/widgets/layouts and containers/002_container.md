# Container

## Introduction

**`Container`** widget in Supervisely is a flexible tool that allows for organizing other widgets within it. It can be used to group related widgets, change the layout of widgets, and adjust the sizing of widgets. `Container` widget supports both horizontal and vertical layout options. 

However, `Container` widget does not have any specific functionality on its own but serves as a wrapper for other widgets.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/container)

## Function signature

```python
container = Container(widgets=[button_1, button_2])
```

## Parameters

| Parameters  |                Type                 |                 Description                  |
| :---------: | :---------------------------------: | :------------------------------------------: |
|  `widgets`  |           `List[Widget]`            |               List of widgets                |
| `direction` | `Literal["vertical", "horizontal"]` | Container direction (vertical or horizontal) |
|    `gap`    |                `int`                |       Gap between widgets in container       |
| `fractions` |             `List[int]`             |      Fractions for container splitting       |
| `widget_id` |                `str`                |                  Widget ID                   |

### widgets

List of widgets

**type:** `List[Widget]`

**default** `[]`

```python
container = Container(widgets=[Input(), Input()])
```

![container](https://user-images.githubusercontent.com/79905215/220125712-c4c98ba6-9cbb-4a6f-944b-335056d59536.png)

### direction

Container direction (vertical or horizontal)

**type:** `Literal["vertical", "horizontal"]`

**default** `"vertical"`

```python
container = Container(
    widgets=[Input(), Input(), Input()],
    direction="horizontal",
)
```

![container-direction](https://user-images.githubusercontent.com/79905215/220126696-8fe7d789-05e1-4dff-8f9d-274c872a0d3b.png)

### gap

Gap between widgets in container

**type:** `int`

**default** `10`

```python
container = Container(widgets=[Input(), Input(), Input()], gap=25)
```

![container-gap](https://user-images.githubusercontent.com/79905215/220127050-fa283570-2fce-4f92-9599-9c21e83fdcaf.png)

### fractions

Fractions for container splitting.
`direction` parameter have to be `horizontal`

**type:** `List[int]`

**default** `None`

```python
container = Container(
    widgets=[Input(), Input(), Input()],
    direction="horizontal",
    fractions=[3, 2, 5],
)
```

![container-fractions](https://user-images.githubusercontent.com/79905215/220127504-4f8ceee2-83f7-40b9-976e-f865d442da86.png)

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/034_container/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/034_container/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Input
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare widgets for `Container` widget

```python
button = Button("Button")

input_1 = Input()
input_2 = Input()
input_3 = Input()
```

### Initialize `Container` widget

```python
inputs_container = Container(
    widgets=[input_1, input_2, input_3],
    direction="horizontal",
    fractions=[3, 2, 5],
    gap=20,
)
```

```python
container = Container(widgets=[inputs_container, button])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
layout = Card(title="Container card", content=container)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```

![container-app](https://user-images.githubusercontent.com/79905215/220128472-5e9de449-a11a-468f-9580-14eb60390db7.png)
