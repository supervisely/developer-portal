# CustomModelsSelector

## Introduction

**`CustomModelsSelector`** widget allows creating a table with NN models that you have trained in Supervisely by providing path to training application default save directory, Team ID and task type (e.g. object detection, instance segmentation, pose estimation and etc).

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/selectors/custom-models-selector)

## Function signature

```python
CustomModelsSelector(
    team_id = team_id,
    training_app_directory="/yolov8_train/",
    task_type="object_detection",
    widget_id=None,
)
```

![custom-models-selector](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1766078982620.png)

## Parameters

|        Parameters        | Type  |                              Description                              |
| :----------------------: | :---: | :-------------------------------------------------------------------: |
|        `team_id`         | `int` |                            Current Team ID                            |
| `training_app_directory` | `str` | Path to directory in Team File to training app default save directory |
|       `task_type`        | `str` |                            Model task type                            |
|       `widget_id`        | `str` |                           ID of the widget                            |

### team_id

Team ID where training app default save directory is located.

**type:** `int`

### training_app_directory

Path to directory in Team File to training app default save directory. For example, if you have trained model in `Train YOLOv8` application, you should provide `/yolov8_train/` path.

**type:** `str`

### task_type

Type of problem that model solves. Can be selected in the Training app GUI. For example, models that were trained with `Train YOLOv8` application can have one of the following task types: `object detection`, `instance segmentation`, `pose estimation`.

**type:** `str`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|            Attributes and Methods            |                            Returns                             |
| :------------------------------------------: | :------------------------------------------------------------: |
|                  `columns`                   |                          `List[str]`                           |
|                    `rows`                    |                  `Dict[str, List[ModelRow]]`                   |
|             `get_selected_row()`             |                      `ModelRow` or `None`                      |
|          `get_selected_row_index()`          |                        `int` or `None`                         |
|           `set_active_row(index)`            |                             `None`                             |
|          `get_selected_task_type()`          |                             `str`                              |
|      `set_active_task_type(task_type)`       |                             `None`                             |
|         `get_available_task_types()`         |                          `List[str]`                           |
|        `get_selected_model_params()`         |                        `Dict` or `None`                        |
|        `use_custom_checkpoint_path()`        |                             `bool`                             |
|        `get_custom_checkpoint_name()`        |                             `str`                              |
|        `get_custom_checkpoint_path()`        |                             `str`                              |
|      `set_custom_checkpoint_path(path)`      |                             `None`                             |
|  `set_custom_checkpoint_preview(file_info)`  |                             `None`                             |
|     `get_custom_checkpoint_task_type()`      |                             `str`                              |
| `set_custom_checkpoint_task_type(task_type)` |                             `None`                             |
|                  `enable()`                  |                             `None`                             |
|                 `disable()`                  |                             `None`                             |
|               `enable_table()`               |                             `None`                             |
|              `disable_table()`               |                             `None`                             |
|               `@value_changed`               |   Decorator function is handled when widget value is changed   |
|             `@task_type_changed`             | Decorator function is handled when widget task type is changed |

## ModelRow

`ModelRow` object represents an automatically generated row in the `CustomModelsSelector` widget. It has information about the training session including:

- ID of the training session
- Names and paths to model artifacts
- Training data project

### Function signature

```python
ModelRow(
    api=api,
    team_id=team_id,
    task_id=task_id,
    task_path=task_path,
    training_project_name=training_project_name,
    artifacts_infos=artifacts_infos,
    session_link=session_link,
)
```

### Methods and attributes

|     Attributes and Methods     | Description                                                      |
| :----------------------------: | ---------------------------------------------------------------- |
|           `task_id`            | Return task id of the training session.                          |
|          `task_date`           | Return date and time of the training session.                    |
|          `task_link`           | Return link to the training session.                             |
|    `training_project_info`     | Return `ProjectInfo` object with project info.                   |
|       `artifacts_paths`        | Return list of all artifacts paths.                              |
|       `artifacts_names`        | Return list of artifacts names.                                  |
|      `artifacts_selector`      | Return artifact selector widget with type `Select`.              |
| `get_selected_artifact_path()` | Return path of the selected artifact.                            |
| `get_selected_artifact_name()` | Return name of the selected artifact.                            |
|          `to_html()`           | Converts object to html string to use it in the widget template. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/019_trained_models_selector/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/019_trained_models_selector/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, CustomModelsSelector, Container, Text, Button
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `CustomModelsSelector` widget

```python
team_id = sly.env.team_id()

checkpoint_infos = sly.nn.checkpoints.yolov8.get_list(api, team_id)
trained_models_table = CustomModelsSelector(team_id, checkpoint_infos)
```

### Create additional widgets to preview selected model

```python
model_name_preview = Text("", "text")
model_path_preview = Text("", "text")
preview_container = Container([model_name_preview, model_path_preview])
preview_container.hide()
preview_button = Button("Show preview")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've just created in the `Container` widget.

```python
container = Container([trained_models_table, preview_button, preview_container])
card = Card(title="TrainedModelsTable", content=container)
layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from code

```python
@trained_models_table.value_changed
def get_selected_row(row: CustomModelsSelector.ModelRow):
    preview_container.hide()


@preview_button.click
def preview_button_click_handler():
    preview_container.hide()
    row = trained_models_table.get_selected_row()
    model_name = row.get_selected_checkpoint_name()
    model_path = row.get_selected_checkpoint_path()

    model_name_preview.set(f"Model name: {model_name}", "text")
    model_path_preview.set(f"Model path: {model_path}", "text")
    preview_container.show()
```
