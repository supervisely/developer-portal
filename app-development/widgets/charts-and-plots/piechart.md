# PieChart

## Introduction

**`PieChart`** widget in Supervisely is a widget used for displaying a pie chart. It allows users to visualize data for comparison distribution of different objects. The `PieChart` widget allows easily visualize data to determine the distribution of objects in comparison to each other.

{% tabs %}
{% tab title="Click slice demo" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/46297e16-c3bb-4986-a225-9a9e82a7cc34" alt=""><figcaption><p>Click slice demo</p></figcaption></figure>
{% endtab %}

{% tab title="Click side value" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/fc558ad8-b4f0-4b0e-b338-1f0e584bf2a4" alt=""><figcaption><p>Click side value'</p></figcaption></figure>
{% endtab %}
{% endtabs %}

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/charts-and-plots/piechart)

## Function signature

```python
PieChart(
    title="Simple Pie",
    series=[
        {"name": "Team A", "data": 44},
        {"name": "Team B", "data": 55},
        {"name": "Team C", "data": 33},
    ],
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/49abc2a2-e975-4838-89ce-e9d6eab5901c)

## Parameters

|   Parameters   |                Type                 |               Description                |
|:--------------:|:-----------------------------------:|:----------------------------------------:|
|    `title`     |                `str`                |             `PieChart` title             |
|    `series`    | `List[Dict[str, Union[int,float]]]` |          `PieChart` slices data          |
| `stroke_width` |                `str`                |            Gap between slices            |
| `data_labels`  |               `bool`                |           Show value on slices           |
|    `height`    |          `Union[int, str]`          |          `PieChart` height size          |
|     `type`     |      `Literal["pie", "donut"]`      | `PieChart` type. Can be `pie` or `donut` |

### title

Determines `PieChart` title.

**type:** `str`

### series

`PieChart` slices data.
The series should be a list of dicts with keys `name` and `data`.
The `name` key should contain the name of the slice and the `data` key should contain the value of the slice.

**type:** `List[Dict[str, Union[int,float]]]`

**default value:** `[]`

```python
series=[
    {"name": "Team A", "data": 44},
    {"name": "Team B", "data": 55},
    {"name": "Team C", "data": 33},
]

PieChart(
    title="Simple Pie",
    series=series,
)
```

### stroke_width

Determines the gap between slices on `PieChart`. Set to `0` to remove the gap.

**type:** `int`

**default value:** `2`

```python
    title="Simple Pie",
    series=[
        {"name": "Team A", "data": 44},
        {"name": "Team B", "data": 55},
        {"name": "Team C", "data": 33},
    ],
    stroke_width=0,
```

![stroke_width = 0](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/5da10e2f-6225-4dfe-99d3-6b24efeff990)

### data_labels

If `True` show values on `PieChart` slices. If `False` hide values on `PieChart` slices.

**type:** `bool`

**default value:** `False`

```python
    title="Simple Pie",
    series=[
        {"name": "Team A", "data": 44},
        {"name": "Team B", "data": 55},
        {"name": "Team C", "data": 33},
    ],
    data_labels=True,
```

![data_labels = True](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/01d77d6b-c942-407b-b28f-418c1b0e1f90)

### height

Determines the size of `PieChart`.

**type:** `Union[int, str]`

**default value:** `350`

```python
    title="Simple Pie",
    series=[
        {"name": "Team A", "data": 44},
        {"name": "Team B", "data": 55},
        {"name": "Team C", "data": 33},
    ],
    height=500,
```

### type

Determines the type of `PieChart`. Can be `pie` or `donut`.

**type:** `Literal["pie", "donut"]`

**default value:** `pie`

```python
    title="Simple Pie",
    series=[
        {"name": "Team A", "data": 44},
        {"name": "Team B", "data": 55},
        {"name": "Team C", "data": 33},
    ],
    type="donut",
```

{% tabs %}
{% tab title="type='donut'" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/4d6c83b1-86b9-49fd-9249-6be1bc364e37" alt=""><figcaption><p>type='donut'</p></figcaption></figure>
{% endtab %}

{% tab title="type='pie'" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/c27e7556-c788-48ce-8e9a-6c3546ecc1c7" alt=""><figcaption><p>type='pie'</p></figcaption></figure>
{% endtab %}
{% endtabs %}

## Methods and attributes

|                     Methods and attributes                      |                                Description                                |
|:---------------------------------------------------------------:|:-------------------------------------------------------------------------:|
| `add_series(names: List[str], values: List[Union[int, float]])` |                Adds a new series to the `PieChart` widget.                |
| `set_series(names: List[str], values: List[Union[int, float]])` |   Sets a series to the `PieChart` widget, removing all previous series.   |
|                    `get_series(index: int))`                    |           Returns a series from the `PieChart` widget by index.           |
|                   `delete_series(index: int)`                   |           Deletes a series from the `PieChart` widget by index.           |
|                    `get_clicked_datapoint()`                    | Returns the clicked datapoint from the `PieChart` widget as `NamedTuple`. |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/charts-and-plots/009_pie_chart/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/charts-and-plots/009_pie_chart/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Button,
    Card,
    Container,
    Flexbox,
    Input,
    InputNumber,
    PieChart,
    Text,
)
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize PieChart widget

```python
# Prepare names and values for series
series = [
    {"name": "Team A", "data": 44},
    {"name": "Team B", "data": 55},
    {"name": "Team C", "data": 13},
    {"name": "Team D", "data": 43},
    {"name": "Team E", "data": 22},
]

