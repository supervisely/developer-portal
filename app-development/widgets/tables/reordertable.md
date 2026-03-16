# ReorderTable

## Introduction

**`ReorderTable`** widget in Supervisely displays tabular data and allows users to reorder rows with drag-and-drop.

It supports multi-row selection and provides quick actions in a floating panel (`Top`, `Up`, `Set to #`, `Down`, `Bottom`) to move selected rows.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/tables/reordertable)

## Function signature

```python
ReorderTable(
    columns: List[str],
    data: Optional[List] = None,
    page_size: int = 10,
    content_top_right: Optional[Widget] = None,
    widget_id: Optional[str] = None,
)
```

## Parameters

|     Parameters      |        Type        |                             Description                             |
| :-----------------: | :----------------: | :-----------------------------------------------------------------: |
|      `columns`      |    `List[str]`     |                        Table columns names.                         |
|       `data`        |  `Optional[List]`  |   Table rows, where each row is a list of cell values by column.    |
|     `page_size`     |       `int`        |                      Number of rows per page.                       |
| `content_top_right` | `Optional[Widget]` | Optional widget rendered in the top-right area of the table header. |
|     `widget_id`     |  `Optional[str]`   |                          ID of the widget.                          |

## Methods and attributes

|        Attributes and Methods         | Description                                                               |
| :-----------------------------------: | ------------------------------------------------------------------------- |
|             `get_order()`             | Get current rows order as list of 0-based indices from the original data. |
|        `get_reordered_data()`         | Get table rows in the current user-defined order.                         |
|  `get_column_data(column_name: str)`  | Get values from one column in the current row order.                      |
| `set_data(columns: List, data: List)` | Replace table data and reset order.                                       |
|            `reset_order()`            | Reset current order to the original identity order.                       |
|         `is_order_changed()`          | Check whether current order differs from the original order.              |
|           `@order_changed`            | Decorator callback fired whenever row order changes in the UI.            |

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
]

reset_btn = Button("Reset order")
reorder_table = ReorderTable(
    columns=columns,
    data=data,
    page_size=5,
    content_top_right=reset_btn,
)
```

### Create app layout

```python
order_text = Text("Current order: [0, 1, 2]")
content = Container([reorder_table, order_text])
card = Card(title="ReorderTable", content=content)
app = sly.Application(layout=card)
```

### Add callbacks

```python
@reorder_table.order_changed
def handle_reorder(order):
    order_text.text = f"Current order: {order}"


@reset_btn.click
def reset_order():
    reorder_table.reset_order()
```
