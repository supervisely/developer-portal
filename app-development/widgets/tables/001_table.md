# Table

## Introduction

**`Table`** widget in Supervisely allows for displaying and manipulating data in a table format. 

It supports data in the Pandas DataFrame format or a Python dictionary with a specific structure. 

The `Table` widget allows searching, sorting by column and direction, and the ability to download or customize data before downloading. It also allows for the creation of buttons in table cells and updating table data in real-time through Python code. 

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/table)

## Function signature

```python
table = Table()
table.read_pandas(pd.DataFrame(data=data, index=b, columns=a))
```

or

```python
table = Table(
    data=pd.DataFrame(data=data, index=b, columns=a),
    fixed_cols=1,
    per_page=10,
    page_sizes=[10, 15, 30, 50, 100],
    width="auto",
    sort_column_id=1,
    sort_direction="asc",
    widget_id=None
)
```

![table-new](https://user-images.githubusercontent.com/79905215/218051109-136fdb07-eccf-420f-ae9d-67dd7f195f21.png)

## Parameters

|    Parameters    |                Type                |                 Description                 |
| :--------------: | :--------------------------------: | :-----------------------------------------: |
|      `data`      |     `pd.DataFrame()` or `dict`     |                Data of table                |
|    `columns`     |          `Optional[list]`          |            List of columns names            |
|   `fixed_cols`   |          `Optional[int]`           |   Number of fixed columns (left to right)   |
|    `per_page`    |          `Optional[int]`           |           Default per page value            |
|   `page_sizes`   |       `Optional[List[int]]`        |             Page sizes presets              |
|     `width`      |          `Optional[str]`           |               Width of table                |
| `sort_column_id` |               `int`                | Column ID by which the table will be sorted |
| `sort_direction` | `Optional[Literal["asc", "desc"]]` |           Table sorting direction           |
|   `widget_id`    |               `str`                |              ID of the widget               |

### data

Data of table in different formats:

1. Pandas Dataframe:

```python
pd.DataFrame(data=data, columns=columns)
```

2. Python dict with structure:

```python
 {
    "columns_names": ["col_name_1", "col_name_2", ...],
    "values_by_rows": [
        ["row_1_column_1", "row_1_column_2", ...],
        ["row_2_column_1", "row_2_column_2", ...],
        ...
    ]
}
```

```python
# prepare data for table
a = list(range(1, 11))
b = list(range(1, 5))

data = []
for row in b:
    temp = [round(row * number, 1) for number in a]
    data.append(temp)

a = [str(i) for i in a]
b = [str(i) for i in b]

data = pd.DataFrame(data=data, index=b, columns=a)

table = Table(data=df)
```

### columns

List of columns names.

**type:** `Optional[list]`

**default value:** `None`

```python
columns = [f"Col#{i}" for i in range(1, 11)]
data = pd.DataFrame(data=data, index=b, columns=columns)

table = Table(data=df)
```

![table-new](https://user-images.githubusercontent.com/79905215/218051109-136fdb07-eccf-420f-ae9d-67dd7f195f21.png)

### fixed_cols

Number of fixed colums (left to right).

**type:** `Optional[int]`

**default value:** `None`

```python
table = Table(data=df, fixed_cols=2)
```

<video width="860" controls>
   <source src="https://user-images.githubusercontent.com/79905215/217930130-1f2585bd-8491-4753-a2f3-d27bd539420d.mp4" type="video/mp4">
 </video>

### per_page

Default per page value.

**type:** `Optional[int]`

**default value:** `10`

```python
table = Table(data=df, per_page=4)
```

![table-4](https://user-images.githubusercontent.com/79905215/217930845-58509b43-3aca-4be5-8039-cf18562ede08.png)

### page_sizes

Page sizes presets.

**type:** `Optional[List[int]]`

**default value:** `[10, 15, 30, 50, 100]`

### width

Width of table.

**type:** `Optional[int]`

**default value:** `10`

```python
table = Table(data=df, width="200px")
```

### sort_column_id

Column ID by which the table will be sorted.

**type:** `int`

**default value:** `0`

```python
table = Table(data=df, sort_column_id=2)
```

### sort_direction

Table sorting direction.

**type:** `Optional[Literal["asc", "desc"]]`

**default value:** `asc`

```python
table = Table(data=df, sort_direction="desc")
```

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                        Attributes and Methods                                        | Description                                                             |
| :--------------------------------------------------------------------------------------------------: | ----------------------------------------------------------------------- |
|                                         `fixed_columns_num`                                          | Get or set number of fixed columns (left to right) property.            |
|                                            `summary_row`                                             | Get or set summary row data property.                                   |
|                                             `to_json()`                                              | Convert table data to json.                                             |
|                                            `to_pandas()`                                             | Convert table data to pandas dataframe.                                 |
|                                       `read_json(value: dict)`                                       | Read and set table data from json.                                      |
|                                  `read_pandas(value: pd.DataFrame)`                                  | Read and set table data from pandas dataframe.                          |
|                                        `insert_row(index=-1)`                                        | Insert new row in table by index.                                       |
|                                    `pop_row(value: pd.DataFrame)`                                    | Remove row from table by index.                                         |
|                                      `get_selected_cell(state)`                                      | Get selected table cell info.                                           |
|                          `get_html_text_as_button(title: str = "preview")`                           | Static method to get html code for button with given title.             |
|                               `create_button(title: str = "preview")`                                | Static method to get html code for button with given title.             |
|                                              `loading`                                               | Get or set table loading status property.                               |
|                                         `clear_selection()`                                          | Deselect a table cell.                                                  |
|                       `delete_row(key_column_name: str, key_cell_value: str)`                        | Delete row by column name and cell value.                               |
|   `update_cell_value(key_column_name: str, key_cell_value: str, column_name: str, new_value: str)`   | Update single cell value by column name and cell value.                 |
| `update_matching_cells(key_column_name: str, key_cell_value: str, column_name: str, new_value: str)` | Update all matching cells values by column name and cell value.         |
|                 `sort(column_id: int, direction: Optional[Literal["asc", "desc"]])`                  | Sort table rows by given column ID and/or order direction.              |
|                                          `@download_as_csv`                                          | Decodator function is handled when "download as csv" button is pressed. |
|                                               `@click`                                               | Decodator function is handled when table cell is pressed.               |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/023_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/023_table/src/main.py)

### Import libraries

```python
import os

import pandas as pd
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Table
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare function that creates example pandas table

```python
def multiplication_table():
    a = list(range(1, 11))
    b = list(range(1, 6))

    data = []
    for row in b:
        temp = [round(row * number, 1) for number in a]
        temp[-1] = sly.app.widgets.Table.create_button("Delete row")
        data.append(temp)

    a = [f"Col#{str(i)}" for i in a]
    b = [str(i) for i in b]
    return pd.DataFrame(data=data, index=b, columns=a)
```

Create data for table.

```python
df = multiplication_table()
```

### Initialize `Table` widget

```python
table = Table(data=df)
```

or you can initialize empty table and set data later.

```python
table = Table()
table.read_pandas(df)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Table",
    content=table,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widget from python code

```python

@table.download_as_csv
def download_as_csv():
    df = table.to_pandas()
    df = df.drop(index=[0, 1, 2])
    return df
```

```python
@table.click
def handle_table_button(datapoint: sly.app.widgets.Table.ClickedDataPoint):
    if datapoint.button_name is None:
        return
    row_num = datapoint.row["Col#1"]
    if datapoint.button_name == "Delete row":
        table.delete_row("Col#1", row_num)
```

![table-app](https://user-images.githubusercontent.com/79905215/219018134-14fd350a-b5b0-49d5-8649-fa1a8956a7ad.gif)
