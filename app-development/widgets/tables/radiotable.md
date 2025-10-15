# RadioTable

## Introduction

**`RadioTable`** widget in Supervisely is a user interface element that allows creating a table of options, each with a corresponding radio button. With `RadioTable` widget, developer can define multiple rows of options, and only one row option can be selected. The RadioTable widget provides a convenient and intuitive way to navigate and make selections within the table.

## Function signature

```python
RadioTable(
    columns, rows,
    subtitles={},
    column_formatters={},
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218065127-20f844dc-09f0-4a9a-a140-6bd3c10e1991.png" alt=""><figcaption></figcaption></figure>

## Parameters

|     Parameters      |             Type              |               Description               |
| :-----------------: | :---------------------------: | :-------------------------------------: |
|      `columns`      |          `List[str]`          |       `RadioTable` columns names        |
|       `rows`        |       `List[List[str]]`       |        `RadioTable` rows content        |
|     `subtitles`     | `Union[Dict[str, str], List]` |     Determine subtitles for columns     |
| `column_formatters` |            `Dict`             | Determine format of output `RadioTable` |
|     `widget_id`     |             `str`             |            ID of the widget             |

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
        ["Supervisely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ]
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218065127-20f844dc-09f0-4a9a-a140-6bd3c10e1991.png" alt=""><figcaption></figcaption></figure>

### subtitles

Determine subtitles for columns.

**type:** `Union[Dict[str, str], List]`

**default value:** `{}`

```python
subtitles = {"Format": "Format subtitle", "Number of Classes": "Number of Classes subtitle"}
radio_table = RadioTable(
    columns=["Format", "Total Images", "Number of Classes", "Number of Objects"],
    rows=[
        ["Supervisely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ],
    subtitles=subtitles,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218070385-8af00847-d258-4b73-84a2-3f3515c1039c.png" alt=""><figcaption></figcaption></figure>

### column_formatters

Determine format of output `RadioTable`.

**type:** `Dict`

**default value:** `{}`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                       Attributes and Methods                                       | Description                                                                                            |
| :------------------------------------------------------------------------------------------------: | ------------------------------------------------------------------------------------------------------ |
|                                             `columns`                                              | Get or set `columns` property.                                                                         |
|                                            `subtitles`                                             | Get or set `subtitles` property.                                                                       |
|                                               `rows`                                               | Get or set `rows` property.                                                                            |
|                           `format_value(column_name: str, value: list)`                            | Return column formatter function by given column name.                                                 |
|                                  `default_formatter(value: list)`                                  | Return default column formatter.                                                                       |
|                                        `get_selected_row()`                                        | Return selected row data.                                                                              |
|                                     `get_selected_row_index()`                                     | Return selected row index.                                                                             |
|           `set_columns(columns: List[str], subtitles: Union[Dict[str, str], List[str]])`           | Set table columns by given list of column names.                                                       |
| `set_data(columns: List[str], rows: List[List[str]], subtitles: Union[Dict[str, str], List[str]])` | Set table data.                                                                                        |
|                                    `select_row(row_index: int)`                                    | Set row selected by given index. If row with given index does not exist raise `ValueError`.            |
|                                          `@value_changed`                                          | Decorator function is handled when widget value is changed                                             |
|                                       `select_row_by_value`                                        | Set row selected by given column and value. If row with given value does not exist raise `ValueError`. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/003_radio_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/003_radio_table/src/main.py)

### Import libraries

```python
import os
import time

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, DoneLabel, Progress, RadioTable
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
        ["Supervisely", "245", "15", "5468"],
        ["PascalVOC", "76", "20", "387"],
        ["COCO", "128", "80", "786"],
        ["Cityscapes", "45", "35", "1334"],
    ],
)
```

### Create additional widgets we will use in UI

```python
button = Button("Button")
progress = Progress(message="processing images", show_percents=True)
progress.hide()

done_label = DoneLabel()
done_label.hide()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
radio_table_card = Card(
    title="Radio Table",
    content=Container([radio_table, button]),
)

progress_card = Card(
    title="Progress",
    content=Container([progress, done_label]),
)

layout = Container(
    widgets=[radio_table_card, progress_card],
    direction="horizontal",
    fractions=[1, 1],
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
@button.click
def show_progress():
    progress.show()
    selected_row = radio_table.get_selected_row()
    images_count = int(selected_row[1])

    with progress(message="Downloading images from Flickr...", total=images_count) as pbar:
        for _ in range(images_count):
            time.sleep(0.01)
            pbar.update(1)
    done_label.text = f"{images_count} images processed from {selected_row[0]} format data."
    done_label.show()
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218076702-49568654-4161-45b7-b87c-91281e08363b.png" alt=""><figcaption></figcaption></figure>
