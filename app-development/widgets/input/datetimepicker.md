# Date and Time Picker

## Introduction

**`DateTimePicker`** is a widget in Supervisely that allows you to choose a date and time on the UI.

[Read this tutorial in the developer portal.](https://developer.supervise.ly/app-development/widgets/charts-and-plots/datetimepicker)

## Function signature

```python
DateTimePicker(
    value=None
    readonly=False,
    disabled=False,
    editable=False,
    clearable=True,
    size=None,
    placeholder="Select date and time",
    w_type="datetime",
    format="yyyy-MM-dd HH:mm:ss",
    widget_id=None,
)
```

<p align="center">
<img src="https://user-images.githubusercontent.com/57998637/227946386-f23cad29-1c32-409d-8531-d9e1024b587e.gif" width="500px" >
</p>

## Parameters

|  Parameters   |                                         Type                                         |            Description             |
| :-----------: | :----------------------------------------------------------------------------------: | :--------------------------------: |
|    `value`    |                         `Union[int, str, list, tuple, None]`                         | Default value in `DateTimePicker`  |
|  `readonly`   |                                        `bool`                                        |    Allows to set read only mode    |
|  `disabled`   |                                        `bool`                                        | Allows to disable `DateTimePicker` |
|  `editable`   |                                        `bool`                                        |        Allows to edit input        |
|  `clearable`  |                                        `bool`                                        |         Shows clear button         |
|    `size`     |                      `Literal["large", "small", "mini", None]`                       |        Size of input field         |
| `placeholder` |                                        `str`                                         |    `DateTimePicker` placeholder    |
|   `w_type`    | `Literal["year", "month", "date", "datetime", "week", "datetimerange", "daterange"]` |    `DateTimePicker` picker type    |
|   `format`    |                   `Literal["yyyy", "MM", "dd", "HH", "mm", "ss"]`                    |     Datetime displaying format     |
|  `widget_id`  |                                        `str`                                         |          ID of the widget          |

### value

Determine `DateTimePicker` default value.

**type:** `Union[int, str, list, tuple, None]`

**default value:** `None`

```python
datetime_picker = DateTimePicker(value="2023-03-22 14:01:02")
```

<img src="https://user-images.githubusercontent.com/57998637/227953057-4a0838c4-92a0-4b62-a6f2-8e9d44b3978d.png" width="500px" >

### placeholder

Determine `DateTimePicker` placeholder.

**type:** `str`

**default value:** `"Select date and time"`

```python
datetime_picker = DateTimePicker(placeholder="Select")
```

<img src="https://user-images.githubusercontent.com/57998637/227946395-466df30b-2ec0-48a3-8b9b-d52cbfa944dc.png" width="500px" >

### size

Determine `DateTimePicker` size of the input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
datetime_picker = DateTimePicker()
datetime_picker_mini = DateTimePicker(size="mini")
datetime_picker_small = DateTimePicker(size="small")
datetime_picker_large = DateTimePicker(size="large")
```

<img src="https://user-images.githubusercontent.com/57998637/227946398-4d086ac3-c591-40a9-884e-2ef6911175bf.png" width="500px" >

### w_type

Determine `DateTimePicker` picker type.

**type:** `Literal["year", "month", "date", "datetime", "week", "datetimerange", "daterange"]`

**default value:** `"datetime"`

```python
datetime_picker = DateTimePicker(w_type="datetimerange")
```

<img src="https://user-images.githubusercontent.com/57998637/227946384-479d384d-e330-4960-b053-599f94cc9980.png" width="500px" >

### format

Determine `DateTimePicker` displaying format.

**type:** `Literal["yyyy", "MM", "dd", "HH", "mm", "ss"]`

**default value:** `"yyyy-MM-dd HH:mm:ss"`

```python
datetime_picker = DateTimePicker(format="yyyy/MM/dd HH:mm")
```

<img src="https://user-images.githubusercontent.com/57998637/227946394-8a494a04-32b4-4a9a-8e9a-6aaf2686924d.png" width="500px" >

### readonly

Determine whether `DateTimePicker` is read-only.

**type:** `bool`

**default value:** `False`

```python
datetime_picker = DateTimePicker(value="2023-03-22 14:01:02", readonly=True)
```

<img src="https://user-images.githubusercontent.com/57998637/227946397-c77714c3-1551-41ac-83f1-e6c25b54e2ed.gif" width="500px" >

### disabled

Determine whether `DateTimePicker` is disabled.

**type:** `bool`

**default value:** `False`

```python
datetime_picker = DateTimePicker(disabled=True)
```

<img src="https://user-images.githubusercontent.com/57998637/227946389-22d37fc5-4b41-4a00-9cdb-539b4a6171b4.png" width="500px" >

### editable

Determine whether the input is editable.

**type:** `bool`

**default value:** `False`

<img src="https://user-images.githubusercontent.com/57998637/227946392-4f5d0fca-bc28-4d2c-916a-42968a80661f.gif" width="500px" >

### clearable

Determine whether to show a clear button.

**type:** `bool`

**default value:** `True`

<img src="https://user-images.githubusercontent.com/57998637/227946399-11d5761b-387c-4b1d-b332-2ae3363e37f0.gif" width="500px" >

### widget_id

The ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

|                   Attributes and Methods                   | Description                                          |
| :--------------------------------------------------------: | ---------------------------------------------------- |
|                       `get_value()`                        | Return `DateTimePicker` current value.               |
| `set_value(value: Union[int, str, datetime, list, tuple])` | Set `DateTimePicker` value.                          |
|                      `@value_changed`                      | Decorator function to handle `DateTimePicker` click. |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/misc/datetime_picker/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/misc/datetime_picker/src/main.py)

### Import libraries

```python
import os
from datetime import datetime, timezone
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Text, DateTimePicker
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize `DateTimePicker` and `Text` widgets

```python
datetime_picker = DateTimePicker()
text = Text()
```

### Create app layout

Prepare a layout for an app using `Card` widget with the `content` parameter.

```python
card = Card(
    "Date and Time Picker",
    content=Container([datetime_picker, text]),
)
layout = Container(widgets=[card])
```

### Create an app using a layout

Create an app object with a layout parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control widgets from Python code

```python
@datetime_picker.value_changed
def set_only_date_from_today(datetime_value):
    if datetime_value is not None:
        format_string = "%Y-%m-%dT%H:%M:%S.%fZ"
        selected = datetime.strptime(datetime_value, format_string).replace(tzinfo=timezone.utc)
        current_day = datetime.combine(datetime.now().date(), datetime.min.time()).replace(
            tzinfo=timezone.utc
        )
        if selected < current_day:
            new_value = datetime.utcnow().replace(tzinfo=timezone.utc)
            iso_value = new_value.strftime("%Y-%m-%dT%H:%M:%S.%f")[:-3] + "Z"
            datetime_picker.set_value(iso_value)

    info = f"Selected date and time: {datetime_value}"
    text.set(text=info, status="info")
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/57998637/227992280-a3a6e837-5641-40bc-99dc-a1cd41ee2302.gif" alt="layout" width="500px"/>
</p>
