---
description: >-
  A step-by-step tutorial of how to create custom import Supervisely app from scratch with GUI.
---

# Create import Supervisely app with GUI from scratch

## Introduction

In this tutorial, we will create a simple import app that will import images from selected folder to Supervisely server.
This application has GUI and is designed to demonstrate the basic principles of creating import applications with interface.

## Data example

```text
📂my_folder
┣ 🖼️cat_1.jpg
┣ 🖼️cat_2.jpg
┗ 🖼️cat_3.jpg
```

You can find the above demo files in the data directory of the template-import-app repo - [here](https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/blob/master/data/)

<img src="https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/assets/48913536/f3fe1dd0-c357-42d0-9122-4591bf91ddcd">

## Tutorial content

[**Step 1.**](#step-1-how-to-debug-import-app) How to debug import app.

[**Step 2.**](#step-2-how-to-write-import-script) How to write import script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

Everything you need to reproduce this tutorial is on [GitHub](https://github.com/supervisely-ecosystem/import-from-scratch-gui): [main.py](https://github.com/supervisely-ecosystem/import-from-scratch-gui/blob/master/src/main.py).

Before we begin, please clone the project and set up the working environment - [here is a link with a description of the steps](./overview.md#set-up-an-environment-for-the-development).

## Step 1. How to debug import app

Open `local.env` and set up environment variables by inserting your values here for debugging. Learn more about environment variables in our [guide](../../getting-started/environment-variables.md)

**local.env:**

```python
TEAM_ID=8                    # ⬅️ change it to your team ID
WORKSPACE_ID=349             # ⬅️ change it to your workspace ID
FOLDER="data/my_folder"      # ⬅️ path to folder on local machine
```

## Step 2. How to write import script

Find source code for this example - [main.py](https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/blob/master/src/main.py)

**Step 1. Import libraries**

```python
import os

import supervisely as sly
from dotenv import load_dotenv

# to show error message to user with dialog window
from supervisely.app import DialogWindowError

# widgets that we will use in GUI
from supervisely.app.widgets import (
    Button,
    Card,
    Checkbox,
    Container,
    Input,
    ProjectThumbnail,
    SelectWorkspace,
    SlyTqdm,
    TeamFilesSelector,
    Text,
)
```

**Step 2. Load environment variables**

Load ENV variables for debug, has no effect in production.

```python
IS_PRODUCTION = sly.is_production()
if IS_PRODUCTION is True:
    load_dotenv("advanced.env")
    STORAGE_DIR = sly.app.get_data_dir()
else:
    load_dotenv("local.env")

load_dotenv(os.path.expanduser("~/supervisely.env"))

# Get ENV variables
TEAM_ID = sly.env.team_id()
WORKSPACE_ID = sly.env.workspace_id()
PATH_TO_FOLDER = sly.env.folder(raise_not_found=False)
```

**Step 3. Initialize API object**

Create API object to communicate with Supervisely Server. Loads from `supervisely.env` file

```python
api = sly.Api.from_env()
```

**Step 4. Make GUI with widgets**

We will build GUI for our import app using [Supervisely widgets](https://developer.supervisely.com/app-development/widgets).

We will breakdown our GUI into 4 steps:

1. File selector to select folder with data.
2. Import settings.
3. Destination project settings.
4. Output card with button to start import and info about result project.

Let's take a closer look at each step:

1. Create FileSelector widget to select folder with data and place it into Card widget with validation.
2. Create Checkbox widget to select if we want to remove source files after successful import and place it into Card widget.
3. Create workspace selector and input widget to enter project name. Combine those widgets into Container widget and place it into Card widget. Using workspace selector we can select team and workspace where we want to create project in which data will be imported.
4. Create Button widget to start import process.
5. Create output text to show warnings and info messages.
6. Create progress widget to show progress of import process.
7. Create ProjectThumbnail to show result project with link to it.
8. Combine all button, output text, progress and project thumbnail .
9. Create layout by combining all created cards into one container.
10. Initialize app object with layout as a parameter.

```python
# Create GUI
# Step 1: Import Data
if IS_PRODUCTION is True:
    tf_selector = TeamFilesSelector(
        team_id=TEAM_ID, multiple_selection=False, max_height=300, selection_file_type="folder"
    )
    data_card = Card(
        title="Select Data",
        description="Check folder in File Browser to import it",
        content=tf_selector,
    )
else:
    data_text = Text()
    if PATH_TO_FOLDER is None:
        data_text.set("Please, specify path to folder with data in local.env file.", "error")
    else:
        if os.path.isdir(PATH_TO_FOLDER):
            data_text.set(f"Folder with data: '{PATH_TO_FOLDER}'", "success")
        else:
            data_text.set(f"Folder with data: '{PATH_TO_FOLDER}' not found", "error")
    data_card = Card(
        title="Local Data", description="App was launched in development mode.", content=data_text
    )

# Step 2: Settings
remove_source_files = Checkbox("Remove source files after successful import", checked=True)
settings_card = Card(
    title="Settings", description="Select import settings", content=remove_source_files
)

# Step 3: Create Project
ws_selector = SelectWorkspace(default_id=WORKSPACE_ID, team_id=TEAM_ID)
output_project_name = Input(value="My Project")
project_creator = Container(widgets=[ws_selector, output_project_name])
project_card = Card(
    title="Create Project",
    description="Select destination team, workspace and enter project name",
    content=project_creator,
)
# Step 4: Output
start_import_btn = Button(text="Start Import")
output_project_thumbnail = ProjectThumbnail()
output_project_thumbnail.hide()
output_text = Text()
output_text.hide()
output_progress = SlyTqdm()
output_progress.hide()
output_container = Container(
    widgets=[output_project_thumbnail, output_text, output_progress, start_import_btn]
)
output_card = Card(
    title="Output", description="Press button to start import", content=output_container
)
# create app object
layout = Container(widgets=[data_card, settings_card, project_card, output_card])
app = sly.Application(layout=layout)
```

**Step 5. Add button click handler to start import process**

In this step we will create button click handler.
We will get state of all widgets and import data to new project.

```python
@start_import_btn.click
def start_import():
    try:
        data_card.lock()
        settings_card.lock()
        project_card.lock()
        output_text.hide()
        project_name = output_project_name.get_value()
        if project_name is None or project_name == "":
            output_text.set(text="Please, enter project name", status="error")
            output_text.show()
            return

        # download folder from Supervisely Team Files to local storage if debugging in production mode
        PATH_TO_FOLDER = tf_selector.get_selected_paths()
        if len(PATH_TO_FOLDER) > 0:
            PATH_TO_FOLDER = PATH_TO_FOLDER[0]
            # specify local path to download
            local_data_path = os.path.join(
                STORAGE_DIR, os.path.basename(PATH_TO_FOLDER).lstrip("/")
            )
            # download file from Supervisely Team Files to local storage
            api.file.download_directory(
                team_id=TEAM_ID, remote_path=PATH_TO_FOLDER, local_save_path=local_data_path
            )
        else:
            output_text.set(
                text="Please, specify path to folder in Supervisely Team Files", status="error"
            )
            output_text.show()
            return
        project = api.project.create(WORKSPACE_ID, project_name, change_name_if_conflict=True)
        dataset = api.dataset.create(project.id, "ds0", change_name_if_conflict=True)
        output_progress.show()
        images_names = []
        images_paths = []
        for file in os.listdir(local_data_path):
            file_path = os.path.join(local_data_path, file)
            images_names.append(file)
            images_paths.append(file_path)

        with output_progress(total=len(images_paths)) as pbar:
            for img_name, img_path in zip(images_names, images_paths):
                try:
                    # upload image into dataset on Supervisely server
                    info = api.image.upload_path(dataset_id=dataset.id, name=img_name, path=img_path)
                    sly.logger.trace(f"Image has been uploaded: id={info.id}, name={info.name}")
                except Exception as e:
                    sly.logger.warn("Skip image", extra={"name": img_name, "reason": repr(e)})
                finally:
                    # update progress bar
                    pbar.update(1)
        # remove source files from Supervisely Team Files if checked
        if remove_source_files.is_checked():
            api.file.remove_dir(TEAM_ID, PATH_TO_FOLDER)
        # hide progress bar after import
        output_progress.hide()
        
        # update project info for thumbnail preview
        project = api.project.get_info_by_id(project.id)
        output_project_thumbnail.set(info=project)
        output_project_thumbnail.show()
        output_text.set(text="Import is finished", status="success")
        output_text.show()
        start_import_btn.disable()
        sly.logger.info(f"Result project: id={project.id}, name={project.name}")
    except Exception as e:
        data_card.unlock()
        settings_card.unlock()
        project_card.unlock()
        raise DialogWindowError(title="Import error", description=f"Error: {e}")

```

**App screenshot**

<img src="https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/assets/48913536/567a34e7-3885-4c70-9b81-68aab54abadc">

## Step 3. Advanced debug

Advanced debug is for final app testing. In this case, import app will download selected folder with data from Supervisely server. You can use this mode to test your app before [publishing it to the Ecosystem](https://developer.supervisely.com/getting-started/cli#release-your-private-apps-using-cli).

To switch between local and advanced debug modes, select corresponding debug configuration in **`Run & Debug`** menu in VS Code

<img src="https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/assets/48913536/4b37f3a4-d1b0-4c23-8f5d-761bcd601d20">

Open `advanced.env` and set up [environment variables](../../getting-started/environment-variables.md) by inserting your values here for debugging.

**advanced.env:**

```python
TEAM_ID=8                    # ⬅️ change it to your team ID
WORKSPACE_ID=349             # ⬅️ change it to your workspace ID
SLY_APP_DATA_DIR="results/"  # ⬅️ path to directory where selected data will be downloaded
```

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for saving application results and temporary files.

For example:
- path on your local computer could be `/Users/admin/projects/import-app-from-scratch-gui/results/`
- path in the current project folder on your local computer could be `results/`

Also note that all paths on Supervisely server are absolute and start from '/' symbol, so you need to specify the full path to the folder, for example `/data/my_folder/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

![Advanced debug](https://github.com/supervisely-ecosystem/import-app-from-scratch-gui/assets/48913536/d1794539-dff7-4c2e-959c-8a7a8db1a87c)
