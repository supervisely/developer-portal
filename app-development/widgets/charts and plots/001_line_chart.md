# Line Chart

## Introduction

**`Linechart`** is a Supervisely widget that allows for visualizing data as a line chart. It supports data in pandas dataframe format or a Python list of dictionaries with a specific structure.

The widget allows for customization of the chart title, axis titles, and color scheme. `Linechart` also supports zooming, panning, and downloading the chart as png, svg, or csv. Additionally, it can detect clicks on data points and respond to them through Python code

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/linechart)

## Function signature

```python
size1 = 10
x1 = list(range(size1))
y1 = np.random.randint(low=10, high=148, size=size1).tolist()
s1 = [{"x": x, "y": y} for x, y in zip(x1, y1)]

size2 = 30
x2 = list(range(size2))
y2 = np.random.randint(low=0, high=300, size=size2).tolist()
s2 = [{"x": x, "y": y} for x, y in zip(x2, y2)]

line_chart = LineChart(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    xaxis_type="category",
)
```

![linechart](https://user-images.githubusercontent.com/79905215/220094045-325e098e-bbe6-4f2a-bb6c-c8f494a203b7.png)

## Parameters

|     Parameters      |                     Type                     |                          Description                           |
| :-----------------: | :------------------------------------------: | :------------------------------------------------------------: |
|       `title`       |                    `str`                     |                        Line chart title                        |
|      `series`       |                    `list`                    |  List of series including names and lists of X, Y coordinates  |
|       `zoom`        |                    `bool`                    |                   Enable zoom on `Linechart`                   |
|   `stroke_curve`    |       `Literal["smooth", "straight"]`        |               Set line type (straight or curved)               |
|   `stroke_width`    |                    `int`                     |                         Set line width                         |
|   `markers_size`    |                    `int`                     |                     Set point markers size                     |
|    `data_labels`    |                    `bool`                    | If `True` it will display `Y` value of data for each datapoint |
|    `xaxis_type`     | `Literal["numeric", "category", "datetime"]` |                Set type of divisions on X axis                 |
|    `xaxis_title`    |                    `str`                     |                    Set title for the X axis                    |
| `yaxis_autorescale` |                    `bool`                    |                 Set autoscaling of the Y axis                  |
|      `height`       |              `Union[int, str]`               |                         Widget height                          |
|  `decimalsInFloat`  |                    `int`                     |       Set number of decimals in float values of `Y` axis       |

### title

Line chart title

**type:** `str`

```python
line_chart = LineChart(
    title="Linechart title",
    series=[{"name": "Max", "data": s1}],
    xaxis_type="numeric",
)
```

![linechart-title](https://user-images.githubusercontent.com/79905215/220101483-306341e9-ff5f-4fa6-879f-2550ea5583c1.png)

### series

List of series including names and lists of X, Y coordinates

**type:** `list`

```python
line_chart = LineChart(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    xaxis_type="numeric",
)
```

![linechart-series](https://user-images.githubusercontent.com/79905215/220101465-c7c8f978-bfe7-434e-a2ae-34959fa56781.png)

### zoom

Enable zoom on `Linechart`

**type:** `bool`

**default** `False`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    zoom=True
)
```

![linechart-zoom](https://user-images.githubusercontent.com/79905215/220102374-effacf49-6e2e-434e-ae71-a783de5c98bf.gif)

### stroke_curve

Set line type (straight or curved)

**type:** `Literal["smooth", "straight"]`

**default** `"smooth"`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    stroke_curve="straight"
)
```

![linechart-curve](https://user-images.githubusercontent.com/79905215/220103062-6e897455-2334-490d-a6a7-e1b04b387641.png)

### stroke_width

Set line width

**type:** `int`

**default** `2`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    stroke_width=5
)
```

![linechart-width](https://user-images.githubusercontent.com/79905215/220103604-657a32b1-6c46-40cf-8af3-60dc83514d0f.png)

### markers_size

Set point markers size

**type:** `int`

**default** `4`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    markers_size=8
)
```

![linechart-markers](https://user-images.githubusercontent.com/79905215/220104341-0a8c3ee5-4f38-45ff-9f91-ea9a5058592e.png)

### data_labels

If `True` it will display `Y` value of data for each

**type:** `bool`

**default** `False`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    data_labels=True
)
```

![linechart-labels](https://user-images.githubusercontent.com/79905215/220105299-b274a3ae-dd03-4929-bde0-40afc5f295ed.png)

### xaxis_type

Set type of divisions on X axis

**type:** `Literal["numeric", "category", "datetime"]`

**default** `numeric`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    xaxis_type="numeric",
)
```

![linechart-xtype](https://user-images.githubusercontent.com/79905215/220106075-5075c4eb-fbd7-4cd7-96c8-baf560152099.png)

### xaxis_title

Set title for the X axis

**type:** `str`

**default** `None`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    xaxis_type="category",
    xaxis_title="days",
)
```

![linechart-xtitle](https://user-images.githubusercontent.com/79905215/220106662-0f213a1e-4959-4762-980e-99d1fdbb02b5.png)

### yaxis_autorescale

Set autoscaling of the Y axis

**type:** `bool`

**default** `True`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    yaxis_autorescale=False
)
```

![linechart-yautoscale](https://user-images.githubusercontent.com/79905215/220107447-72ba2861-ebda-49de-8a3d-bf2de9f33fd5.png)

### height

Widget height

**type:** `Union[int, str]`

**default** `350`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    height=150
)
```

![linechart-height](https://user-images.githubusercontent.com/79905215/220109141-9f903cb5-2d3b-4a00-a605-709b88689fc8.png)

### decimalsInFloat

Set number of decimals in float values of `Y` axis

**type:** `int`

**default** `2`

```python
line_chart = LineChart(
    title="Title",
    series=[{"name": "Max", "data": s1}],
    decimalsInFloat=1
)
```

![linechart-decfloat](https://user-images.githubusercontent.com/79905215/220111027-e661f232-c02e-4ee5-bbd5-69b08337606f.png)

## Methods and attributes

|                  Attributes and Methods                   | Description                               |
| :-------------------------------------------------------: | ----------------------------------------- |
| `update_y_range(ymin: int, ymax: int, send_changes=True)` | Update chart `Y` axis range.           |
|                   `get_clicked_value()`                   | Get value of clicked datapoint.           |
|                 `get_clicked_datapoint()`                 | Get clicked datapoint.                    |
|    `set_title(title: str, send_changes: bool = True)`     | Set chart title.                          |
|         `add_series(name: str, x: list, y: list)`         | Add new series to chart.                  |
|             `add_series_batch(series: dict)`              | Add batch of series to chart.                  |
|       `set_series(series: list, description: str)`        | Set series to chart.                      |
|       `set_colors(colors: list, description: str)`        | Set colors for series in chart.           |
|                         `@click`                          | Decorator function to handle chart click. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/032_line_chart/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/032_line_chart/src/main.py)

### Import libraries

```python
import os

import numpy as np
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, LineChart, Table
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare series for chart

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

### Initialize `LineChart` widget

```python
line_chart = LineChart(
    title="Max vs Denis",
    series=[{"name": "Max", "data": s1}, {"name": "Denis", "data": s2}],
    xaxis_type="category",
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
row_id = 1
columns = ["id", "Line", "x", "y"]
result_table = Table(data=[], columns=columns, width="25%")


container = Container(widgets=[line_chart, result_table])
card = Card(title="Line Chart", content=container)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control widgets from python code

```python
@line_chart.click
def add_row_to_table(datapoint: sly.app.widgets.LineChart.ClickedDataPoint):
    global row_id
    row = [row_id, datapoint.series_name, datapoint.x, datapoint.y]
    row_id += 1
    result_table.insert_row(row)
```

![linechart-app](https://user-images.githubusercontent.com/79905215/220120213-8debc158-5cb7-46ce-91b0-49a73695c081.gif)
