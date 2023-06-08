---
description: >-
    A step-by-step tutorial of how to create custom import app using import template from SDK class `sly.app.Import`.
---

# Create import app from template

## Introduction

In this tutorial, we will create a simple import app that will upload images from a folder, archive or text file to Supervisely using import app template from SDK with a following structure:

**folder and archive structure:**

```text
📂my_folder         🗃️ my_archive.zip
┣ 🖼️cat_1.jpg       ┣ 🖼️cat_1.jpg
┣ 🖼️cat_2.jpg       ┣ 🖼️cat_2.jpg
┗ 🖼️cat_3.jpg       ┗ 🖼️cat_3.jpg
```

**text file:**

```text
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-couleur-2317904.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-kammeran-gonzalezkeola-7925859.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-stijn-dijkstra-7177188.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-taryn-elliott-3889728.jpg
https://github.com/supervisely-ecosystem/demo-data-for-import-template/releases/download/images/pexels-taryn-elliott-9565787.jpg/home/paul/projects/template-import-app/results
```

You can find the above demo folder in the data directory of the template-import-app repo - [here](https://github.com/supervisely-ecosystem/template-import-app/blob/master/data/)

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/de4a23b0-0c86-45f8-8431-e292bf9ecdf9">

**We will go through the following steps:**

[**Step 1.**](#step-1-how-to-debug-import-app) How to debug import app.

[**Step 2**](#step-2-how-to-write-an-import-script) How to write an import script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

[**Step 4.**](#step-4-how-to-run-it-in-supervisely) How to run it in Supervisely.

Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/template-import-app): [source code](https://github.com/supervisely-ecosystem/template-import-app/blob/master/src/main.py).

Before we begin, please clone the project and set up the working environment - [here is a link with a description of the steps](/README.md#set-up-an-environment-for-development).

## Step 1. How to debug import app

Open `local.env` and `advanced.env` files and set up environment variables by inserting your values here for debugging.

Learn more about environment variables in our [guide](https://developer.supervisely.com/getting-started/environment-variables)

For this example, we will use the following environment variables:

**local.env:**

```python
TEAM_ID=8                      # ⬅️ change it to your team ID
WORKSPACE_ID=349               # ⬅️ change it to your workspace ID

# Optional. Specify one of the following variables:
FOLDER="data/my_folder"        # ⬅️ path to directory with data
# FILE="data/my_archive.zip"   # ⬅️ path to archive with data
# FILE="data/my_file.txt"      # ⬅️ path to text file with links to images

# Optional. Specify following variables if you want to import data to existing:
# PROJECT_ID=20811             # ⬅️ put your value here
# DATASET_ID=64686             # ⬅️ put your value here | requires PROJECT_ID
```

**advanced.env:**

```python
TASK_ID=35038                     # ⬅️ requires to use advanced debugging
TEAM_ID=8                         # ⬅️ change it to your team ID
WORKSPACE_ID=349                  # ⬅️ change it to your workspace ID
SLY_APP_DATA_DIR="results/"       # ⬅️ path to directory for local debugging

# Optional. Specify one of the following variables if you want to simulate import from:
# FOLDER="/data/my_folder"      # ⬅️ path to directory with data
# FILE="/data/my_archive.zip"   # ⬅️ path to archive with data
# FILE="data/my_file.txt"       # ⬅️ path to text file with links to images

# or one of the following variables if you want to import data to existing:
# PROJECT_ID=20811              # ⬅️ put your value here
# DATASET_ID=64686              # ⬅️ put your value here | requires PROJECT_ID
```

In order to get `TASK_ID` you need to run [`While True Script`](https://ecosystem.supervisely.com/apps/while-true-script) app on the agent specifed in `supervisely.env`

<!-- SCREENSHOT GIF HERE -->

![copy-task-id](https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/e6a6ac90-b239-47fd-b099-90fd51e818dd)

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for saving application results and temporary files (temporary files will be removed at the end).

For example:
- path on your local computer could be `/Users/admin/Downloads/`
- path in the current project folder on your local computer could be `results/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

## Step 2. How to write an import script

Find source code for this example [here](https://github.com/supervisely-ecosystem/template-import-app/blob/master/src/main.py)

**Step 1. Import libraries**

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from tqdm import tqdm
```

**Step 2. Load environment variables**

Load ENV variables for debug, has no effect in production

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
```

**Step 3. Create class MyImport that inherits from sly.app.Import with process method**

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        ...
```

**Step 4. Reimplement process method**

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        # create api object to communicate with Supervisely Server
        api = sly.Api.from_env()

        # get or create project
        project_id = context.project_id
        if project_id is None:
            project = api.project.create(
                workspace_id=context.workspace_id, name="My Project", change_name_if_conflict=True
            )
            project_id = project.id

        # get or create dataset
        dataset_id = context.dataset_id
        if dataset_id is None:
            dataset = api.dataset.create(
                project_id=project_id, name="ds0", change_name_if_conflict=True
            )
            dataset_id = dataset.id

        # list images in directory
        images_names = []
        images_paths = []
        for file in os.listdir(context.path):
            file_path = os.path.join(context.path, file)
            images_names.append(file)
            images_paths.append(file_path)

        # process images and upload them by paths
        with tqdm(total=len(images_paths)) as pbar:
            for img_name, img_path in zip(images_names, images_paths):
                try:
                    # upload image into dataset on Supervisely server
                    info = api.image.upload_path(
                        dataset_id=dataset_id, name=img_name, path=img_path
                    )
                    sly.logger.trace(f"Image has been uploaded: id={info.id}, name={info.name}")
                except Exception as e:
                    sly.logger.warn("Skip image", extra={"name": img_name, "reason": repr(e)})
                finally:
                    pbar.update(1)

        return project_id
```

**Step 5. Create app object and execute run() method**

```python
app = MyImport()
app.run()
```

**Output of this python program in local debug mode:**

```text
{"message": "Application is running on localhost in development mode", "timestamp": "2023-06-08T12:11:38.929Z", "level": "info"}
{"message": "Application PID is 29901", "timestamp": "2023-06-08T12:11:38.929Z", "level": "info"}
100%|██████████████████████████████████████████████████████████████████████████████████████████████| 3/3 [00:01<00:00,  1.97it/s]
{"message": "Result project: id=22913, name=My Project_004", "timestamp": "2023-06-08T12:11:42.674Z", "level": "info"}
```

## Step 3. Advanced debug

Advanced debug is for final testing and debugging. In this case, data will be downloaded from Supervisely instance Team Files and uploaded to specified project or dataset on Supervisely platform, source data will be removed if specified.

![Advanced debug](https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/aab1c0dd-a4a5-41ab-bab0-b5691460a55b)

Output of this python program:

```text
{"message": "Application is running on Supervisely Platform in production mode", "timestamp": "2023-05-10T14:17:57.194Z", "level": "info"}
{"message": "Application PID is 19320", "timestamp": "2023-05-10T14:17:57.194Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 0, "total": 3, "timestamp": "2023-05-10T14:18:01.261Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 3, "total": 3, "timestamp": "2023-05-10T14:18:04.766Z", "level": "info"}
{"message": "Result project: id=21417, name=My Project", "timestamp": "2023-05-10T14:18:05.958Z", "level": "info"}
{"message": "Shutting down [pid argument = 19320]...", "timestamp": "2023-05-10T14:18:05.958Z", "level": "info"}
{"message": "Application has been shut down successfully", "timestamp": "2023-05-10T14:18:05.959Z", "level": "info"}
```
