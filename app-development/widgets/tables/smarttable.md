# Smart Table

## Introduction

**`SmartTable`** widget in Supervisely allows for displaying and manipulating data of various dataset statistics and processing it on the server side.

The `SmartTable` widget allows searching, sorting by column and direction, and the ability to customize data. It also allows updating table data in real-time through Python code.

## Function signature

Data structure example

```python
smart_table = SmartTable(
    data=data_list,
    columns=columns_list,
    columns_options=columns_options_dict,
    project_meta=project_meta,
    fixed_columns=1,
    page_size=10,
    clickable_rows=True,
    clickable_cells=False,
    sort_column_idx=1,
    sort_order="desc",
    width="auto",
    widget_id=None,
)
```

<figure><img src="../../../.gitbook/assets/widget-smartTable.png" alt=""/><figcaption></figcaption></figure>

## Parameters

|    Parameters     |                 Type                  |                Description                 |
| :---------------: | :-----------------------------------: | :----------------------------------------: |
|      `data`       | `Optional[Union[list, pd.DataFrame]]` |                 Table data                 |
|     `columns`     |           `Optional[list]`            |               Table columns                |
| `columns_options` |        `Optional[list[dict]]`         |              Columns options               |
|  `project_meta`   | `Optional[Union[ProjectMeta, dict]]`  |              Project metadata              |
|  `fixed_columns`  |        `Optional[Literal[1]]`         |     Number of the first fixed columns      |
|    `page_size`    |            `Optional[int]`            |     Table page size in number of rows      |
| `clickable_rows`  |           `Optional[bool]`            |             Are rows clickable             |
| `clickable_cells` |           `Optional[bool]`            |            Are cells clickable             |
| `sort_column_idx` |                 `int`                 | Index of the column by which sorting works |
|   `sort_order`    |  `Optional[Literal["asc", "desc"]]`   |                 Sort order                 |
|      `width`      |            `Optional[str]`            |               Width of table               |
|    `widget_id`    |            `Optional[str]`            |              ID of the widget              |

### data

Table data in different formats:

**type:** `Optional[Union[list, pd.DataFrame]]`

**default value:** `None`

1. Python `list` with structure:

    ```python
    data_list = [        
        [
            ["apple", "21"],
            ["banana", "15"],
        ],
    ]

    smart_table = SmartTable(data=data_list)
    ```

    Where:
      - `data_list` - list with `rows`  

2. Pandas `DataFrame`:   
    
    ```python
    dataframe = pd.DataFrame(data=data, columns=columns)
    smart_table = SmartTable(data=dataframe)
    ```
    Where:
      - `columns` - list of column names
      - `data` - list with `rows`
        - `row` - list with values, requires `len(row) == len(columns)`

### columns

Column names. If passed - will overwrite columns if data in `DataFrame` type is passed.

**type:** `Optional[list]`

**default value:** `None`

```python
smart_table = SmartTable(data=data_list, columns=columns_list)
```

### columns_options

List of dictionaries with column options. `len(columns_options)` must equal `len(columns)` to properly add options. If the column has no options, just pass an empty dictionary `{}`

Each dictionary may contain:
    - `type` - determines special type of column, depending on which styles will be applied, now supports only `class` as special type, you don't need to specify a type for regular
    - `subtitle` - provide additional clarification, specify units of measurement, offer context, and enhance overall understanding of the data
    - `maxValue` - determines the maximum value for the column and activates special bars that visualize how close the value is to the maximum value
    - `postfix` - is substituted after each value to denote dimensionality
    - `tooltip` - tooltip with description for a column

**type:** `Optional[list[dict]]`

**default value:** `None`

```python
smart_table = SmartTable(data=data_list, columns=columns_list)
```

### project_meta

Project metadata with classes used to apply special styles for columns with `type`:`class`

**type:** `Optional[Union[ProjectMeta, dict]]`

**default value:** `None`

```python
meta_json = api.project.get_meta(id=project_id)
smart_table = SmartTable(data=data_dict, project_meta=meta_json)

# or

meta = sly.ProjectMeta.from_json(data=meta_json)
smart_table = SmartTable(data=data_dict, project_meta=meta)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/57998637/e7a86d91-88a6-49ee-8021-189693fb41fd" alt="Project Meta"><figcaption></figcaption></figure>


### fixed_columns

Number of the first fixed columns if the table has horizontal scrolling.

💡 Currently supports only `1` or `None` as value.

**type:** `Optional[Literal[1]]`

**default value:** `None`

```python
smart_table = SmartTable(data=dataframe, fixed_columns=1)
```

### page_size

Table page size in number of rows

**type:** `int`

**default value:** `10`

```python
smart_table = SmartTable(data=dataframe, page_size=15)
```

### clickable_rows

Whether the rows are clickable, an event is called when the row is clicked

💡 Cannot be used together with `clickable_cells`

**type:** `Optional[bool]`

**default value:** `False`

```python
smart_table = SmartTable(data=data_dict, clickable_rows=True)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/57998637/e72c987e-49a9-40b0-a4af-a2e970a9265b" alt="Clickable Rows"><figcaption></figcaption></figure>

