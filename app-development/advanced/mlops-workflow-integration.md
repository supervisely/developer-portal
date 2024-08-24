---
description: This doc explains how to integrate MLOps Workflow in your application
---

# MLOps Workflow integration

## What is Workflow and what is it used for

When working in teams, it's important to stay informed about what has happened with your data and have the ability to reproduce an experiment using the same data that was initially used. There isn't always the opportunity to create a backup in time, and in the past, there was no quick and centralized way to track changes. But now, all of this is possible. To make it work for your application, you can easily set it up with just a few lines of code.

You can view the history of data changes in various scopes, meaning you can limit the visible area. To see the broadest scope, you should view the workflow for the workspace, while the narrowest scope can be seen at the task level. The standard view is the project workflow, which will display only the connections for the selected project. However, regardless of the scope you choose, it will display the history of all connections between all nodes. Currently, scopes only filter out unrelated elements at their level.


## The key formula for successfully building a workflow

Almost every process on the platform has input and output data, so the workflow is mostly built around tasks.

<img src="../../.gitbook/assets/wf-simple-schema.png" alt="" style='width: 480px'>

Therefore, when you develop an application, it's nice to to mark data import and export points for the workflow.

So, here are the types of import and export data that we can display in the workflow.

<table>
  <tr>
    <td valign="top" style="padding-right: 40px;">
      <strong>Input data types:</strong>
      <ul>
        <li>project</li>
        <li>dataset</li>
        <li>file in Files</li>
        <li>folder in Files</li>
        <li>task</li>
      </ul>
    </td>
    <td valign="top">
      <strong>Output data types:</strong>
      <ul>
        <li>project</li>
        <li>dataset</li>
        <li>file in Files</li>
        <li>folder in Files</li>
        <li>app session</li>
        <li>task</li>
        <li>labeling job</li>
      </ul>
    </td>
  </tr>
</table>

## A few words about data versioning

**`Exclusively for Pro and Enterprise subscribers`**<br>
**`Currently, only IMAGE projects are supported`**<br>
Another important part of the MLOps Workflow is data versioning. This mechanism allows you to save the state of a project to ensure the reproducibility of experiments. At any moment, you can recreate the current project as a new one, and its state will correspond to the selected save point. This mechanism has advantages over simple data copying.<br>
It is recommended to implement this functionality for automatic versioning of projects in applications like "Train NN model". 

## Workflow technical implementation

Applications you write using the SDK use an instance of the `Api` class to send requests. When the `Api` instance is initialized, it automatically initializes the `AppApi` and `Workflow` for it. Therefore, in your application's code, you need to use the corresponding methods in the places where you handle data retrieval and/or export.

```python
from supervisely import Api
api = Api.from_env()
```

### Project

These methods add project cards with preview and a link that allows you to directly access the project.

```python
api.app.workflow.add_input_project()
# or
api.app.workflow.add_output_project()
```

### Dataset

These methods add dataset cards with preview and a link that allows you to directly access the project.

```python
api.app.workflow.add_input_dataset()
# or
api.app.workflow.add_output_dataset()
```

### File in Files

These methods add file cards with corresponding icons and a link that allows you to directly access the catalog where the files are stored.

You can set a parameter that helps identify that a file is a model with weights.

```python
api.app.workflow.add_input_file()
# or
api.app.workflow.add_output_file()
```

### Folder in Files

These methods are the same as for files, but already customized for folders.

```python
api.app.workflow.add_input_folder()
# or
api.app.workflow.add_output_folder()
```

### Task

These methods add task cards with application icon and a link that allows you to directly access task logs.

```python
api.app.workflow.add_input_task()
# or
api.app.workflow.add_output_task()
```

### Offline session of application

This method is used to add a card that indicates the application has an offline session where you can find the result of its work.

```python
api.app.workflow.add_output_app()
```

### Labeling Job

This method is used to add a card that indicates the application has created a labeling job with the result of its work.

```python
api.app.workflow.add_output_job()
```

## How to customize cards

Each card added to the workflow can be customized as to what type of data it displays and what type of connection it relates to. You can also customize the main node at the time of card creation. Therefore, customizations can be made for two linked elements at once within a single operation: for `relation` which is our input or output data card and for the main `node` which is usually a task.

{% hint style="warning" %}
However, if you configure the main node settings across different inputs or outputs, the state of the main node will be overwritten by the last action.
{% endhint %}

To prepare the settings for any card, you need to use the `WorkflowSettings` class:

```python
from supervisely import WorkflowSettings
input_card_settings = WorkflowSettings()
```

