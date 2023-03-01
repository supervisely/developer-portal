# Line Plot

## Introduction

In this tutorial you will learn how to use `LinePlot` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/lineplot)

## Function signature

```python
LinePlot(
    title,
    series=[],
    smoothing_weight=0,
    group_key=None,
    show_legend=True,
    decimals_in_float=2,
    xaxis_decimals_in_float=None,
    yaxis_interval=None,
    widget_id=None,
    yaxis_autorescale=True,
)
```

![default](https://user-images.githubusercontent.com/120389559/219958867-13bb604e-d890-475d-ab46-8b41063620ba.png)

## Parameters

|        Parameters         |  Type  |                         Description                          |
| :-----------------------: | :----: | :----------------------------------------------------------: |
|          `title`          | `str`  |                       `LinePlot` title                       |
|         `series`          | `list` |                  List of input data series                   |
|    `smoothing_weight`     | `int`  |                          Smoothing                           |
|        `group_key`        | `str`  |                      Synced charts key                       |
|       `show_legend`       | `bool` |                  Show legend on `LinePlot`                   |
|    `decimals_in_float`    | `int`  | The number of fractions to display floating values in y-axis |
| `xaxis_decimals_in_float` | `int`  | The number of fractions to display floating values in x-axis |
|     `yaxis_interval`      | `list` |          Min and max values on y axis (e.g. [0, 1])          |
|        `widget_id`        | `str`  |                       Id of the widget                       |
|    `yaxis_autorescale`    | `bool` |                Set autoscaling of the Y axis                 |

### title

Determine `LinePlot` title.

**type:** `str`

```python
line_chart = LinePlot(title="Max vs Denis")
```

![title](https://user-images.githubusercontent.com/120389559/219959818-7f392fac-7d2e-4bfc-b54e-850dd8f3d75f.png)

### series

Determine list of input data series.

**type:** `list`

**default value:** `[]`

```python
size1 = 10
x1 = list(range(size1))
y1 = np.random.randint(low=10, high=148, size=size1).tolist()
s1 = [{"x": x, "y": y} for x, y in zip(x1, y1)]

size2 = 30
x2 = list(range(size2))
y2 = np.random.randint(low=0, high=300, size=size2).tolist()
s2 = [{"x": x, "y": y} for x, y in zip(x2, y2)]

line_chart = LinePlot(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
)
```

![series](https://user-images.githubusercontent.com/120389559/220658262-3d4af3e6-70cd-4b02-9a75-18046d4bc401.png)

### smoothing_weight

Determine `LinePlot` smoothing.

**type:** `int`

**default value:** `0`

### group_key

Synced `LinePlot` key.

**type:** `str`

**default value:** `None`

### show_legend

Determine showing legend on `LinePlot`.

**type:** `bool`

**default value:** `True`

```python
line_chart = LinePlot(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    show_legend=False,
)
```

![show_legend](https://user-images.githubusercontent.com/120389559/220660068-1801f5ad-fc21-4ab5-88e4-6f30a5d0a0e4.png)

### decimals_in_float

The number of fractions to display floating values in y-axis.

**type:** `int`

**default value:** `2`

### xaxis_decimals_in_float

The number of fractions to display floating values in x-axis.

**type:** `int`

**default value:** `None`

```python
line_chart = LinePlot(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    xaxis_decimals_in_float=44,
)
```

![xaxis_decimals_in_float](https://user-images.githubusercontent.com/120389559/220661506-646cc687-bc27-445d-8f05-0b3a2b2efccf.png)

### yaxis_interval

Determine min and max values on y axis (e.g. [0, 1]).

**type:** `list`

**default value:** `None`

```python
line_chart = LinePlot(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    yaxis_interval=[0, 200],
)
```

![yaxis_interval](https://user-images.githubusercontent.com/120389559/220662241-f6e86fd9-0410-4e8f-bd18-859e5dfc6dc9.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

### yaxis_autorescale

Set autoscaling of the Y axis.

**type:** `bool`

**default value:** `True`

## Methods and attributes

|                        Attributes and Methods                        | Description                                    |
| :------------------------------------------------------------------: | ---------------------------------------------- |
|                `update_y_range(ymin: int, ymax: int)`                | Update `LinePlot` data.                        |
| `add_series(name: str, x: list, y: list, send_changes: bool = True)` | Add new series of data in `LinePlot`.          |
|                   `add_series_batch(series: list)`                   | Add new series of data in `LinePlot` by batch. |
|         `add_to_series(name_or_id: str or int, data: dict)`          | Add data to exist series.                      |
|                   `get_series_by_name(name: str)`                    | Return series data by name.                    |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/053_line_plot/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/053_line_plot/src/main.py)

### Import libraries

```python
import os
import numpy as np
import supervisely as sly
from supervisely.app.widgets import Card, Container, LinePlot
from dotenv import load_dotenv
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
```

### Initialize `LinePlot` widget

```python
line_chart = LinePlot(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    show_legend=False,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Line Plot",
    content=line_chart,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/220869621-7e055e46-a81d-4c92-b263-d5ce8e2cc5b5.gif)
