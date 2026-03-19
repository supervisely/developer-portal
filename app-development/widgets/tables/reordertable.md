# ReorderTable

## Introduction

**`ReorderTable`** widget in Supervisely displays tabular data and lets users reorder rows directly in the UI.

**Key features:**

- Drag-and-drop row reordering
- Search rows by text
- Multi-row selection
- Floating action panel for moving selected rows (`Top`, `Up`, `Set to #`, `Down`, `Bottom`)
- Pagination (`page_size`)

![ReorderTable](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/reordertable.png)

## Function signature

```python
ReorderTable(
    columns: List[str],
    data: Optional[List[List]] = None,
    page_size: int = 10,
    content_top_right: Optional[Widget] = None,
    widget_id: Optional[str] = None,
)
```

![ReorderTable](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/reordertable2.png)

## Parameters

|     Parameters      |          Type          |                             Description                             |
| :-----------------: | :--------------------: | :-----------------------------------------------------------------: |
|      `columns`      |      `List[str]`       |                     Table column header names.                      |
|       `data`        | `Optional[List[List]]` |  Table rows; each row is a list of cell values ordered by columns.  |
|     `page_size`     |         `int`          |                      Number of rows per page.                       |
| `content_top_right` |   `Optional[Widget]`   | Optional widget rendered in the top-right area of the table header. |
|     `widget_id`     |    `Optional[str]`     |                          ID of the widget.                          |

### columns

Column header names.

**type:** `List[str]`

```python
columns = ["Name", "Score", "Department"]
```

### data

Initial table rows. Each row is a list of cell values ordered by `columns`.

**type:** `Optional[List[List]]`

**default value:** `None` (treated as an empty list)

**Notes:**

- Every row must have exactly `len(columns)` cells; otherwise a `ValueError` is raised.
- Cell values should be JSON-serializable (strings, numbers, booleans, etc.).

```python
data = [
    ["Alice", 95, "CV"],
    ["Bob", 87, "NLP"],
    ["Carol", 92, "ML Platform"],
]
reorder_table = ReorderTable(columns=columns, data=data)
```

### page_size

Number of rows per page.

**type:** `int`

**default value:** `10`

**Notes:**

- The value is converted to `int` and clamped to a minimum of `1`.

```python
reorder_table = ReorderTable(columns=columns, data=data, page_size=25)
```

### content_top_right

Optional widget rendered in the top-right area of the table header.
Common use-cases: refresh button, help button, status indicator.

**type:** `Optional[Widget]`

**default value:** `None`

```python
refresh_btn = Button("Refresh")
apply_btn = Button("Apply changes")

top_right_controls = Container(
    [apply_btn, refresh_btn],
    direction="horizontal",
)
reorder_table = ReorderTable(
    columns=columns,
    data=data,
    content_top_right=top_right_controls,
)
```

### widget_id

ID of the widget.

**type:** `Optional[str]`

**default value:** `None`

## Methods and attributes

|              Methods and attributes              |   Returns   | Description                                                                 |
| :----------------------------------------------: | :---------: | --------------------------------------------------------------------------- |
|                  `get_order()`                   | `List[int]` | Get current row order (0-based indices of rows in the original data).       |
|              `get_reordered_data()`              |   `List`    | Get all table rows arranged in the current user-defined order.              |
|       `get_column_data(column_name: str)`        |   `List`    | Get values from one column in the current row order.                        |
| `set_data(columns: List[str], data: List[List])` |   `None`    | Replace table data and reset order, page, and selection.                    |
|                 `reset_order()`                  |   `None`    | Reset current order to the original identity order (also clears selection). |
|               `is_order_changed()`               |   `bool`    | Check whether current order differs from the original order.                |
|                 `@order_changed`                 |  decorator  | Register a callback fired whenever row order changes in the UI.             |

![ReorderTable](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/reordertable1.png)

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/008_reorder_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/008_reorder_table/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Button, Card, Container, ReorderTable, Text
```

### Initialize widget

```python
columns = ["Name", "Score", "Department"]
data = [
    ["Alice", 95, "CV"],
    ["Bob", 87, "NLP"],
    ["Carol", 92, "ML Platform"],
    ["David", 79, "MLOps"],
    ["Eve", 98, "Research"],
    ["Frank", 84, "Data Engineering"],
]

refresh_btn = Button("Refresh")
apply_btn = Button("Apply changes")

saved_data = [list(row) for row in data]

top_right_controls = Container(
    [apply_btn, refresh_btn],
    direction="horizontal",
)
reorder_table = ReorderTable(
    columns=columns,
    data=data,
    page_size=5,
    content_top_right=top_right_controls,
)
```

### Create app layout

```python
content = Container([reorder_table])
card = Card(title="ReorderTable", content=content)
app = sly.Application(layout=card)
```

### Add functions to control widgets from code

```python

@refresh_btn.click
def refresh_state():
    reorder_table.set_data(columns, saved_data)
    update_state(reorder_table.get_order())


@apply_btn.click
def apply_changes():
    saved_data[:] = [list(row) for row in reorder_table.get_reordered_data()]
    reorder_table.set_data(columns, saved_data)
    update_state(reorder_table.get_order())
```
