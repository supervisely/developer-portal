# Date Picker

## Introduction

**`DatePicker`** widget in Supervsely is a user-friendly and customizable date input solution for Supervisely app developers. It features an intuitive interface that allows users to select a date from a calendar or enter one manually. The widget supports a wide range of date formats, making it suitable for use in diverse applications. With its flexible styling options, the DatePicker component can seamlessly integrate with apps and enhance the user experience.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/misc/datepicker)

## Function signature

```python
date_picker = DatePicker(
    value=None,
    placeholder="Select date",
    picker_type="date",
    size=None,
    readonly=False,
    disabled= False,
    editable=True,
    clearable=True,
    format="yyyy-MM-dd",
    first_day_of_week=1,
    widget_id=None,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/226650502-d9409595-2775-4c96-9e80-be7f8bf3ddc1.gif" alt="default" />
</p>

## Parameters

|     Parameters      |                                         Type                                         |                      Description                       |
| :-----------------: | :----------------------------------------------------------------------------------: | :----------------------------------------------------: |
|       `value`       |                            `Union[int, str, list, tuple]`                            |              Default value to date picker              |
|    `placeholder`    |                                   `str = "Select`                                    |              Date picker input help text               |
|    `picker_type`    | `Literal["year", "month", "date", "datetime", "week", "datetimerange", "daterange"]` |                    Date picker type                    |
|       `size`        |                      `Literal["large", "small", "mini", None]`                       |                 Determine widget size.                 |
|     `readonly`      |                                        `bool`                                        | Specifies that a date picker input should be read-only |
|     `disabled`      |                                        `bool`                                        |                     Disable widget                     |
|     `editable`      |                                        `bool`                                        |      Allows to edit date picker input value text       |
|     `clearable`     |                                        `bool`                                        |           Allows to delete value by clicking           |
| `first_day_of_week` |                                        `int`                                         |            Determine first day of the week             |
|      `format`       |                                        `str`                                         |           Determine value displaying format            |
|     `widget_id`     |                                        `str`                                         |                    ID of the widget                    |

### value

Default value to date picker

**type:** `Union[int, str, list, tuple]`

**default value:** `None`

```python
date_picker = DatePicker(value="2023 13 03")
```

