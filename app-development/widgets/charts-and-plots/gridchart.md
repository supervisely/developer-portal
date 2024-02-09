# GridСhart

## Introduction

**`GridChart`** widget is used to create a grid of `LineChart` subplots with a given data. Users can add plots in real-time making it useful for interactive data exploration and analysis. This widget could be considered as an advanced version of a `GridPlot` widget with the support of [ApexCharts](https://apexcharts.com/) library.

## Function signature

```python
data_1 = {
    "title": "Line 1",
    "series": [{"name": "Line 1", "data": s1}],
}

data_2 = {
    "title": "Line 2",
    "series": [{"name": "Line 2", "data": s2}],
}

data_all = {
    "title": "All lines",
    "series": [{"name": "Line 1", "data": s1}, {"name": "Line 2", "data": s2}],
}

grid_chart = GridChart(data=[data_1, data_2, data_all], columns=3)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/78355358/624a6587-b2cc-452e-ad31-6516e2485e95" alt=""><figcaption></figcaption></figure>

## Parameters


| Parameters |        Type        |              Description              |
| :-----------: | :-------------------: | :--------------------------------------: |
|   `data`   | `List[dict or str]` |        List of data to display. `str` inputs will be recognized as a titles for empty widgets.        |
|  `columns`  |        `int`        |           Number of columns.           |
|    `gap`    |        `int`        | Gap between widgets inside `GridChart`. |
| `widget_id` |        `str`        |            ID of the widget            |

### data

List of data to display on `GridPlot`.

**type:** `List[Dict or str]`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_chart = GridChart(data=[data_max, data_denis])
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/78355358/c837a424-26fa-415f-8304-0a8b3b326dc3" alt="" width=500><figcaption></figcaption></figure>

### columns

Determine the number of columns on `GridChart`.

**type:** `int`

**default value:** `1`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_chart = GridChart(data=[data_max, data_denis], columns=2)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/78355358/e2e5ad0b-0031-4cde-8474-b3478d3beb04" alt=""><figcaption></figcaption></figure>

### gap

Determine the gap between widgets on `GridChart`.

**type:** `int`

**default value:** `10`

```python
data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}
data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

grid_chart = GridChart(data=[data_max, data_denis], columns=2, gap=100)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/78355358/f0e99122-b4c9-4882-917c-b3857ffc5d73" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes


|               Attributes and Methods               | Description                       |
| :---------------------------------------------------: | ----------------------------------- |
|         `add_scalar(identifier: str, y, x)`         | Add data in `GridChart`.          |
| `add_scalars(plot_title: str, new_values: dict, x)` | Add series of data in `GridChart`. |

## Mini App Example

You can find this example in our GitHub repository:

[ui-widgets-demos/charts and plots/010_grid_chart/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/charts%20and%20plots/010_grid_chart/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, GridChart
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
s2 = [(x, y) for x, y in zip(x2, y2)]

data_max = {"title": "Max", "series": [{"name": "Max", "data": s1}]}

data_denis = {"title": "Denis", "series": [{"name": "Denis", "data": s2}]}

data_all = {
    "title": "Max vs Denis",
    "series": [{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}]
}
```

### Initialize `GridChart` widget

```python
grid_chart = GridChart(data=[data_max, data_denis, data_all], columns=2)
```

### Create app layout

Prepare a layout for an app using a `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="GridChart",
    content=grid_chart,
)

layout = Container(widgets=[card], direction="vertical")
```

### Create an app using a layout

Create an app object with a layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/78355358/6295494f-149d-47a5-bbd5-20e5e664ad54" alt=""><figcaption></figcaption></figure>
