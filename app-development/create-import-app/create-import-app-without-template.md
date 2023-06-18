---
description: >-
  A step-by-step tutorial of how to create custom import Supervisely app from scratch.
---

# Create import Supervisely app from scratch

## Introduction

In this tutorial, we will create a simple import app that will import images from selected folder to Supervisely server.
This application is headless (no GUI) and is designed to demonstrate the basic principles of creating minimalistic import applications.

## Data example

```text
📂my_folder
┣ 🖼️cat_1.jpg
┣ 🖼️cat_2.jpg
┗ 🖼️cat_3.jpg
```

You can find the above demo files in the data directory of the template-import-app repo - [here](https://github.com/supervisely-ecosystem/import-app-from-scratch/blob/master/data/)

<img src="https://github.com/supervisely-ecosystem/import-app-from-scratch/assets/48913536/ae076053-c904-434b-a2c8-0dd01a7694ba">

## Tutorial content

[**Step 1.**](#step-1-how-to-debug-import-app) How to debug import app.

[**Step 2.**](#step-2-how-to-write-import-script) How to write import script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

Everything you need to reproduce this tutorial is on [GitHub](https://github.com/supervisely-ecosystem/import-app-from-scratch): [main.py](https://github.com/supervisely-ecosystem/import-app-from-scratch/blob/master/src/main.py).

Before we begin, please clone the project and set up the working environment - [here is a link with a description of the steps](./overview.md#set-up-an-environment-for-the-development).

## Step 1. How to debug import app

Open `local.env` and set up environment variables by inserting your values here for debugging. Learn more about environment variables in our [guide](../../getting-started/environment-variables.md)

**local.env:**

```python
TEAM_ID=8                    # ⬅️ change it to your team ID
WORKSPACE_ID=349             # ⬅️ change it to your workspace ID
FOLDER="data/my_folder/"     # ⬅️ path to folder on local machine
```

## Step 2. How to write import script

Find source code for this example - [main.py](https://github.com/supervisely-ecosystem/import-app-from-scratch/blob/master/src/main.py)

**Step 1. Import libraries**

```python
import os

import supervisely as sly
from dotenv import load_dotenv

from tqdm import tqdm
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
PATH_TO_FOLDER = sly.env.folder()
```

**Step 3. Initialize API object**

Create API object to communicate with Supervisely Server and initialize application.
Loads from `supervisely.env` file

```python
# Create api object to communicate with Supervisely Server
api = sly.Api.from_env()

# Initialize application
app = sly.Application()
```

**Step 4. Create new project and dataset on Supervisely server**

```python
project = api.project.create(WORKSPACE_ID, "My Project", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "ds0", change_name_if_conflict=True)
```

**Step 5. Download data from Supervisely server**

Check if app was launched in production mode and download data from Supervisely server

```python
# Download folder from Supervisely server
if IS_PRODUCTION is True:
    api.file.download_directory(TEAM_ID, PATH_TO_FOLDER, STORAGE_DIR)
    # Set path to folder with images
    PATH_TO_FOLDER = STORAGE_DIR
```

**Step 6. List files in directory**

Get list of files in directory and create list of images names and paths

```python
images_names = []
images_paths = []
for file in os.listdir(PATH_TO_FOLDER):
    file_path = os.path.join(PATH_TO_FOLDER, file)
    images_names.append(file)
    images_paths.append(file_path)
```

**Step 7. Upload images to new project**

```python
# Process folder with images and upload them to Supervisely server
with tqdm(total=len(images_paths)) as pbar:
    for img_name, img_path in zip(images_names, images_paths):
        try:
            # Upload image into dataset on Supervisely server
            info = api.image.upload_path(dataset_id=dataset.id, name=img_name, path=img_path)
            sly.logger.trace(f"Image has been uploaded: id={info.id}, name={info.name}")
        except Exception as e:
            sly.logger.warn("Skip image", extra={"name": img_name, "reason": repr(e)})
        finally:
            # Update progress bar
            pbar.update(1)

# Log info about result project
sly.logger.info(f"Result project: id={project.id}, name={project.name}")
```

**Output of the app in development mode:**

```text
{"message": "Application is running on localhost in development mode", "timestamp": "2023-06-09T17:06:12.671Z", "level": "info"}
{"message": "Application PID is 11269", "timestamp": "2023-06-09T17:06:12.671Z", "level": "info"}
Processing: 100%|██████████████████████████████████████████████████████████████████████████| 3/3 [00:01<00:00,  2.02it/s]
{"message": "Result project: id=22962, name=My Project_004", "timestamp": "2023-06-09T17:06:16.139Z", "level": "info"}
```

## Step 3. Advanced debug

Advanced debug is for final app testing. In this case, import app will download data from Supervisely server. You can use this mode to test your app before [publishing it to the Ecosystem](../basics/add-private-app.md).

To switch between local and advanced debug modes, select corresponding debug configuration in **`Run & Debug`** menu in VS Code

<img src="https://github.com/supervisely-ecosystem/import-app-from-scratch/assets/48913536/f191f0f3-43be-451a-8787-5ada0b9b74f9">

Open `advanced.env` and set up [environment variables](../../getting-started/environment-variables.md) by inserting your values here for debugging.

**advanced.env:**

```python
TEAM_ID=8                              # ⬅️ change it to your team ID
WORKSPACE_ID=349                       # ⬅️ change it to your workspace ID
FOLDER="/data/my_folder/"              # ⬅️ path to folder on Supervisely server
SLY_APP_DATA_DIR="input/"              # ⬅️ path to directory for local debugging
```

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for storing import data.

For example:
- path on your local computer could be `/Users/admin/projects/import-app-from-scratch/input/`
- path in the current project folder on your local computer could be `input/`

Also note that all paths on Supervisely server are absolute and start from '/' symbol, so you need to specify the full path to the folder, for example `/data/my_folder/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

**Output of the app in production mode:**

```text
{"message": "Application is running on Supervisely Platform in production mode", "timestamp": "2023-06-09T16:12:40.673Z", "level": "info"}
{"message": "Application PID is 10646", "timestamp": "2023-06-09T16:12:40.673Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 0, "total": 3, "timestamp": "2023-06-09T16:12:42.911Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 3, "total": 3, "timestamp": "2023-06-09T16:12:44.355Z", "level": "info"}
```
