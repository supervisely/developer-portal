---
description: >-
    A step-by-step tutorial of how to create custom import app using import template from SDK class `sly.app.Import`.
---

# Create import app from template

## Introduction

In this tutorial, we will create a simple import app that will upload images from a folder, archive or `.txt` file to Supervisely using import app template from SDK.

**About `sly.app.Import` class**

Import template has GUI out of the box and allows you skip boilerplate/routine operations. The only thing you need to do is to code your logic in the `process` method. Import template is customizable and allows you to tail import app to your needs.

**GUI consists of 4 steps:**

1. Select data to import
2. Select import settings
3. Select destination team, workspace and project
4. Output information

**App screenshot**

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/2910c05d-92c3-496b-9583-2cea8affd3b1">

You can customize `sly.app.Import` class by passing parameters to the constructor:

**allowed_project_types**

Pass list of project types that you want to allow for import. If you pass None, all project types will be allowed in project selector.
Available project types: `["images", "videos", "volumes", "pointclouds", "pointcloud_episodes"]`

```python
app = MyImport(allowed_project_types=[sly.ProjectType.Volumes])
```

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/437cf96b-147a-4f71-adeb-488577c63305">

**allowed_destination_options**

Pass list of destination options that you want to allow for import. If you pass None, all destination options will be allowed.
Allowed destinations: `["new_project", "existing_project", "existing_dataset"]`

```python
app = MyImport(allowed_destination_options=["New Project"])
```

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/7d9ad0ad-3914-48bd-90c3-87e920dfda10">

```python
app = MyImport(allowed_destination_options=["New Project", "Existing Project"])

```

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/f676b45b-9cd7-4339-abee-92f0076ad801">

**allowed_data_type**

Pass list of data types that you want to allow for import. If you pass None, all data types will be allowed.
Allowed data types: `["folder", "file"]`

```python
app = MyImport(allowed_data_type="folder")
```

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/d6b56204-a684-48fa-b880-b0616e10ab44">

**`sly.app.Import` class has 2 methods:**

* process()
* add_custom_settings()

**method process()**

Main method where you implement your import logic, process and upload data to Supervisely server. This method must return project ID

```python
def process(self, context: sly.app.Import.Context):
    # get or create project
    project_id = context.project_id
    if project_id is None:
        project = api.project.create(
            workspace_id=context.workspace_id,
            name=context.project_name or "My Project",
            change_name_if_conflict=True,
        )
        project_id = project.id
    # implement your import logic here
    return project_id
```

`context` is passed as an argument to the `process` method and the `context` object will be created automatically when you execute import script. `context` contains all the necessary information about the import process. 

You can get the following information from the context:

- `Team ID` - ID of the destination Team
- `Workspace ID` - ID of the destination Workspace
- `Project ID` - ID of an existing project to which data will be imported. None if you import data to a new project
- `Dataset ID` - ID of an existing dataset to which data will be imported. None if you import data to a new dataset
- `Path` - Path to your data on the local machine.
- `Project name` - name of the project provided in the GUI if import data to new project
- `Progress` - `tqdm` progress that you can use in your app to update progress bar
- `Is on agent` - Shows if your data is located on the agent or not

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        print(context)
        # implement your import logic here
```

Output:

```text
Team ID: 8
Workspace ID: 349
Project ID: 8534
Dataset ID: 22852
Path: /data/my_file.txt
Project name: ""
Is on agent: False
```

**method add_custom_settings()**

You can add custom settings to the import template using the [`add_custom_settings`](./overview.md#slyappimport-custom-settings-in-gui-of-your-app) method. This method should return any [Widget](../widgets/README.md). If you want to add multiple widgets, you can return a widget [`Container`](../widgets/layouts-and-containers/container.md).

```python
def add_custom_settings(self) -> List[Widget]:
    self.ann_checkbox = sly.app.widgets.Checkbox("Upload annotations", True)
    # return widget
    return self.ann_checkbox
```

## Data example

We prepared an app that can import images from: folder, archive or `.txt` file to Supervisely server.
We will review each case separately.

**folder and archive structure:**

```text
📂my_folder         🗃️ my_archive.zip
┣ 🖼️cat_1.jpg       ┣ 🖼️cat_1.jpg
┣ 🖼️cat_2.jpg       ┣ 🖼️cat_2.jpg
┗ 🖼️cat_3.jpg       ┗ 🖼️cat_3.jpg
```

**.txt file:**

```text
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-couleur-2317904.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-kammeran-gonzalezkeola-7925859.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-stijn-dijkstra-7177188.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-taryn-elliott-3889728.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-taryn-elliott-9565787.jpg/home/paul/projects/template-import-app/results
```

You can find the above demo folder in the data directory of the template-import-app repo - [here](https://github.com/supervisely-ecosystem/template-import-app/blob/master/data/)

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/de4a23b0-0c86-45f8-8431-e292bf9ecdf9">

## Tutorial content

[**Step 1.**](#step-1-how-to-debug-import-app) How to debug import app.

[**Step 2**](#step-2-how-to-write-an-import-script) How to write an import script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/template-import-app): [source code](https://github.com/supervisely-ecosystem/template-import-app/blob/master/src/main.py).

Before we begin, please clone the project and set up the working environment - [here is a link with a description of the steps](/README.md#set-up-an-environment-for-development).

## Step 1. How to debug import app

Open `local.env` files and set up environment variables by inserting your values here for debugging.

Put data that you want to import to the folder specified in `SLY_APP_DATA_DIR`. This tutorial comes with demo data that you can use for debugging. You can find it in the `data` directory of the template-import-app repo - see **[Data example](#data-example)** section.

For this example, we will use the following environment variables:

**local.env:**

```python
TEAM_ID=8                      # ⬅️ change it to your team ID
WORKSPACE_ID=349               # ⬅️ change it to your workspace ID
SLY_APP_DATA_DIR="input/"      # ⬅️ path to directory for local debugging

