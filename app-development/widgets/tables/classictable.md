# ClassicTable

## Introduction

**`ClassicTable`** is a widget in Supervisely that is used for displaying and manipulating data in a table format. It is similar to the `Table` widget but with fewer customization options and functionalities.

{% hint style="info" %}
Note, that this is a legacy version. It is recommended to use the newer `Table` widget instead.
{% endhint %}

## Function signature

```python
classic_table = ClassicTable()
classic_table.read_pandas(pd.DataFrame(data=data, index=b, columns=a))
```

or

```python
classic_table = ClassicTable(
    data=pd.DataFrame(data=data, index=b, columns=a),
    fixed_columns_num=1,
    widget_id=None
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/218053006-6a62f66e-2848-42cb-8dda-6f36146107f0.png" alt=""><figcaption></figcaption></figure>

## Parameters

|      Parameters     |            Type            |               Description               |
| :-----------------: | :------------------------: | :-------------------------------------: |
|        `data`       | `pd.DataFrame()` or `dict` |              Data of table              |
|      `columns`      |           `list`           |          List of columns names          |
| `fixed_columns_num` |            `int`           | Number of fixed columns (left to right) |
|     `widget_id`     |            `str`           |             ID of the widget            |

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

classic_table = ClassicTable(data=df)py
```

### columns

List of columns names.

**type:** `list`

**default value:** `None`

```python
columns = [f"Col#{i}" for i in range(1, 11)]
data = pd.DataFrame(data=data, index=b, columns=columns)

classic_table = ClassicTable(data=df)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/218053549-86215c69-3d43-4a99-a3f4-1eb53b168dcd.png" alt=""><figcaption></figcaption></figure>

### fixed\_columns\_num

Number of fixed colums (left to right).

**type:** `int`

**default value:** `None`

```python
classic_table = ClassicTable(data=df, fixed_columns_num=2)
```

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

### Methods and attributes

|       Attributes and Methods       | Description                                                  |
| :--------------------------------: | ------------------------------------------------------------ |
|         `fixed_columns_num`        | Get or set number of fixed columns (left to right) property. |
|             `to_json()`            | Convert table data to json.                                  |
|            `to_pandas()`           | Convert table data to pandas dataframe.                      |
|      `read_json(value: dict)`      | Read and set table data from json.                           |
| `read_pandas(value: pd.DataFrame)` | Read and set table data from pandas dataframe.               |
|       `insert_row(index=-1)`       | Insert new row in table.                                     |
|   `pop_row(value: pd.DataFrame)`   | Remove row from table by index.                              |
|     `get_selected_cell(state)`     | Get selected table cell info.                                |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/002\_classic\_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/002\_classic\_table/src/main.py)

### Import libraries

```python
import os

import pandas as pd
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, ClassicTable, Container
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

### Initialize `ClassicTable` widget

```python
classic_table = ClassicTable(data=df)
```

or you can initialize empty table and set data later.

```python
classic_table = ClassicTable()
classic_table.read_pandas(df)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Classic Table",
    content=classic_table,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/218065148-e15d9642-2ca2-4132-9f76-52ed3cff5cce.png" alt=""><figcaption></figcaption></figure>
