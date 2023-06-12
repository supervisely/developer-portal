---
description: >-
    A step-by-step tutorial of how to create custom import app using import template from SDK class `sly.app.Import`.
---

# Create import app from template

## Introduction

In this tutorial, we will create a simple import app that will upload images from a folder, archive or `.txt` file to Supervisely using import app template from SDK with a following structure:

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

**We will go through the following steps:**

[**Step 1.**](#step-1-how-to-debug-import-app) How to debug import app.

[**Step 2**](#step-2-how-to-write-an-import-script) How to write an import script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

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

Upload [demo data](https://github.com/supervisely-ecosystem/template-import-app/blob/master/data/) provided in the repository to Supervisely Team Files in order to use it in the app.

```python
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

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for saving application results and temporary files (temporary files will be removed at the end).

For example:
- path on your local computer could be `/Users/admin/Downloads/`
- path in the current project folder on your local computer could be `results/`

Also note that all paths in Supervisely server are absolute and start from '/' symbol, so you need to specify the full path to the folder, for example `/data/my_folder/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

To switch between local and advanced debug modes, select corresponding debug configuration in **`Run & Debug`** menu in VS Code

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/d77bb7a1-063a-4045-8d73-f303dc17d452">

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

**Step 3. Create class MyImport that inherits from sly.app.Import with process method**

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        ...
```

**Step 4. Reimplement process method**

1. Create api object to communicate with Supervisely Server
2. Get or create project and dataset
3. Process data based on it's type: folder, archive, text file with links to images (see step 5)
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

        if context.is_directory:
            images_names, images_paths = process_folder(context.path)
        else:
            if sly.fs.get_file_ext(context.path) == ".txt":
                images_names, images_paths = process_text_file(context.path)
            else:
                images_names, images_paths = process_archive(context.path)
        upload_images(api, dataset_id, images_names, images_paths, context.progress)
        return project_id
```

**Step 5. Write methods to process data**

Process folder with images

```python
def process_folder(path_to_folder):
    # list images in directory
    images_names = []
    images_paths = []
    for file in os.listdir(path_to_folder):
        file_path = os.path.join(path_to_folder, file)
        images_names.append(file)
        images_paths.append(file_path)
    return images_names, images_paths
```

Process archive with images

```python
def process_archive(path_to_archive):
    local_data_dir = os.path.join(sly.app.get_data_dir(), sly.fs.get_file_name(path_to_archive))
    shutil.unpack_archive(path_to_archive, extract_dir=local_data_dir)
    images_names, images_paths = process_folder(local_data_dir)
    return images_names, images_paths
```

Process text file with links to images and download them to local data directory

```python
def process_text_file(path_to_file, progress):
    # read input file, remove empty lines + leading & trailing whitespaces
    with open(path_to_file) as file:
        lines = [line.strip() for line in file.readlines() if line.strip()]

    # process text file and download links
    images_names, images_paths = [], []
    with progress(total=len(lines)) as pbar:
        for index, img_url in enumerate(lines):
            try:
                img_ext = Path(img_url).suffix
                img_name = f"{index:03d}{img_ext}"
                img_path = os.path.join(sly.app.get_data_dir(), img_name)
                # download image
                response = requests.get(img_url)
                with open(img_path, "wb") as file:
                    file.write(response.content)

                images_names.append(img_name)
                images_paths.append(img_path)
            except Exception as e:
                sly.logger.warn("Skip image", extra={"url": img_url, "reason": repr(e)})
            finally:
                pbar.update(1)
    return images_names, images_paths
```

**Step 6. Write method to upload images to dataset**

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

**App screenshot**

<img src="https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/2910c05d-92c3-496b-9583-2cea8affd3b1">

## Step 3. Advanced debug

Advanced debug is for final app testing. In this case, import app will download data from Supervisely server and upload images to new project. You can use this mode to test your app before [publishing it to the Ecosystem](https://developer.supervisely.com/getting-started/cli#release-your-private-apps-using-cli).

![Advanced debug](https://github.com/supervisely-ecosystem/template-import-app/assets/48913536/7ba8b1c6-d1f0-4423-bb82-81710b143a93)