The class has the following properties:
 - **title**: The name of the card
 - **icon**: The zmdi icon that will be displayed on the card
 - **icon_color**: The color of the icon in hex format
 - **icon_bg_color**: The background color of the icon in hex format
 - **url**: A link to any resource on the platform
 - **url_title**: A short title for the link

To assign these settings to a card, you need to compile them into a `WorkflowMeta` object:

```python
from supervisely import WorkflowMeta
input_card_settings = WorkflowMeta(relation_settings=input_card_settings)
```

You can configure either one or both cards at the same time. When assigning the meta, there's no need to specify the relation to any particular operation; in input methods, it will always relate to the input.

{% hint style="info" %}
☝️ In some cards, such as the project card, you can't customize the icon because they use an image instead. A complete list of what cannot be customized for each type of card will be provided later.
{% endhint %}

## Something special for Pro and Enterprise subscribers

You can easily add a data backup to your workflow. All you need to do is decide before which data operation you want to create the backup and add a single method.

```python
from supervisely import Api
api = Api.from_env()
...
version_id = api.project.version.create(
                  project_info, # or just ID as value
                  version_title,
                  version_description,
              )
...
```

{% hint style="info" %}
If a project already has a backup and there haven't been any changes since it was created, you'll receive the ID of that backup instead of creating a duplicate.
{% endhint %}

To recreate a project from a version, you need to use

```python
api.project.version.restore(
                  project_info, # or just ID as value
                  version_id # or version_num for this project,
                  skip_missed_entities,
              )
```

You can view the project version numbers and their corresponding IDs by using the method

```python
api.project.version.get_list(project_id)
```

## Minimum example for maximum understanding

Представим, что у нас есть какоето приложение, которое работает с аннотациями, а сессия приложения автоматически завершается после основного процесса. На входе у нас в большинстве случаев будет использоваться проект или датасет, а на выходе мы хотим получить новый проект и сразу же выгрузить его в каком-то из полулярных форматов в архиве.

Лучшей пактикой является добавление отдельного модуля workflow.py в src директорию приложения. В этом модуле будет описана вся логика подключения workflow вашего приложения.

### `workflow.py`

```python

from typing import Optional
import supervisely as sly

def workflow_input(api: sly.Api, project_info: sly.ProjectInfo):
    try:        
        api.app.workflow.add_input_project(project_info.id)
        sly.logger.debug(f"Workflow: Input project - {project_info.id}")
    except Exception as e:
        sly.logger.debug(f"Workflow: Failed to add input project to the workflow: {repr(e)}")

def workflow_output(
    api: sly.Api,
    project_info: Optional[sly.ProjectInfo] = None,
    file_info: Optional[FileInfo] = None,
):
    if project_info is None and file_info is None:
        sly.logger.warning(
            "Workflow: at least one of the `project_info` or `file_info` must be provided. "
            "Workflow output will not be processed."
        )
        return
    try:
        if project_info is not None:
            api.app.workflow.add_output_project(project_info.id)
            sly.logger.debug(f"Workflow: Output project - {project_info.id}")
    except Exception as e:
        sly.logger.debug(f"Workflow: Failed to add output project to the workflow: {repr(e)}")    
    try:
        if file_info is not None:
            relation_settings = sly.WorkflowSettings(
                title=f"Project Archive: {file_info.name}",
                icon="archive",
                icon_color="#ff4c9e",
                icon_bg_color="#ffe3ed",
                url=f"/files/{file_info.id}/true/?teamId={file_info.team_id}",
                url_title="Download",
            )
            meta = sly.WorkflowMeta(relation_settings=relation_settings)
            api.app.workflow.add_output_file(file_info, meta=meta)
            sly.logger.debug(f"Workflow: Output file - {file_infos[0]}")
    except Exception as e:
        sly.logger.debug(
            f"Workflow: Failed to add output file with project archive to the workflow: {repr(e)}"
        )
```

Next, integrate the workflow into your application code.

### `main.py`

```python

import globals as g # This is where we usually store the values of global variables
impor workflow as w 

def main()

    ... # Code where you retrieve project_info with the data you will work with

    w.workflow_input(g.api, project_info)

    ... # Code where you process the data, create a new project, and upload the result archive to Files

    w.workflow_output(g.api, res_project_info, file_info)
```

This way, we can track the history of data changes and visually see these changes at any time by accessing the MLOps Workflow.

<img src="../../.gitbook/assets/wf-example.png" alt="" style='width: 1898px'>