# Optional. Specify following variables if you want to import data to existing:
# PROJECT_ID=20811             # ⬅️ put your value here
# DATASET_ID=64686             # ⬅️ put your value here | requires PROJECT_ID
```

Learn more about environment variables in our [guide](https://developer.supervisely.com/getting-started/environment-variables)

For advanced debugging with GUI see **[Step 3](#step-3-advanced-debug)**

## Step 2. How to write an import script

Find source code for this example [here](https://github.com/supervisely-ecosystem/template-import-app/blob/master/src/main.py)

**Step 1. Import libraries**

```python
import os
import shutil
from pathlib import Path

import requests
import supervisely as sly
from dotenv import load_dotenv
```

**Step 2. Load environment variables**

Load ENV variables for debug, has no effect in production

```python
if sly.is_production():
    load_dotenv("advanced.env")
else:
    load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

```

**Step 3. Create class MyImport that inherits from `sly.app.Import` with process method**

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        ...
```

**Step 4. Reimplement process method**

1. Create api object to communicate with Supervisely Server
2. Get or create project and dataset
3. Process import data (see step 5)
4. Upload images to dataset (see step 6)
5. Return result project id

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        # create api object to communicate with Supervisely Server
        api = sly.Api.from_env()

        # get or create project
        project_id = context.project_id
        if project_id is None:
            project = api.project.create(
                workspace_id=context.workspace_id,
                name=context.project_name or "My Project",
                change_name_if_conflict=True,
            )
            project_id = project.id

        # get or create dataset
        dataset_id = context.dataset_id
        if dataset_id is None:
            dataset = api.dataset.create(
                project_id=project_id, name="ds0", change_name_if_conflict=True
            )
            dataset_id = dataset.id

        # process images and upload them by paths
        images_names, images_paths = process_data(context)
        upload_images(api, dataset_id, images_names, images_paths, context.progress)

        # clean local data dir after successful import
        sly.fs.remove_dir(context.path)
        return project_id
```

**Step 5. Write function to process data**

In this app example function `process_data` can process 3 types of data:

* folder - `process_folder` function
* archive - `process_archive` function
* `.txt` file with links to images - `process_text_file` function

For local debugging, put data that you want to import to the folder specified in `SLY_APP_DATA_DIR` in `local.env` file.
For advanced debugging, selected data is downloaded automatically to the folder specified in `SLY_APP_DATA_DIR` in `advanced.env` file.

Data processing logic is up to you. You can implement any logic that you need to process your data.
Please see [`main.py`](https://github.com/supervisely-ecosystem/template-import-app/blob/master/src/main.py) file for implementation details

```python
def process_data(context):
    path = os.path.join(context.path, os.listdir(context.path)[0])
    if os.path.isdir(path):
        images_names, images_paths = process_folder(path)
    elif sly.fs.get_file_ext(path) == ".txt":
        images_names, images_paths = process_text_file(path, context.progress)
    else:
        images_names, images_paths = process_archive(path)
    return images_names, images_paths
```

**Step 6. Write function to upload images to dataset**

```python
def upload_images(api, dataset_id, images_names, images_paths, progress):
    # process images and upload them by paths
    with progress(total=len(images_paths)) as pbar:
        for img_name, img_path in zip(images_names, images_paths):
            try:
                # upload image into dataset on Supervisely server
                info = api.image.upload_path(dataset_id=dataset_id, name=img_name, path=img_path)
                sly.logger.trace(f"Image has been uploaded: id={info.id}, name={info.name}")
            except Exception as e:
                sly.logger.warn("Skip image", extra={"name": img_name, "reason": repr(e)})
            finally:
                pbar.update(1)
```

**Step 7. Create app object and execute run() method**

```python
app = MyImport()
app.run()
```

## Step 3. Advanced debug

Advanced debug is for final app testing. In this case, import app will download data from Supervisely server and upload images to new project. You can use this mode to test your app before [publishing it to the Ecosystem](https://developer.supervisely.com/getting-started/cli#release-your-private-apps-using-cli).

To switch between local and advanced debug modes, select corresponding debug configuration in **`Run & Debug`** menu in VS Code

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/d77bb7a1-063a-4045-8d73-f303dc17d452">

**advanced.env:**

Upload [demo data](https://github.com/supervisely-ecosystem/template-import-app/blob/master/data/) provided in the repository to Supervisely Team Files in order to use it in the app or use your own data.

```python
TEAM_ID=8                         # ⬅️ change it to your team ID
WORKSPACE_ID=349                  # ⬅️ change it to your workspace ID
SLY_APP_DATA_DIR="input/"         # ⬅️ path to directory for local debugging

# Optional. Specify one of the following variables if you want to simulate import from:
# FOLDER="/data/my_folder"        # ⬅️ path to directory with data
# FILE="/data/my_archive.zip"     # ⬅️ path to archive with data
# FILE="/data/my_file.txt"        # ⬅️ path to text file with links to images

# or one of the following variables if you want to import data to existing:
# PROJECT_ID=20811               # ⬅️ put your value here
# DATASET_ID=64686               # ⬅️ put your value here | requires PROJECT_ID
```

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for storing import data, it means that data that you select or drag & drop in GUI to import will automatically be downloaded to this folder.

For example:
- path on your local computer could be `/Users/admin/projects/template-import-app/input/`
- path in the current project folder on your local computer could be `input/`

Also note that all paths in Supervisely server are absolute and start from '/' symbol, so you need to specify the full path to the folder, for example `/data/my_folder/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

![Advanced debug](https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/7ba8b1c6-d1f0-4423-bb82-81710b143a93)
