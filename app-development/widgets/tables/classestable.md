# ClassesTable

## Introduction

**`ClassesTable`** widget in Supervisely allows users to display all classes from given project in a table format. The widget shows the class name, geometry type, and the number of images and labels associated with each class. With `ClassesTable` widget, users can enable multi-selection mode, restrict the display of certain geometry types, and manage the selected classes. Users can customize the appearance and behavior of the widget to match their project requirements. `ClassesTable` widget also allows users to retrieve a list of the selected classes from the code. Overall, the ClassesTable widget is a valuable tool for organizing and managing the classes in Supervisely apps.

## Function signature

```python
ClassesTable(
    project_meta=None,
    project_id=None,
    project_fs=None,
    allowed_types=None,
    selectable=True,
    disabled=False,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219954018-a0d76d1e-d617-4729-9f8f-62ad688031ad.png" alt=""><figcaption></figcaption></figure>

## Parameters

|    Parameters   |       Type       |                              Description                              |
| :-------------: | :--------------: | :-------------------------------------------------------------------: |
|  `project_meta` |   `ProjectMeta`  |                          Input `ProjectMeta`                          |
|   `project_id`  |       `int`      |                      Input Supervisely project ID                     |
|   `project_fs`  |     `Project`    |                            Input `Project`                            |
| `allowed_types` | `List[Geometry]` | `Geometry` types that will be display from all types in given project |
|   `selectable`  |      `bool`      |                  Whether the component is selectable                  |
|    `disabled`   |      `bool`      |                   Whether the component is disabled                   |
|   `widget_id`   |       `str`      |                            ID of the widget                           |

### project\_meta

Determine input `ProjectMeta`.

**type:** `ProjectMeta`

**default value:** `None`

```python
meta_json = api.project.get_meta(id=project_id)
meta = sly.ProjectMeta.from_json(data=meta_json)
classes_table = ClassesTable(project_meta=meta)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219953958-f31b1c04-4a2e-4be4-8f48-039b71ebb2f9.png" alt=""><figcaption></figcaption></figure>

### project\_id

Determine input project ID.

**type:** `int`

**default value:** `None`

```python
classes_table = ClassesTable(project_id=project_id)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219954018-a0d76d1e-d617-4729-9f8f-62ad688031ad.png" alt=""><figcaption></figcaption></figure>

### project\_fs

Determine input `Project`, located on local host.

**type:** `Project`

**default value:** `None`

```python
project_path = "/home/admin/work/supervisely/projects/lemons_annotated"
project = sly.Project(project_path, sly.OpenMode.READ)
classes_table = ClassesTable(project_fs=project)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219954018-a0d76d1e-d617-4729-9f8f-62ad688031ad.png" alt=""><figcaption></figcaption></figure>

### allowed\_types

Determine `Geometry` types that will be display from all types in given project.

**type:** `List[Geometry]`

**default value:** `None`

```python
classes_table = ClassesTable(
    project_id=project_id,
    allowed_types=[sly.Polygon],
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219954233-dd463cec-b385-4386-b951-3b017df55f3e.png" alt=""><figcaption></figcaption></figure>

### selectable

Determine whether the component is selectable.

**type:** `bool`

**default value:** `True`

```python
classes_table = ClassesTable(
    project_id=project_id,
    selectable=False,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219954378-3ddb4098-93c7-49fe-9a8d-dc49515d60a6.png" alt=""><figcaption></figcaption></figure>

### disabled

Determine whether the component is disabled.

**type:** `bool`

**default value:** `False`

```python
classes_table = ClassesTable(
    project_id=project_id,
    disabled=True,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219955255-6b2a7075-8e58-4934-9ab4-b3dbb4c11ce8.gif" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|           Attributes and Methods           | Description                                                  |
| :----------------------------------------: | ------------------------------------------------------------ |
| `read_meta(project_meta: sly.ProjectMeta)` | Read given `ProjectMeta`.                                    |
|   `read_project(project_fs: sly.Project)`  | Read given `Project`.                                        |
|   `read_project_from_id(project_id: int)`  | Read given `Project` by ID.                                  |
|          `get_selected_classes()`          | Return list of selected classes.                             |
|             `clear_selection()`            | Clear selected data.                                         |
|              `value_changed()`             | Decorator function is handled when input value is changed.   |
|           `loading(value: bool)`           | Decorator function is handled when input value is uplouding. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/004\_classes\_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/004\_classes\_table/src/main.py)

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Container, ClassesTable, Text, Card
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `ClassesTable` widget by project ID

```python
project_id = sly.env.project_id()
class_table = ClassesTable(project_id=project_id)
label = Text("")
```

### Initialize `ClassesTable` widget by local project

```python
data_dir = sly.app.get_data_dir() # create local directory
if sly.fs.dir_exists(data_dir):
    sly.fs.clean_dir(data_dir) # clean directory

project_dir = os.path.join(data_dir, "sly_project")
sly.Project.download(api, project_id, project_dir)
project = sly.Project(project_dir, sly.OpenMode.READ)
local_class_table = ClassesTable(project_fs=project)
local_label = Text("")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Classes Table",
    content=Container([class_table, label], gap=5),
)
card_local = Card(
    title="Classes Table Local",
    content=Container([local_class_table, local_label], gap=5),
)

layout = Container(widgets=[card, card_local])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Handle ClassesTable clicks

Use the decorator as shown below to handle `ClassesTable` clicks.

```python
@class_table.value_changed
def class_table_value_changed(selected_classes):
    label.text = f"Selected classes: {', '.join(selected_classes)}"


@local_class_table.value_changed
def class_table_value_changed(selected_classes):
    local_label.text = f"Selected classes: {', '.join(selected_classes)}"
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221355359-03e32c23-3a89-4e63-996d-78417ba43e39.gif" alt=""><figcaption></figcaption></figure>