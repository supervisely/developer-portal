# Basic operations with volumes

## Introduction

In this tutorial we will focus on working with volumes using Supervisely SDK.

You will learn how to:

1. [upload volume from local directory to Supervisely dataset](#upload-nrrd-format-volume)
2. [upload volume to Supervisely as NumPy matrix](#upload-volume-as-numpy-array)
3. [upload list of volumes from local directory to Supervisely](#upload-list-of-volumes-from-local-directory)
4. [get list of volume infos](#get-list-of-volumes-infos-from-current-dataset)
5. [get single volume info by id](#get-single-volume-info-by-id)
6. [get single volume info by name](#get-single-volume-info-by-name)
7. [upload DICOM series from local directory](#inspect-and-upload-dicom-series-from-local-directory)
8. [download volume from Supervisely to local directory](#download-volume-from-supervisely-to-local-directory)
9. [download slice as NumPy from Supervisely by ID](#download-slice-as-numpy-from-supervisely-by-id)
10. [save slice as NRRD or JPG file](#save-slice-to-local-directory-as-nrrd)
11. [read NRRD files from local directory](#read-nrrd-file-from-local-directory)
12. [get volume slices from local directory](#get-slices-from-volume)


📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-volume): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-volume) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-volume.git

cd tutorial-volume

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

**Step 5.** Download sample [volumes](https://github.com/supervisely-ecosystem/tutorial-volume/releases/download/v0.0.1/upload.tar.gz)


**Step 6.** Place downloaded files in the project structure as shown below:
```
tutorial-volume
├── .vscode
├── src
│     ├── upload
│     │     ├── MRHead_dicom        <-- sample dicom files
│     │     │   ├── 000000.dcm
│     │     │   ├── 000001.dcm
│     │     │   └── ...
│     │     └── nrrd
│     │         ├── CTACardio.nrrd  <-- sample NRRD
│     │         ├── CTChest.nrrd    <-- sample NRRD
│     │         └── MRHead.nrrd     <-- sample NRRD
│     └── main.py
├── .gitignore
├── create_venv.sh
├── local.env
├── README.md
└── requirements.txt
```

**Step 7.** Start debugging `src/main.py`.

### Import libraries

```python
import os

import cv2
from dotenv import load_dotenv
import nrrd
import numpy as np
from pprint import pprint
import supervisely as sly
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

Create new project with **`ProjectType.VOLUMES`** type.

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
local_path = "src/upload/nrrd/MRHead.nrrd"

nrrd_info = api.volume.upload_nrrd_serie_path(
    dataset.id,
    "MRHead.nrrd",
    local_path,
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
np_volume, meta = sly.volume.read_nrrd_serie_volume_np(local_path)

nrrd_info_np = api.volume.upload_np(
    dataset.id,
    "MRHead_np.nrrd",
    np_volume,
    meta,
)

print(f"Volume uploaded as NumPy array to Supervisely with ID:{nrrd_info_np.id}")
```

**Output:**

```python
# Volume uploaded as NumPy array to Supervisely with ID:18562982
```

### Upload list of volumes from local directory

**Source code:**

```python
local_dir_name = "src/upload/nrrd/"
all_nrrd_names = os.listdir(local_dir_name)
names = [f"1_{name}" for name in all_nrrd_names]
paths = [os.path.join(local_dir_name, name) for name in all_nrrd_names]

volume_infos = api.volume.upload_nrrd_series_paths(dataset.id, names, paths)
print(f"All volumes has been uploaded with IDs: {[x.id for x in volume_infos]}")
```

**Output:**

```python
# All volumes has been uploaded with IDs: [18630605, 18630606, 18630607]
```




## Get volume info from Supervisely


### Get list of volumes infos from current dataset

**Source code:**

```python
volume_infos = api.volume.get_list(dataset.id)

volumes_ids = [x.id for x in volume_infos]

print(f"List of volumes`s IDs: {volumes_ids}")
```

**Output:**

```python
# List of volumes`s IDs: [18562986, 18562987, 18562988, 18562989, 18562990]
```

### Get single volume info by id

**Source code:**

```python
volume_id = volume_infos[0].id

volume_info_by_id = api.volume.get_info_by_id(id=volume_id)

print(f"Volume name: ", volume_info_by_id.name)
```

**Output:**

```python
# Volume name:  NRRD_1.nrrd
```

### Get single volume info by name

**Source code:**

```python
volume_info_by_name = api.volume.get_info_by_name(dataset.id, name="MRHead.nrrd")

print(f"Volume name: ", volume_info_by_name.name)
```

**Output:**

```python
# Volume name:  NRRD_1.nrrd
```





### Inspect and upload DICOM series from local directory

**Source code:**

```python
dicom_dir_name = "src/upload/MRHead_dicom/"

# inspect you local directory and collect all dicom series.
series_infos = sly.volume.inspect_dicom_series(root_dir=dicom_dir_name)

# upload DICOM series from local directory to Supervisely platform
for serie_id, files in series_infos.items():
    item_path = files[0]
    name = f"{sly.fs.get_file_name(path=item_path)}.nrrd"
    dicom_info = api.volume.upload_dicom_serie_paths(
        dataset_id=dataset.id,
        name=name,
        paths=files,
        anonymize=True,
    )
    print(f"DICOM volume has been uploaded to Supervisely with ID: {dicom_info.id}")

```

> Set **`anonymize=True`** if you want to anonymize DICOM series and hide **`PatientID`** and **`PatientName`** fields.

**Output:**

```python
# DICOM volume has been uploaded to Supervisely with ID: 18630608
```

<figure><img src="https://user-images.githubusercontent.com/79905215/212952335-d5abd038-e0c9-4ad3-b716-c8658bbba5d5.png" alt=""><figcaption></figcaption></figure>

**Now you can explore and label it in [Supervisely labeling tool](https://dev.supervise.ly/ecosystem/annotation_tools/dicom-labeling-tool)**:

<figure><img src="https://user-images.githubusercontent.com/79905215/212951761-97facd6d-143e-4edc-8568-6c1c63471f99.png" alt=""><figcaption></figcaption></figure>

## Download volume from Supervisely to local directory

**Source code:**

```python
volume_id = volume_infos[0].id
volume_info = api.volume.get_info_by_id(id=volume_id)

download_dir_name = "src/download/"
path = os.path.join(download_dir_name, volume_info.name)
if os.path.exists(path):
    os.remove(path)

api.volume.download_path(volume_info.id, path)

if os.path.exists(path):
    print(f"Volume (ID {volume_info.id}) successfully downloaded.")
```

**Output:**

```python
# Volume (ID 18630603) successfully downloaded.
```


## Download slice from Supervisely

### Download slice as NumPy from Supervisely by ID

**Source code:**

```python
slice_index = 60

image_np = api.volume.download_slice_np(
    volume_id=volume_id,
    slice_index=slice_index,
    plane=sly.Plane.SAGITTAL,
)

print(f"Image downloaded as NumPy array. Image shape: {image_np.shape}")
```

**Output:**

```python
# Image downloaded as NumPy array. Image shape: (256, 256, 3)
```

### Save slice to local directory as NRRD

Recommended way to save slice to preserve image quality (pixel depth)

```python
# save slice as NRRD file
nrrd_slice_path = os.path.join(download_dir_name, 'slice.nrrd')

nrrd.write(nrrd_slice_path, image_np)
```
### Save slice to local directory as JPG

```python
# save slice as jpg
image_slice_path = os.path.join(download_dir_name, 'slice.jpg')

cv2.imwrite(image_slice_path, image_np)
```





## Get volume slices from local directory

### Read NRRD file from local directory

Read NRRD file from local directory and get meta and volume (as NumPy array).
**Source code:**

```python
# read NRRD file from local directory
nrrd_path = os.path.join(download_dir_name, "MRHead.nrrd")
volume_np, meta = sly.volume.read_nrrd_serie_volume_np(nrrd_path)

pprint(meta)
```


**Output:**

```python
# {
#     'ACS': 'RAS',
#     'channelsCount': 1,
#     'dimensionsIJK': {'x': 130, 'y': 256, 'z': 256},
#     'directions': (1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0),
#     'intensity': {'max': 279.0, 'min': 0.0},
#     'origin': (-86.64489746093749, -121.07139587402344, -138.21430206298828),
#     'rescaleIntercept': 0,
#     'rescaleSlope': 1,
#     'spacing': (1.2999954223632812, 1.0, 1.0),
#     'windowCenter': 139.5,
#     'windowWidth': 279.0
# }
```


### Get slices from volume

Get slices from current volume.
In this example we will get sagittal slices.

**Source code:**

```python
slices = {}

dimension = volume_np.shape[0]  # change index: 0 - sagittal, 1 - coronal, 2 - axial
for batch in sly.batched(list(range(dimension))):
    for i in batch:
        if i >= dimension:
            continue
        pixel_data = volume_np[i, :, :]  # sagittal
        # pixel_data = volume_np[:, i, :]  # coronal
        # pixel_data = volume_np[:, :, i]  # axial
        slices[i] = pixel_data

print(f"{len(slices.keys())} slices has been received from current volume.")
```

**Output:**

```python
# 130 slices has been received from current volume.
```

### Display slices using OpenCV library

**Source code:**

```python
for i, s in slices.items():
    frame = np.array(s, dtype=np.uint8)
    cv2.imshow(f"frame #{i}", frame)
    cv2.waitKey(10)
    cv2.destroyAllWindows()
```