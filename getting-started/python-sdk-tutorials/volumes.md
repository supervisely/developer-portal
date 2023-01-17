# Basic operations with volumes

## Introduction

In this tutorial we will focus on working with volumes using Supervisely SDK.

You will learn how to:

1. [upload volume from local directory to Supervisely dataset.](#upload-nrrd-format-volume)
2. [upload volume to Supervisely as NumPy matrix.](#upload-volume-as-numpy-array)
3. [upload list of volumes from local directory to Supervisely.](#upload-list-of-volumes-from-local-directory)
4. [upload DICOM format volumes from local directory.](#upload-dicom-volumes-from-local-directory)
5. [get single volume info by id.](#get-single-volume-info-by-id)
6. [get single volume info by name.](#get-single-volume-info-by-name)
7. [get list of informations about volumes.](#get-list-of-volumes-infos-from-current-dataset)
8. [download volume from Supervisely to local directory.](#download-volume-from-supervisely-to-local-directory)


📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-volumes): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-image) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-volumes.git

cd tutorial-volumes

./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Change workspace ID in `local.env` file by copying the ID from the context menu of the workspace.

```
context.workspaceId=654 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209327856-e47fb82b-c207-48fc-bb36-1fe795d45f6f.png" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `src/main.py`.

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
from supervisely.project.project_type import ProjectType
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance.

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get variables from environment

In this tutorial, you will need an workspace ID that you can get from environment variables. [Learn more here](../environment-variables.md#workspace_id)

```python
workspace_id = sly.env.workspace_id()
```

### Create new project and dataset

Create new project.

**Source code:**

```python
project = api.project.create(
    workspace_id,
    "Volume tutorial",
    ProjectType.VOLUMES,
    change_name_if_conflict=True,
)

print(f"Project ID: {project.id}")
```

**Output:**

```python
# Project ID: 16342
```

Create new dataset.

**Source code:**

```python
dataset = api.dataset.create(project.id, "dataset_1")

print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 54698
```

## Upload volumes from local directory to Supervisely

### Upload NRRD format volume

**Source code:**

```python
upload_path = "src/upload/MRHead.nrrd"

nrrd_info = api.volume.upload_nrrd_serie_path(
    dataset.id,
    "NRRD_1.nrrd",
    upload_path,
    log_progress=False,
)

print(f'"{nrrd_info.name}" volume uploaded to Supervisely with ID:{nrrd_info.id}')
```

**Output:**

```python
# "NRRD_1.nrrd" volume uploaded to Supervisely with ID:18562981
```

### Upload volume as NumPy array

**Source code:**

```python
np_volume, meta = sly.volume.read_nrrd_serie_volume_np(upload_path)

nrrd_info_np = api.volume.upload_np(dataset.id, "Np_volume.nrrd", np_volume, meta)

print(f"Volume uploaded as NumPy array to Supervisely with ID:{nrrd_info_np.id}")
```

**Output:**

```python
# Volume uploaded as NumPy array to Supervisely with ID:18562982
```

### Upload list of volumes from local directory

**Source code:**

```python
upload_dir_path = "src/upload/folder/"
paths = ["src/upload/folder/CTACardio.nrrd", "src/upload/folder/another_folder/MRHead-1.nrrd"]
names = ["CTACardio.nrrd", "MRHead-1.nrrd"]

volume_infos = api.volume.upload_nrrd_series_paths(dataset.id, names, paths)

print(f"All volumes has been uploaded with IDs: {[x.id for x in volume_infos]}")
```

**Output:**

```python
# All volumes has been uploaded with IDs: [18562983, 18562984]
```

### Upload DICOM volumes from local directory

**Source code:**

```python
upload_dir_path = "src/upload/Dicom_files/"
series_infos = sly.volume.inspect_dicom_series(root_dir=upload_dir_path)
for serie_id, files in series_infos.items():
    item_path = files[0]
    if sly.volume.get_extension(path=item_path) is None:
        sly.logger.warn(f"Can not recognize file extension {item_path}, serie will be skipped")
        continue
    name = f"{sly.fs.get_file_name(path=item_path)}.nrrd"

    volume_info = api.volume.upload_dicom_serie_paths(
        dataset_id=dataset.id,
        name=name,
        paths=files,
        log_progress=False,
        anonymize=True,  # hide patient's name and ID before uploading to Supervisely platform
    )
     print(f"DICOM volume has been uploaded to Supervisely with ID: {volume_info.id}")

```

**Output:**

```python
# DICOM volume has been uploaded to Supervisely with ID: 18562990
```

<figure><img src="https://user-images.githubusercontent.com/79905215/212952335-d5abd038-e0c9-4ad3-b716-c8658bbba5d5.png" alt=""><figcaption></figcaption></figure>

**Now you can explore and label it in [Supervisely labeling tool](https://dev.supervise.ly/ecosystem/annotation_tools/dicom-labeling-tool)**:

<figure><img src="https://user-images.githubusercontent.com/79905215/212951761-97facd6d-143e-4edc-8568-6c1c63471f99.png" alt=""><figcaption></figcaption></figure>

## Get volume info from Supervisely

### Get single volume info by id

**Source code:**

```python
info_by_id = api.volume.get_info_by_id(id=nrrd_info.id)

print(f"Volume name: ", info_by_id.name)
```

**Output:**

```python
# Volume name:  NRRD_1.nrrd
```

### Get single volume info by name

**Source code:**

```python
info_by_name = api.volume.get_info_by_name(dataset.id, name=nrrd_info.name)

print(f"Volume name: ", info_by_name.name)
```

**Output:**

```python
# Volume name:  NRRD_1.nrrd
```

### Get list of volumes infos from current dataset

**Source code:**

```python
volumes_list = api.volume.get_list(dataset.id)

volumes_ids = [x.id for x in volumes_list]

print(f"List of volumes`s IDs: {volumes_ids}")
```

**Output:**

```python
# List of volumes`s IDs: [18562986, 18562987, 18562988, 18562989, 18562990]
```


## Download volume from Supervisely to local directory

**Source code:**

```python
download_dir = "src/download/"
download_path = os.path.join(download_dir, "MRHead.nrrd")

if os.path.exists(download_path):
    os.remove(download_path)

api.volume.download_path(nrrd_info.id, download_path)

list_dir = os.listdir(download_dir)
if len(list_dir) > 0:
    print("Volume successfully downloaded.")
```

**Output:**

```python
# Volume successfully downloaded.
```
