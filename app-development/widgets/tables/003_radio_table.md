# Radio Table

## Introduction

In this tutorial you will learn how to use `RadioTable` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/radiotable)

## Function signature

```python
RadioTable(columns, rows, subtitles={}, column_formatters={}, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218065127-20f844dc-09f0-4a9a-a140-6bd3c10e1991.png)

## Parameters

|     Parameters      |       Type        |               Description               |
| :-----------------: | :---------------: | :-------------------------------------: |
|      `columns`      |    `List[str]`    |       `RadioTable` columns names        |
|       `rows`        | `List[List[str]]` |        `RadioTable` rows content        |
|     `subtitles`     |      `dict`       |     Determine subtitles for columns     |
| `column_formatters` |      `dict`       | Determine format of output `RadioTable` |
|     `widget_id`     |       `str`       |            Id of the widget             |

### columns

Determine `RadioTable` columns names.

**type:** `List[str]`

### rows

Determine `RadioTable` rows content.

**type:** `List[List[str]]`

```python
radio_table = RadioTable(
    columns=["Format", "Total Images", "Number of Classes", "Number of Objects"],
    rows=[
        ["Supervsiely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ]
)
```

![default](https://user-images.githubusercontent.com/120389559/218065127-20f844dc-09f0-4a9a-a140-6bd3c10e1991.png)

### subtitles

Determine subtitles for columns.

**type:** `dict`

**default value:** `{}`

```python
subtitles = {"Format": "Format subtitle", "Number of Classes": "Number of Classes subtitle"}
radio_table = RadioTable(
    columns=["Format", "Total Images", "Number of Classes", "Number of Objects"],
    rows=[
        ["Supervsiely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ],
    subtitles=subtitles,
)
```

![subtitles](https://user-images.githubusercontent.com/120389559/218070385-8af00847-d258-4b73-84a2-3f3515c1039c.png)

### column_formatters

Determine format of output `RadioTable`.

**type:** `dict`

**default value:** `{}`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|      Attributes and Methods      | Description                                                                                   |
| :------------------------------: | --------------------------------------------------------------------------------------------- |
| `format_value(column_name: str)` | Return column formatter function by given column name.                                        |
| `default_formatter(value: list)` | Return column formatter.                                                                      |
| `get_selected_row(state: dict)`  | Return selected row data.                                                                     |
|   `select_row(row_index: int)`   | Set row with given index selected. If row with given index does not exist raise `ValueError`. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/026_radio_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/026_radio_table/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, RadioTable
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `RadioTable` widget

```python
radio_table = RadioTable(
    columns=["Format", "Total Images", "Number of Classes", "Number of Objects"],
    rows=[
        ["Supervsiely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ],
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Radio Table",
    content=radio_table,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218076702-49568654-4161-45b7-b87c-91281e08363b.png)
