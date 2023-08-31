# TreemapChart

## Introduction

**`TreemapChart`** widget in Supervisely is a widget used for displaying a treemap chart. It allows users to visualize data for comparison distribution of different objects. The TreemapChart widget allows easily visualize data to determine the distribution of objects in comparison to each other.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/charts-and-plots/TreemapChart)

## Function signature

```python
TreemapChart(
    title="Treemap Chart",
    colors=None,
    tooltip=None,
)
```

![default](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/242254345-8fc4eccd-b04b-489a-89ee-4569de6a2624.png)

## Parameters

| Parameters |    Type     |                       Description                        |
| :--------: | :---------: | :------------------------------------------------------: |
|  `title`   |    `str`    |                   `TreemapChart` title                   |
|  `colors`  | `List[str]` | Determines colors for cells in series in `TreemapChart`  |
| `tooltip`  |    `str`    | Determines tooltip for cells in series in `TreemapChart` |

### title

Determines `TreemapChart` title.

**type:** `str`

### colors

Determines colors for cells in series in `TreemapChart`.
The colors should be in `hex` format (e.g. `#ff0000`).

**type:** `List[str]`

**default value:** `None`

```python
colors = ["#ff0000", "#00ff00", "#0000ff"]
chart = TreemapChart(
    title="Treemap Chart",
    colors=colors,
)
```

### tooltip

Determines the tooltip for cells in series in `TreemapChart`.
The tooltip should be in `str` format and may contain `{x}` and `{y}` placeholders.
The name of the cell will be shown instead of `{x}` and the value of the cell will be shown instead of `{y}`.
If not specified, the default tooltip will be used (e.g. `name: value`)

**type:** `str`

**default value:** `None`

```python
tooltip = "This is the name: {x}, this is the value: {y}"
chart = TreemapChart(
    title="Treemap Chart",
    tooltip=tooltip,
)
```

## Methods and attributes

|                     Methods and attributes                      |                                 Description                                 |
| :-------------------------------------------------------------: | :-------------------------------------------------------------------------: |
| `add_series(names: List[str], values: List[Union[int, float]])` |               Adds a new series to the `TreemapChart` widget.               |
| `set_series(names: List[str], values: List[Union[int, float]])` |  Sets a series to the `TreemapChart` widget, removing all previous series.  |
|                    `get_series(index: int))`                    |          Returns a series from the `TreemapChart` widget by index.          |
|                   `delete_series(index: int)`                   |          Deletes a series from the `TreemapChart` widget by index.          |
|                    `get_clicked_datapoint()`                    | Returns the clicked datapoint from the `TreemapChart` widget as namedtuple. |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/charts-and-plots/008_treemap_chart/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/charts-and-plots/008_treemap_chart/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, TreemapChart, Container, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize TreemapChart widget

```python
# Colors are optional. If not specified, the default colors will be used.
colors = [
    "#008FFB",
    "#00E396",
    "#FEB019",]

# Tooltip is optional. If not specified, the default tooltip will be used.
tooltip = "This is the name: {x}, this is the value: {y}"

tc = TreemapChart(title="Treemap Chart", colors=colors, tooltip=tooltip)
```

### Prepare names and values for series

```python
names = ["cats", "dogs", "birds", "fish", "snakes"]
values = [10, 4, 35, 12, 5]
```

### Set series to TreemapChart widget

```python
tc.set_series(names, values)
```

### Using @click event

```python
clicked_datapoint = Text(status="info")

@tc.click
def clicked(datapoint):
    # Datapoint is a namedtuple with fields: series_index, data_index, data
    # data is a dict with fields: name, value
    clicked_datapoint.text = datapoint
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# Creating Card widget, which will contain the Transfer widget and the Text widget.
card = Card(title="TreemapChart", content=Container(widgets=[dc, clicked_datapoint]))
# Creating the application layout.
layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
# Initializing the application.
app = sly.Application(layout=layout)
```

![mini_app](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/242254345-8fc4eccd-b04b-489a-89ee-4569de6a2624.png)
