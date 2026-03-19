# SelectDatasetTree

## Introduction

The **`SelectDatasetTree`** widget in Supervisely is a user interface element that allows users to select datasets from a tree-like structure that can be useful when working with hierarchical data.
The `SelectDatasetTree` widget has event handlers that are triggered when selected datasets are changed. This can be useful for applications that require dynamic change handling, such as showing data from the selected items.

[Read this tutorial in the developer portal.](https://developer.supervisely.com/app-development/widgets/selection/selectdatasettree)

## Function signature

```python
SelectDatasetTree(
    default_id = None,
    project_id = None,
    multiselect = False,
    compact = False,
    select_all_datasets = False,
    allowed_project_types = None,
    flat = False,
    always_open = False,
    team_is_selectable = True,
    workspace_is_selectable = True,
    append_to_body = True,
    widget_id = None,
)
```

![default](https://github.com/user-attachments/assets/6a18f9f9-20d1-4388-ace9-fa587c41dc33)

## Parameters

|        Parameters         |             Type              |                                       Description                                       |
| :-----------------------: | :---------------------------: | :-------------------------------------------------------------------------------------: |
|       `default_id`        |        `Optional[int]`        |          The default dataset ID to be selected when the widget is initialized.          |
|       `project_id`        |        `Optional[int]`        |                      The project ID to be displayed in the widget.                      |
|       `multiselect`       |            `bool`             |            If `True`, multiple datasets can be selected. Default is `False`.            |
|         `compact`         |            `bool`             |      If `True`, the widget will be displayed in compact mode. Default is `False`.       |
|   `select_all_datasets`   |            `bool`             |        If `True`, all datasets will be selected by default. Default is `False`.         |
|  `allowed_project_types`  | `Optional[List[ProjectType]]` |       A list of project types that are allowed to be selected. Default is `None`.       |
|          `flat`           |            `bool`             | If `True`, selecting a parent dataset will not select its children. Default is `False`. |
|       `always_open`       |            `bool`             |           If `True`, the tree will be opened when the widget is initialized.            |
|   `team_is_selectable`    |            `bool`             |                 If `True`, the team can be selected. Default is `True`.                 |
| `workspace_is_selectable` |            `bool`             |              If `True`, the workspace can be selected. Default is `True`.               |
|     `append_to_body`      |            `bool`             |  If `True`, the widget will be appended to the body. If `False`, it will be prepended.  |
|        `widget_id`        |             `str`             |                             An optional ID for the widget.                              |

### default_id

Determine the `default_id` of the dataset to be selected when the widget is initialized.<br>
NOTE: You must provide the `project_id` to use this parameter.

**type:** `int`

**default value:** `None`

Initialize the widget with the `default_id` and `project_id`:

```python
select_dataset_tree = SelectDatasetTree(default_id=default_id, project_id=project_id)
```

![default_id](https://github.com/user-attachments/assets/5237339e-0bed-44c1-8d8b-e76c151611b2)

### project_id

Determine the `project_id` to be displayed in the widget.<br>

**type:** `int`

**default value:** `None`

Initialize the widget with the `project_id`:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id)
```

![project_id](https://github.com/user-attachments/assets/fb062ee0-fc5d-4db9-bb78-f8584010146d)

### multiselect

If `True`, multiple datasets can be selected. Default is `False`.<br>

**type:** `bool`

**default value:** `False`

Initialize the widget with multiple selection:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, multiselect=True)
```

![multiselect](https://github.com/user-attachments/assets/a8f1c692-c693-4f82-87f5-303ee214f673)

### compact

If `True`, the widget will be displayed in compact mode without team, workspace, and project selection. Default is `False`.<br>

**type:** `bool`

**default value:** `False`

Initialize the widget in compact mode:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, compact=True)
```

![compact](https://github.com/user-attachments/assets/02641ac6-09ee-44e5-a439-8e397a27500e)

### select_all_datasets

If `True`, all datasets will be selected by default. Default is `False`.<br>
NOTE: This parameter is only available when `multiselect` is `True`.

**type:** `bool`

**default value:** `False`

Initialize the widget with all datasets selected:

```python
select_dataset_tree = SelectDatasetTree(
    project_id=project_id, select_all_datasets=True, multiselect=True
)
```

![select_all_datasets](https://github.com/user-attachments/assets/e55c2c7f-47a4-433e-bccd-dea99f742a67)

### allowed_project_types

A list of project types that are allowed to be selected. Default is `None`.<br>

**type:** `List[ProjectType]`

**default value:** `None`

Initialize the widget with allowed project types:

```python
select_dataset_tree = SelectDatasetTree(
    project_id=project_id, allowed_project_types=[ProjectType.IMAGES]
)
```

### flat

If `True`, selecting a parent dataset will not select its children. Default is `False`.<br>

**type:** `bool`

**default value:** `False`

Initialize the widget with the flat option:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, flat=True, multiselect=True)
```

![flat](https://github.com/user-attachments/assets/d3339a38-f5a5-4685-b2f4-b08b3bb20fdf)

### always_open

If `True`, the tree will be opened when the widget is initialized.<br>

**type:** `bool`

**default value:** `False`

Initialize the widget with the always open option:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, always_open=True)
```

![always_open](https://github.com/user-attachments/assets/71bd940e-a69f-442d-b91b-743ee8bdf03d)

### team_is_selectable

If `True`, the team can be selected. Default is `True`.<br>

**type:** `bool`

**default value:** `True`

Initialize the widget with the team selectable:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, team_is_selectable=False)
```

![team_is_selectable](https://github.com/user-attachments/assets/e5ecd73d-2f73-4916-9410-541578b1b870)

### workspace_is_selectable

If `True`, the workspace can be selected. Default is `True`.<br>

**type:** `bool`

**default value:** `True`

Initialize the widget with the workspace selectable:

```python
select_dataset_tree = SelectDatasetTree(project_id=project_id, workspace_is_selectable=False)
```

![workspace_is_selectable](https://github.com/user-attachments/assets/7bf46e45-05a2-49f9-b167-4d0f26beaaa3)

## Methods and attributes

|          Methods and attributes           |                              Description                               |
| :---------------------------------------: | :--------------------------------------------------------------------: |
|     `set_project_id(project_id: int)`     |           Set the project ID to be displayed in the widget.            |
|        `get_selected_project_id()`        |               Get the selected project ID in the widget.               |
|           `get_selected_ids()`            |    Get the selected dataset IDs in the widget for multiselect mode.    |
|            `get_selected_id()`            |   Get the selected dataset ID in the widget for single select mode.    |
|      `value_changed(func: Callable)`      |  Set the callback function to be triggered when the value is changed.  |
|     `set_dataset_id(dataset_id: int)`     |            Set the dataset ID to be selected in the widget.            |
| `set_dataset_ids(dataset_ids: List[int])` | Set the dataset IDs to be selected in the widget for multiselect mode. |
|         `get_selected_team_id()`          |                Get the selected team ID in the widget.                 |
|        `set_team_id(team_id: int)`        |             Set the team ID to be selected in the widget.              |
|       `get_selected_workspace_id()`       |              Get the selected workspace ID in the widget.              |
|   `set_workspace_id(workspace_id: int)`   |           Set the workspace ID to be selected in the widget.           |
|            `is_all_selected()`            |                  Check if all datasets are selected.                   |
|              `select_all()`               |                   Select all datasets in the widget.                   |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/selection/022_select_dataset_tree/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/021_select_dataset_tree/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from typing import List

from supervisely.app.widgets import SelectDatasetTree, Card, Container
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize the widget

```python
project_id = 39396
default_id = 93446

select_dataset_tree = SelectDatasetTree(
    project_id=project_id, default_id=default_id, multiselect=True, flat=True
)
```

### Create an event handler for the SelectDatasetTree widget

```python
@select_dataset_tree.value_changed
def on_change(dataset_ids: List[int]) -> None:
    print(f"Selected dataset ids: {dataset_ids}")
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# Creating Card widget, which will contain all the widgets.
card = Card(
    title="SelectDatasetTree",
    content=Container(widgets=[select_dataset_tree]),
)

# Creating the application layout.
layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

![mini_app](https://github.com/user-attachments/assets/6a18f9f9-20d1-4388-ace9-fa587c41dc33)
