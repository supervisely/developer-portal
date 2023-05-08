---
description: >-
  Step-by-step tutorial of how to create custom export app from template.
---

# Create export app from template

## Introduction

In this tutorial, you will learn how to create custom export app for exporting your data from Supervisely platform using an [export template app](https://github.com/supervisely-ecosystem/template-export-app) that we have prepared for you.

We advise reading our [from script to supervisely app](../../basics/from-script-to-supervisely-app.md) guide if you are unfamiliar with the [file structure](../../basics/from-script-to-supervisely-app.md#repository-structure) of a Supervisely app repository because it addresses the majority of the potential questions.

We will go through the following steps:

[**Step 1.**](#step-1-set-up-an-environment-for-development) Set up an environment for development.

[**Step 2.**](#step-2-how-to-debug-export-app) How to debug export app.

[**Step 3.**](#step-3-simple-illustrative-example) Simple illustrative example.

[**Step 4**](#step-4-how-to-write-an-export-script) How to write an export script.

[**Step 5.**](#step-5-advanced-debug) Advanced debug.

[**Step 6.**](#step-6-how-to-run-it-in-supervisely) How to run it in Supervisely.

Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/template-export-app): source code and additional app files.

## Step 1. Set up an environment for development

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#how-to-use-in-python)

**Step 2.** Fork and clone [repository](https://github.com/supervisely-ecosystem/template-export-app) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/template-export-app
cd template-export-app
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Select created virtual environment as python interpreter.

**Step 5.** Open `local.env` and insert your values here. Learn more about environment variables in our [guide](../../../getting-started/environment-variables.md)

```python
TASK_ID=10                    # ⬅️ requires to use advanced debugging
TEAM_ID=447                   # ⬅️ change it
WORKSPACE_ID=680              # ⬅️ change it
PROJECT_ID=20934              # ⬅️ ID of the project that you want to export
DATASET_ID=64985              # ⬅️ ID of the dataset that you want to export (leave empty if you want to export whole project)
SLY_APP_DATA_DIR="results/"   # ⬅️ path to directory for local debugging
```

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for saving temporary files (they will be removed at the end) and application results. 

For example:
- path on your local computer could be `/Users/admin/Downloads/`
- path in the current project folder on your local computer could be `results/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

![Change variables in local.env](https://user-images.githubusercontent.com/79905215/236182190-3438d72e-919f-4a8f-9544-a105e8441a5a.gif)


When running the app from Supervisely platform: Project and Dataset IDs will be automatically detected depending on how you run your application.

## Step 2. How to debug export app

In this tutorial, we will be using the **Run & Debug** section of the VSCode to debug our export app.

The export template has 2 launch options for debugging: `Debug` and `Advanced Debug`. 
The settings for these options are configured in the `launch.json` file. Lets start from oprtion #1 - `Debug`

![launch.json](https://user-images.githubusercontent.com/79905215/236759275-c3200ca1-a176-44c0-9cdb-9218db39da74.png)

This option is a good starting point. In this case, the resulting archive or folder with the exported data will remain on your computer and be saved in the path that we defined in the `local.env` file (`SLY_APP_DATA_DIR="results/"`).

![Debug](https://user-images.githubusercontent.com/79905215/236765763-daef4e90-ce89-4bc3-9037-cbbf39c64902.gif)


Output of this python program:

```text
{"message": "Exporting Project: id=20934, name=Model predictions, type=images", "timestamp": "2023-05-08T11:30:06.341Z", "level": "info"}
{"message": "Exporting Dataset: id=64895, name=Week # 1", "timestamp": "2023-05-08T11:30:06.651Z", "level": "info"}
Processing: 100%|████████████████████████████████████████████████████████████████████████████████████| 6/6 [00:06<00:00,  1.12s/it]
```

## Step 3. Simple (illustrative) example

In this exmaple app we will convert selected project or dataset to the following format and prepares downloadable .tar archive:

- original images
- annotations in json format

Output example:

```text
<id_project_name>.tar
- ds0
  - image_1.jpg
  - image_2.jpg
  - labels.json
- ds1
  - image_1.jpg
  - image_2.jpg
  - labels.json
```

Labels.json:

Contain bounding box coordinates(top, left, right, bottom) of all objects in project or dataset

```text
{
    "image_1.jpg": [
        {
            "class_name": "cat",
            "coordinates": [top, left, right, bottom]
        },
        {
            "class_name": "cat",
            "coordinates": [top, left, right, bottom]
        },
        {
            "class_name": "dog",
            "coordinates": [top, left, right, bottom]
        },
    ]
}

```

{% hint style="info" %}
You can add your own custom script instead of the provided example by reimplementing the "process" method and returning the path to the directory you want to download. This will allow you to customize the format and contents of the downloadable archive according to your specific needs.
{% endhint %}

## Step 4. How to write an export script

You can find source code for this example [here](https://github.com/supervisely-ecosystem/template-export-app/blob/master/src/main.py)

**Step 1. Import libraries**

```python
import json, os
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

**Step 3. Write Export script**

Create a class that inherits from `sly.app.Export` and write `process` method that will handle the `team_id`, `workspace_id`, `project_id` and `dataset_id` that you specified in the `local.env`. In this example our class called `MyExport`.

`sly.app.Export` class will handle export routines for you:

- it will check that selected project or dataset exist and that you have access to work with it,
- it will upload your result data to Team Files and clean temporary folder, containing result archive in remote container or local hard drive if you are debugging your app. 
- Your application must return string, containing path to result archive or folder. If you return path to folder - this folder will be automatically archived.

`sly.app.Export` has a `Context` subclass which contains all required information that you need for exporting your data from Supervisely platform:

- `Team ID` - shows team id where exporting project or dataset is located
- `Workspace ID` - shows workspace id where exporting project or dataset is located
- `Project ID` - id of exporting project
- `Dataset ID` - id of exporting dataset (detected only if the export is performed from dataset context menu)

`context` variable is passed as an argument to `process` method of class `MyExport` and `context` object will be created automatically when you execute export script.

```python
class MyExport(sly.app.Export):
    def process(self, context: sly.app.Export.Context):
        print(context)
```

Output:

```text
Team ID: 1
Workspace ID: 1
Project ID: 555
Dataset ID: 55555
```

Now let's get to the code part

```python
STORAGE_DIR = sly.app.get_data_dir() # path to directory for temp files and result archive
ANN_FILE_NAME = "labels.json"

class MyExport(sly.app.Export):
    def process(self, context: sly.app.Export.Context):
        # create api object to communicate with Supervisely Server
        api = sly.Api.from_env()

        # get project info from server
        project_info = api.project.get_info_by_id(id=context.project_id)

        # make project directory path
        data_dir = os.path.join(STORAGE_DIR, f"{project_info.id}_{project_info.name}")

        # get project meta
        meta_json = api.project.get_meta(id=context.project_id)
        project_meta = sly.ProjectMeta.from_json(meta_json)

        # Check if the app runs from the context menu of the dataset. 
        if context.dataset_id is not None:
            # If so, get the dataset info from the server.
            dataset_infos = [api.dataset.get_info_by_id(context.dataset_id)]
        else:
            # If it does not, obtain all datasets infos from the current project.
            dataset_infos = api.dataset.get_list(context.project_id)

        # iterate over datasets in project
        for dataset in dataset_infos:
            result_anns = {}

            # get dataset images info
            images_infos = api.image.get_list(dataset.id)

            # track progress using Tqdm
            with tqdm(total=dataset.items_count) as pbar:
                # iterate over images in dataset
                for image_info in images_infos:
                    labels = []

                    # create path for each image and download it from server
                    image_path = os.path.join(data_dir, dataset.name, image_info.name)
                    api.image.download(image_info.id, image_path)

                    # download annotation for current image
                    ann_json = api.annotation.download_json(image_info.id)
                    ann = sly.Annotation.from_json(ann_json, project_meta)

                    # iterate over labels in current annotation
                    for label in ann.labels:
                        # get obj class name
                        name = label.obj_class.name

                        # get bounding box coordinates for label
                        bbox = label.geometry.to_bbox()
                        labels.append(
                            {
                                "class_name": name,
                                "coordinates": [
                                    bbox.top,
                                    bbox.left,
                                    bbox.bottom,
                                    bbox.right,
                                ],
                            }
                        )

                    result_anns[image_info.name] = labels

                    # increment the current progress counter by 1
                    pbar.update(1)

            # create JSON annotation in new format
            filename = os.path.join(data_dir, dataset.name, ANN_FILE_NAME)
            with open(filename, "w") as file:
                json.dump(result_anns, file, indent=2)

        return data_dir
```

Create `MyExport` object and execute `run` method to start export

```python
app = MyExport()
app.run()
```

## Step 5. Advanced debug
 
In addition to the regular debug option, this template also includes setting for `Advanced debugging`.

![launch.json](https://user-images.githubusercontent.com/79905215/236436739-9bc2192d-e34f-4630-bf63-5ab184710526.png)

The advanced debugging option is somewhat identical, however it will upload result archive or folder with data to `Team Files` instead (Path to result archive - /tmp/supervisely/export/Supervisely App/<SESSION ID>/<PROJECT_ID>_<PROJECT_NAME>.tar).
This option is an example of how production apps work in Supervisely platform.

![Advanced debug](https://user-images.githubusercontent.com/79905215/236766557-34031634-6284-4714-a589-43dd1e5c456a.gif)

Output of this python program:

```text
{"message": "App data directory results/ doesn't exist. Will be made automatically.", "timestamp": "2023-05-08T10:39:15.626Z", "level": "info"}
{"message": "Exporting Project: id=20934, name=Model predictions, type=images", "timestamp": "2023-05-08T10:39:17.918Z", "level": "info"}
{"message": "Exporting Dataset: id=64895, name=Week # 1", "timestamp": "2023-05-08T10:39:18.209Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 0, "total": 6, "timestamp": "2023-05-08T10:39:19.478Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 1, "total": 6, "timestamp": "2023-05-08T10:39:20.592Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 6, "total": 6, "timestamp": "2023-05-08T10:39:25.918Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Uploading '20934_Model predictions.tar'", "current": 0, "total": 4567476, "current_label": "0.0 B", "total_label": "4.4 MiB", "timestamp": "2023-05-08T10:39:26.135Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Uploading '20934_Model predictions.tar'", "current": 1048576, "total": 4567476, "current_label": "1.0 MiB", "total_label": "4.4 MiB", "timestamp": "2023-05-08T10:39:26.676Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Uploading '20934_Model predictions.tar'", "current": 4567476, "total": 4567476, "current_label": "4.4 MiB", "total_label": "4.4 MiB", "timestamp": "2023-05-08T10:39:36.578Z", "level": "info"}
{"message": "Remote file: id=1273718, name=20934_Model predictions.tar", "timestamp": "2023-05-08T10:39:37.451Z", "level": "info"}
```

## Step 6. How to run it in Supervisely

Submitting an app to the Supervisely Ecosystem isn’t as simple as pushing code to github repository, but it’s not as complicated as you may think of it either.

Please follow this [link](../../basics/add-private-app.md) for instructions on adding your app. We have produced a step-by-step guide on how to add your application to the Supervisely Ecosystem.

![Release custom export app](https://user-images.githubusercontent.com/79905215/236189327-6a3ac061-2cb6-4614-84ec-8d4196f75582.gif)
