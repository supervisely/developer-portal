---
description: >-
  Step-by-step tutorial of how to create custom export app without using template (from scratch).
---

# Create export app from scratch without template 

## Introduction

[In the previous tutorial] we learned how to create an application for export custom data using our prepared template (`sly.app.Export`). You can review the description of this example at this [link](./create-export-app-from-template.md#step-2-overview-of-the-simple-illustrative-example-we-will-use-in-tutorial). In this lesson, we will look at how to create the same application from scratch without using our template (from stratch).

It is more recommended way to use SDK export template class (`sly.app.Export`) to create custom export app ([learn more here.](./create-export-app-from-template.md)).
However, if your use case is not covered by our export template, you can create your own app without the template. 

We advise reading our [from script to supervisely app](../basics/from-script-to-supervisely-app.md) guide if you are unfamiliar with the [file structure](../basics/from-script-to-supervisely-app.md#repository-structure) of a Supervisely app repository because it addresses the majority of the potential questions.

We will go through the following steps:

[**Step 1.**](#step-1-how-to-debug-export-app) How to debug export app.

[**Step 2**](#step-2-how-to-write-an-export-script) How to write an export script.

[**Step 3.**](#step-3-advanced-debug) Advanced debug.

[**Step 4.**](#step-4-how-to-run-it-in-supervisely) How to run it in Supervisely.

Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/export-custom-format): source code and additional app files.

## Step 1. How to debug export app

In this tutorial, we will be using the **Run & Debug** section of the VSCode to debug our export app.

The export template has 2 launch options for debugging: `Debug` and `Advanced Debug`. 
The settings for these options are configured in the `launch.json` file. Lets start from oprtion #1 - `Debug`

![launch.json](https://user-images.githubusercontent.com/79905215/236759275-c3200ca1-a176-44c0-9cdb-9218db39da74.png)

This option is a good starting point. In this case, the resulting archive or folder with the exported data will remain on your computer and be saved in the path that we defined in the `local.env` file (`SLY_APP_DATA_DIR="results/"`).

![Debug](https://user-images.githubusercontent.com/79905215/236843626-df94117a-889c-4321-9925-2985896f6f89.gif)


Output of this python program:

```text
{"message": "Exporting Project: id=20934, name=Model predictions, type=images", "timestamp": "2023-05-08T11:30:06.341Z", "level": "info"}
{"message": "Exporting Dataset: id=64895, name=Week # 1", "timestamp": "2023-05-08T11:30:06.651Z", "level": "info"}
Processing: 100%|████████████████████████████████████████████████████████████████████████████████████| 6/6 [00:06<00:00,  1.12s/it]
```


## Step 2. How to write an export script

You can find source code for this example [here](https://github.com/supervisely-ecosystem/export-custom-format/blob/master/src/main.py)

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

Get variables

```python
TASK_ID = sly.env.task_id()
TEAM_ID = sly.env.team_id()
WORKSPACE_ID = sly.env.workspace_id()
PROJECT_ID = sly.env.project_id()
DATASET_ID = sly.env.dataset_id(raise_not_found=False)
IS_PRODUCTION = sly.is_production()
```

**Step 3. Write Export script**

```python
STORAGE_DIR = sly.app.get_data_dir()  # path to directory for temp files and result archive
PROJECT_DIR = os.path.join(STORAGE_DIR, str(PROJECT_ID))  # project directory path
sly.io.fs.mkdir(PROJECT_DIR, True)
ANN_FILE_NAME = "labels.json"

app = sly.Application() # run app

# get project info from server
project_info = api.project.get_info_by_id(id=PROJECT_ID)

if project_info is None:
    raise ValueError(
        f"Project with ID: '{PROJECT_ID}' either doesn't exist, archived or you don't have access to it"
    )
sly.logger.info(
    f"Exporting Project: id={project_info.id}, name={project_info.name}, type={project_info.type}",
)

meta_json = api.project.get_meta(id=PROJECT_ID)
project_meta = sly.ProjectMeta.from_json(meta_json)

# Check if the app runs from the context menu of the dataset.
if DATASET_ID is not None:
    # If so, get the dataset info from the server.
    dataset_infos = [api.dataset.get_info_by_id(DATASET_ID)]
    if dataset_infos is None:
        raise ValueError(
            f"Dataset with ID: '{DATASET_ID}' either doesn't exist, archived or you don't have access to it"
        )
    sly.logger.info(f"Exporting Dataset: id={dataset_infos[0].id}, name={dataset_infos[0].name}")
else:
    # If it does not, obtain all datasets infos from the current project.
    dataset_infos = api.dataset.get_list(PROJECT_ID)
    sly.logger.info(f"Exporting all datasets from project.")

# track progress datasets processing using Tqdm
with tqdm(total=len(dataset_infos)) as ds_pbar:
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
                image_path = os.path.join(PROJECT_DIR, dataset.name, image_info.name)
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

                # increment the current images progress counter by 1
                pbar.update(1)
        # increment the current dataset progress counter by 1
        ds_pbar.update(1)

        # create JSON annotation in new format
        filename = os.path.join(PROJECT_DIR, dataset.name, ANN_FILE_NAME)
        with open(filename, "w") as file:
            json.dump(result_anns, file, indent=2)

# prepare archive from result project dir
archive_path = f"{PROJECT_DIR}.tar"
sly.fs.archive_directory(PROJECT_DIR, archive_path)
sly.fs.remove_dir(PROJECT_DIR)
PROJECT_DIR = archive_path

# upload project to Supervsiely in production mode 
if IS_PRODUCTION:
    progress = tqdm(
        desc=f"Uploading '{os.path.basename(PROJECT_DIR)}'",
        total=sly.fs.get_directory_size(PROJECT_DIR),
        unit="B",
        unit_scale=True,
    )
        
    remote_path = os.path.join(
        sly.team_files.RECOMMENDED_EXPORT_PATH,
        "Supervisely App",
        str(TASK_ID),
        f"{sly.fs.get_file_name_with_ext(PROJECT_DIR)}",
    )

    file_info = api.file.upload(
        team_id=TEAM_ID,
        src=PROJECT_DIR,
        dst=remote_path,
        progress_cb=progress,
    )
    api.task.set_output_archive(
        task_id=TASK_ID, file_id=file_info.id, file_name=file_info.name
    )
    sly.logger.info(f"Remote file: id={file_info.id}, name={file_info.name}")
    sly.fs.silent_remove(PROJECT_DIR) # remove local directory

app.shutdown() # stop app
```

## Step 3. Advanced debug
 
In addition to the regular debug option, this template also includes setting for `Advanced debugging`.

![launch.json](https://user-images.githubusercontent.com/79905215/236436739-9bc2192d-e34f-4630-bf63-5ab184710526.png)

The advanced debugging option is somewhat identical, however it will upload result archive or folder with data to `Team Files` instead (Path to result archive - `/tmp/supervisely/export/Supervisely App/<SESSION ID>/<PROJECT_ID>_<PROJECT_NAME>.tar`).
This option is an example of how production apps work in Supervisely platform.

![Advanced debug](https://user-images.githubusercontent.com/79905215/236843765-f86a4c4d-c649-4cd5-b840-2ad266e381e3.gif)

Output of this python program:

```text
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 0, "total": 1, "timestamp": "2023-05-08T17:25:24.404Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 0, "total": 6, "timestamp": "2023-05-08T17:25:24.726Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 1, "total": 6, "timestamp": "2023-05-08T17:25:25.949Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Processing", "current": 1, "total": 1, "timestamp": "2023-05-08T17:25:33.269Z", "level": "info"}
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Uploading '20934.tar'", "current": 0, "total": 4567440, "current_label": "0.0 B", "total_label": "0.0 B", "timestamp": "2023-05-08T17:26:05.111Z", "level": "info"}
...
{"message": "progress", "event_type": "EventType.PROGRESS", "subtask": "Uploading '20934.tar'", "current": 4567440, "total": 4567440, "current_label": "4.4 MiB", "total_label": "4.4 MiB", "timestamp": "2023-05-08T17:13:19.529Z", "level": "info"}
{"message": "Remote file: id=1274760, name=20934.tar", "timestamp": "2023-05-08T17:13:20.646Z", "level": "info"}
```

## Step 4. How to run it in Supervisely

Submitting an app to the Supervisely Ecosystem isn’t as simple as pushing code to github repository, but it’s not as complicated as you may think of it either.

Please follow this [link](../basics/add-private-app.md) for instructions on adding your app. We have produced a step-by-step guide on how to add your application to the Supervisely Ecosystem.

![Release custom export app](https://user-images.githubusercontent.com/79905215/236866286-283f646d-73a3-4180-a14b-6990feeffa98.gif)