![value](https://user-images.githubusercontent.com/79905215/226652706-16c62fd6-4230-440b-acbb-5256f10af3cf.png)

### placeholder

Date picker input help text

**type:** `str`

**default value:** `Select date"`

```python
date_picker = DatePicker(placeholder="placeholder")
```

![placeholder](https://user-images.githubusercontent.com/79905215/226655559-5915ab0f-8fd2-4c6c-8e6e-4c9af1bdd690.png)

### picker_type

Date picker type

**type:** `Literal["year", "month", "date", "datetime", "week", "datetimerange", "daterange"]`

**default value:** `"date"`

```python
date_picker = DatePicker(picker_type="year")
```

![type-year](https://user-images.githubusercontent.com/79905215/226656844-192d92ab-47b3-4e8c-9ab0-aeb6027d6269.png)

```python
date_picker = DatePicker(picker_type="month")
```

![type-month](https://user-images.githubusercontent.com/79905215/226656862-6ee616c4-cedb-4c6e-9f0a-75ecd74bab74.png)

```python
date_picker = DatePicker(picker_type="week")
```

![type-week](https://user-images.githubusercontent.com/79905215/226656890-5ec80bcb-32f9-4441-a64a-72e4acf22afc.png)

```python
date_picker = DatePicker(picker_type="date")
```

![type-date](https://user-images.githubusercontent.com/79905215/226657766-5758f52e-5173-42ff-a2e6-f9bc32335c20.png)

```python
date_picker = DatePicker(picker_type="datetime")
```

![type-dt](https://user-images.githubusercontent.com/79905215/226657847-4ad6a36a-9036-4560-a063-b1da08b7574b.png)

```python
date_picker = DatePicker(picker_type="daterange")
```

![type-range](https://user-images.githubusercontent.com/79905215/226656188-85e95928-11d2-4851-aa84-6609d85f8d93.png)

```python
date_picker = DatePicker(picker_type="datetimerange")
```

![type-dtrange](https://user-images.githubusercontent.com/79905215/226657880-23d5c257-5320-483d-b298-b903e75515b4.png)

### size

Determine widget size

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
date_picker = DatePicker(size="mini")
```

![mini](https://user-images.githubusercontent.com/79905215/226842782-09d793fe-3902-4da5-b6d9-ae5ef0cd31d1.png)

```python
date_picker = DatePicker(size="small")
```

![small](https://user-images.githubusercontent.com/79905215/226842792-f71c346c-6418-4b83-bc3a-7dbb8ea80744.png)

```python
date_picker = DatePicker(size="large")
```

![large](https://user-images.githubusercontent.com/79905215/226842814-58d16bc4-09dd-4f84-9245-c64c7d5ca87e.png)

### readonly

Specifies that a date picker input should be read-only

**type:** `bool`

**default value:** `False`

```python
date_picker = DatePicker(
    value="2023 03 21",
    readonly=True,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/226665172-dadc160b-e026-492d-a3a5-57552d344bc9.gif" alt="readonly" />
</p>

### disabled

Disable widget

**type:** `bool`

**default value:** `False`

```python
date_picker = DatePicker(disabled=True)
```

![disabled](https://user-images.githubusercontent.com/79905215/226667379-4a778ed7-3b2b-46f6-b58c-a48140d04f48.png)

### editable

Allows to edit date picker input value text

**type:** `bool`

**default value:** `True`

```python
date_picker = DatePicker(editable=True)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/226678607-1fa49944-d2b0-48fb-b437-d894150704ef.gif" alt="editable" />
</p>

### clearable

Allows to delete value by clicking

**type:** `bool`

**default value:** `True`

```python
date_picker = DatePicker(clearable=True)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/226684080-f99b6c73-8f4f-4595-ae0d-b5a0743cfe83.gif" alt="clearable" />
</p>

```python
date_picker = DatePicker(
    editable=False,
    clearable=False,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/226684134-970d569d-959a-467b-a1ab-7eb389f68f8b.gif" alt="clearable" />
</p>

### format

Determine value displaying format

**type:** `str`

**default value:** `"yyyy-MM-dd"`

```python
date_picker = DatePicker(format="dd/MM/yyyy")
```

![format](https://user-images.githubusercontent.com/79905215/226684703-b21e03e2-2fc4-4d49-b778-f262fd0b7d37.png)

### first_day_of_week

Determine first day of the week

**type:** `int`

**default value:** `1`

```python
date_picker = DatePicker(first_day_of_week=4)
```

![weekday](https://user-images.githubusercontent.com/79905215/226685175-d3348f3e-fa68-4208-a3b5-44d3ead8d97c.png)

### widget_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

|             Attributes and Methods             | Description                                                           |
| :--------------------------------------------: | --------------------------------------------------------------------- |
|                 `get_value()`                  | Get date picker datetime string value                                 |
| `set_value(value: Union[int, str, datetime])`  | Set date picker value                                                 |
| `set_range_values(values: Union[list, tuple])` | Set date range picker values                                          |
|                `clear_value()`                 | Clear date picker value                                               |
|                `@value_changed`                | Decorator function is handled when date picker value is changed value |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/input/005_date_picker/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/input/005_date_picker/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, DatePicker, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api: sly.Api = sly.Api.from_env()
```

### Initialize `DatePicker` widgets

```python
daterange_picker = DatePicker(
    editable=False,
    placeholder="Select date range",
    picker_type="daterange",
)

date_picker = DatePicker(
    editable=False,
)
```

### Create `Button`, `Text` widgets we will use in UI for demo

```python
text = Text()
button_clear = Button("Clear")
button_default = Button("Set default")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
controls_container = Container([button_clear, button_default], direction="horizontal")
middle_container = Container([controls_container, text])

card = Card(
    "Date Picker",
    content=Container([daterange_picker, middle_container, date_picker], direction="horizontal"),
)


layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from code

```python
def _check_dates(value1, value2):
    from datetime import datetime

    if value1 is None or value2 is None:
        text.set(text="", status="info")
        return

    def _get_datetime(val: str):
        val = val.replace("T", " ").replace("Z", "")
        return datetime.strptime(val, "%Y-%m-%d %H:%M:%S.%f")

    if _get_datetime(value2[0]) <= _get_datetime(value1) <= _get_datetime(value2[1]):
        text.set(text="Date 1 in daterange.", status="success")
        return
    text.set(text="Date 1 not in daterange.", status="error")


@date_picker.value_changed
def check_date(date_value):
    daterange_value = daterange_picker.get_value()
    _check_dates(date_value, daterange_value)


@daterange_picker.value_changed
def check_daterange(daterange_value):
    date_value = date_picker.get_value()
    _check_dates(date_value, daterange_value)


@button_clear.click
def clear_dates():
    date_picker.clear_value()
    daterange_picker.clear_value()


@button_default.click
def set_default():
    from datetime import datetime, timedelta

    now = datetime.now()
    yesterday = now - timedelta(days=1)
    tomorrow = now + timedelta(days=1)

    date_picker.set_value(now)
    daterange_picker.set_range_values([yesterday, tomorrow])

```

<p align="center">
    <img src="https://user-images.githubusercontent.com/79905215/226875476-e2350f86-da21-43c8-b0bf-eef46b51a004.gif" alt="layout">
</p>
