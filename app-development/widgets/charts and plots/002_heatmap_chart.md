# Heatmap Chart

## Introduction

In this tutorial you will learn how to use `HeatmapChart` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/heatmapchart)

## Function signature

```python
HeatmapChart(title, data_labels=True, xaxis_title=None, color_range="row", tooltip=None)
```

![default](https://user-images.githubusercontent.com/120389559/218247387-621e000a-56ef-4b0a-9900-1ef8cd0ebf38.gif)

## Parameters

|  Parameters   |           Type            |                                Description                                |
| :-----------: | :-----------------------: | :-----------------------------------------------------------------------: |
|    `title`    |           `str`           |                           `HeatmapChart` title                            |
| `data_labels` |          `bool`           | Determines whether the values ​​in the `HeatmapChart` cells are displayed |
| `xaxis_title` |           `str`           |                               `X` axe title                               |
| `color_range` | `Literal["table", "row"]` |            Determines the color distribution on `HeatmapChart`            |
|   `tooltip`   |           `str`           |        Determines the displayed value in the `HeatmapChart` cells         |

### title

Determines `HeatmapChart` title.

**type:** `str`

### data_labels

Determines whether the values ​​in the `HeatmapChart` cells are displayed.

**type:** `bool`

**default value:** `true`

```python
chart = HeatmapChart(
    title="Multiplication Table",
    xaxis_title="",
    data_labels='False'
    color_range="row",
    tooltip="Result multiplication of {x} * {series_name}",
)
```

![data_labels](https://user-images.githubusercontent.com/120389559/218247687-c27fcc47-16ab-40a5-a025-df8766dc5f42.gif)

### xaxis_title

Determines `X` axe title.

**type:** `str`

**default value:** `None`

```python
chart = HeatmapChart(
    title="Multiplication Table",
    xaxis_title="xaxis_title",
    color_range="row",
    tooltip="Result multiplication of {x} * {series_name}",
)
```

![xaxis_title](https://user-images.githubusercontent.com/120389559/218247762-ea5506e9-c029-41cf-b976-d9d80aee8b09.png)

### color_range

Determines the color distribution on `HeatmapChart`.

**type:** `Literal["table", "row"]`

**default value:** `row`

```python
chart = HeatmapChart(
    title="Multiplication Table",
    xaxis_title="xaxis_title",
    color_range="table",
    tooltip="Result multiplication of {x} * {series_name}",
)
```

![color_range](https://user-images.githubusercontent.com/120389559/220913004-f46fa248-3368-4be9-9bb4-0396ebffe56c.png)

### tooltip

Determines the displayed value in the `HeatmapChart` cells.

**type:** `str`

**default value:** `None`

```python
chart = HeatmapChart(
    title="Multiplication Table",
    tooltip="Result multiplication of {x} * {series_name}",
)
```

![tooltip](https://user-images.githubusercontent.com/120389559/218247998-6d6503a6-d5b4-4565-a4de-7aa0e96d52e1.gif)

## Methods and attributes

|                    Attributes and Methods                    | Description                                    |
| :----------------------------------------------------------: | ---------------------------------------------- |
|               `add_series_batch(series: dict)`               | Add series of data in `HeatmapChart` by batch. |
| `add_series(name: str, x: list, y: list, send_changes=True)` | Add series of data in `HeatmapChart`.          |
|                  `get_clicked_datapoint()`                   | Return data clicked in `HeatmapChart`.         |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/030_heatmap_chart/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/030_heatmap_chart/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, HeatmapChart
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize function to build example chart

```python
def multiplication_chart():
    data = []
    for row in list(range(1, 11)):
        temp = [round(row * number, 1) for number in list(range(1, 11))]
        data.append(temp)
    return data
```

### Initialize `HeatmapChart` widget

```python
chart = HeatmapChart(
    title="Multiplication Table",
    xaxis_title="",
    color_range="row",
    tooltip="Result multiplication of {x} * {series_name}",
)
```

### Fill `HeatmapChart` with data

```python
data = multiplication_chart()

lines = [
    {"name": idx + 1, "x": list(range(1, 11)), "y": line}
    for idx, line in enumerate(data)
]

chart.add_series_batch(lines)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Heatmap Chart",
    content=chart,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```