pie_chart = PieChart(
    title="Simple Pie",
    series=series,
    stroke_width=1,
    data_labels=True,
    height=350,
    type="pie",
)
```

### Create GUI with buttons

```python
# Create Text widget for displaying clicked datapoint info and place it in the Container widget with PieChart.
text = Text("Click on the slice to see it's value", status="info")
container_chart = Container(widgets=[pie_chart, text])

# Create Input widget for slice name, InputNumber widget for slice value and Buttons for managing slices.
input_name = Input(value="Team F", maxlength=10)
input_value = InputNumber(value=10)
button_add = Button("Add slice")
button_set = Button("Set slice")
button_remove = Button("Delete selected slice", "danger")

# Disable remove button until slice is selected.
button_remove.disable()

# Create Flexbox widget for buttons and place it in the Container widget with Input widgets.
buttons_flex = Flexbox(widgets=[button_add, button_set, button_remove])
container_add = Container(widgets=[input_name, input_value, buttons_flex])

# Place both Container widgets in the main Container widget. We will pass it to the app layout later.
main_container = Container(widgets=[container_chart, container_add])
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# Creating Card widget, which will contain the Transfer widget and the Text widget.
card = Card(title="Pie Chart", content=main_container)

# Creating the application layout.
layout = card
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
# Initialize the application.
app = sly.Application(layout=layout)
```

### Using @click event

```python
@pie_chart.click
def show_selection(datapoint: PieChart.ClickedDataPoint):
    # Datapoint is a namedtuple with fields: series_index, data_index, data
    data_name = datapoint.data["name"]
    data_value = datapoint.data["value"]
    data_index = datapoint.data_index

    # Update text widget with selected slice info.
    text.set(f"Selected slice: {data_name}, Value: {data_value}, Index: {data_index}", "info")

    # Enable remove button when slice is selected.
    button_remove.enable()
```

### Add buttons logic: Add, Set, Remove

```python
@button_add.click
def add_slice():
    name = input_name.get_value()
    value = input_value.get_value()
    pie_chart.add_series(names=[name], values=[value])


@button_set.click
def set_slice():
    name = input_name.get_value()
    value = input_value.get_value()
    pie_chart.set_series(names=[name], values=[value])


@button_remove.click
def remove_slice():
    datapoint: PieChart.ClickedDataPoint = pie_chart.get_clicked_datapoint()
    datapoint.data_index
    pie_chart.delete_series(datapoint.data_index)
    button_remove.disable()
```

![mini_app](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/9fc610fa-d1a8-4a6b-a0ec-d56ea6f33e0f)