### clickable_cells

Whether the cells are clickable, an event is called when the cell is clicked

💡 Cannot be used together with `clickable_rows`

**type:** `Optional[bool]`

**default value:** `False`

```python
smart_table = SmartTable(data=data_dict, clickable_cells=True)
```

### sort_column_idx

Index of the column by which sorting works.

💡 Cannot be used without `sort_order`

**type:** `int`

**default value:** `None`

```python
smart_table = SmartTable(data=dataframe, sort_column_idx=0, sort_order='asc')
```

### sort_order

Sort order.

💡 Cannot be used without `sort_column_idx`

**type:** `Optional[Literal["asc", "desc"]]`

**default value:** `None`

```python
smart_table = SmartTable(data=dataframe, sort_column_idx=1, sort_order='desc')
```

### width

Width of table.

**type:** `Optional[str]`

**default value:** `auto`

```python
smart_table = SmartTable(data=data_dict, width="50%")
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/57998637/bba653f9-d5aa-421d-8c74-ff01942e355e" alt="Width"><figcaption></figcaption></figure>

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`


## Methods and attributes

|                     Attributes and Methods                      | Description                                                               |
| :-------------------------------------------------------------: | ------------------------------------------------------------------------- |
|                       `fixed_columns_num`                       | Get or set number of fixed columns (left to right) property.              |
|                         `project_meta`                          | Get or set number project meta property.                                  |
|                           `page_size`                           | Get or set table page size property.                                      |
|           `read_json(data: dict, meta: dict = None)`            | Read and set table data from JSON format.                                 |
|                `read_pandas(data: pd.DataFrame)`                | Read and set table data from `DataFrame`.                                 |
|                 `to_json(active_page = False)`                  | Convert table data to JSON format. Full table or active page only.        |
|                `to_pandas(active_page = False)`                 | Convert table data to pandas `DataFrame`. Full table or active page only. |
|                       `clear_selection()`                       | Deselect a table cell.                                                    |
|                      `get_selected_row()`                       | Get selected table row info.                                              |
|                      `get_selected_cell()`                      | Get selected table cell info.                                             |
|                `insert_row(row: List, index=-1)`                | Insert new row in table by index.                                         |
|                       `pop_row(index=-1)`                       | Remove row from table by index.                                           |
|                     `search(search_value)`                      | Search source data for `search_value` and return results as `DataFrame`.  |
| `sort(column_id: int, order: Optional[Literal["asc", "desc"]])` | Sort table rows by given column ID and/or order direction.                |
|                          `@row_click`                           | Decorator function is handled when table row is clicked.                  |
|                          `@cell_click`                          | Decorator function is handled when table cell is clicked.                 |


### read_json()

The `data` structure must correspond to the dictionary given in the example below 👇

Structure:
  - `columns` - list of column names
  - `data` - list with `rows`
    - `row` - list with values, requires `len(row) == len(columns)`
  - `columnsOptions` - list of dictionaries in which settings for each column are stored
    - `type` - determines special type of column, depending on which styles will be applied, now supports only `class` as special type, you don't need to specify a type for regular
    - `subtitle` - provide additional clarification, specify units of measurement, offer context, and enhance overall understanding of the data
    - `maxValue` - determines the maximum value for the column and activates special bars that visualize how close the value is to the maximum value
    - `postfix` - is substituted after each value to denote dimensionality
    - `tooltip` - tooltip with description for a column
  - `options` - dict with table options
    - `fixColumns` - number of first fixed table columns. ℹ️ Currently, only the first column is supported
    - `sort` - dict with applied sorting options
      - `columnIndex` - index of the `column`, by the values of which the data should be sorted
      - `order` - sort order
    - `pageSize` - how many `rows` will be displayed on the table page


## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/006\_smart\_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/006\_smart\_table/src/main.py)

### Import libraries

```python
import os
import json
import pandas as pd

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Card,
    Container,
    SmartTable,
)
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare data and meta that will be used to create example table

```python
data = [
    ["apple", "21"],
    ["banana", "15"]
  ]

columns = ["Class", "Items"]

dataframe = pd.DataFrame(data=data, columns=columns)

columns_options = [
    { "type": "class"},
    { "maxValue": 21, "postfix": "pcs", "tooltip": "description text", "subtitle": "boxes" }
  ]

meta_path = "meta.json"
with open(meta_path, "r") as json_file:
    meta = json.load(json_file)    
```

### Initialize `SmartTable` widget

```python
smart_table = SmartTable(
    data=dataframe,
    project_meta=meta,
    columns_options=columns_options,
    clickable_rows=True,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Smart Table",
    content=smart_table,
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
@smart_table.row_click
def handle_table_row(datapoint: sly.app.widgets.SmartTable.ClickedDataRow):
    sly.app.show_dialog(
        f"{datapoint.row[0]}",
        f"You clicked table row with idx={datapoint.row_index} in source data",
    )
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/57998637/2f119b05-6a1b-49d6-8f5e-099c8a5dc826" alt="Smart Table Example"><figcaption></figcaption></figure>
