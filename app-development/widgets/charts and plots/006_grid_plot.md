# Grid Plot

## Introduction

In this tutorial you will learn how to use `GridPlot` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/gridplot)

## Function signature

```python
GridPlot(data, columns=1, gap=10, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/221510629-ac9d6f3d-cf11-450c-b145-1e493a8d7707.png)

## Parameters

| Parameters  |        Type         |              Description              |
| :---------: | :-----------------: | :-----------------------------------: |
|   `data`    | `List[Dict or str]` | List if data to display on `GridPlot` |
|  `columns`  |        `int`        |    Number of columns on `GridPlot`    |
|    `gap`    |        `int`        |   Gap between widgets on `GridPlot`   |
| `widget_id` |        `str`        |           Id of the widget            |

### data

List if data to display on `GridPlot`.

**type:** `List[Dict or str]`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_plot = GridPlot(data=[data_max, data_denis])
```

![data](https://user-images.githubusercontent.com/120389559/221512143-719f1aa2-ced7-4d26-90b0-1152638d4bcf.png)

### columns

Determine number of columns on `GridPlot`.

**type:** `int`

**default value:** `1`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_plot = GridPlot(data=[data_max, data_denis], columns=2)
```

![columns](https://user-images.githubusercontent.com/120389559/221512647-45437a89-0a1d-4dd3-aab5-737d22a78c4e.png)

### gap

Determine gap between widgets on `GridPlot`.

**type:** `int`

**default value:** `10`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_plot = GridPlot(data=[data_max, data_denis], columns=2, gap=100)
```

![gap](https://user-images.githubusercontent.com/120389559/221513120-18ce7fe6-231c-4f96-bb9c-49e054198b50.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|               Attributes and Methods                | Description                       |
| :-------------------------------------------------: | --------------------------------- |
|         `add_scalar(dentifier: str, y, x)`          | Add data in `GridPlot`.           |
| `add_scalars(plot_title: str, new_values: dict, x)` | Add series of data in `GridPlot`. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/061_grid_plot/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/061_grid_plot/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, GridPlot
from dotenv import load_dotenv
import numpy as np
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare series for plot

```python
size1 = 10
x1 = list(range(size1))
y1 = np.random.randint(low=10, high=148, size=size1).tolist()
s1 = [{"x": x, "y": y} for x, y in zip(x1, y1)]

size2 = 30
x2 = list(range(size2))
y2 = np.random.randint(low=0, high=300, size=size2).tolist()
s2 = [{"x": x, "y": y} for x, y in zip(x2, y2)]

data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}

data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

data_all = {
    "title": "Max vs Denis",
    "series": [{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}]}
```

### Initialize `GridPlot` widget

```python
grid_plot = GridPlot(data=[data_max, data_denis, data_all], columns=2)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="GridPlot",
    content=grid_plot,
)

layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/221514239-467ba551-97b9-42d5-bccf-459121d9d5d3.png)
