# Volumes (DICOM)

## Introduction

In this tutorial we will focus on working with volumes using Supervisely SDK.

You will learn how to:

1. [upload volume (NRRD) from local directory to Supervisely dataset](volumes.md#upload-nrrd-format-volume)
2. [upload volume to Supervisely as NumPy matrix](volumes.md#upload-volume-as-numpy-array)
3. [upload DICOM series from local directory](volumes.md#upload-dicom-series-from-local-directory)
4. [upload list of volumes from local directory to Supervisely](volumes.md#upload-list-of-volumes-from-local-directory)
5. [get list of volume infos](volumes.md#get-list-of-volumes-infos-from-current-dataset)
6. [get single volume info by id](volumes.md#get-single-volume-info-by-id)
7. [get single volume info by name](volumes.md#get-single-volume-info-by-name)
8. [download volume from Supervisely to local directory](volumes.md#download-volume-from-supervisely-to-local-directory)
9. [read NRRD files from local directory](volumes.md#read-nrrd-file-from-local-directory)
10. [get volume slices from local directory](volumes.md#get-slices-from-volume)
11. [download slice as NumPy from Supervisely by ID](volumes.md#download-slice-from-supervisely)
12. [save slice as NRRD or JPG file](volumes.md#save-slice-to-local-directory)

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-volume): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

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
WORKSPACE_ID=654 # ⬅️ change value
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

from dotenv import load_dotenv
from pprint import pprint
import supervisely as sly
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance.

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get variables from environment

In this tutorial, you will need an workspace ID that you can get from environment variables. [Learn more here](../../environment-variables.md#workspace\_id)

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

### Upload DICOM series from local directory

Inspect you local directory and collect all dicom series.

**Source code:**

```python
dicom_dir_name = "src/upload/MRHead_dicom/"

series_infos = sly.volume.inspect_dicom_series(root_dir=dicom_dir_name)
```

Upload DICOM series from local directory to Supervisely platform.

**Source code:**

```python
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

<figure><img src="https://user-images.githubusercontent.com/79905215/212952335-d5abd038-e0c9-4ad3-b716-c8658bbba5d5.png" alt=""><figcaption></figcaption></figure>

**Now you can explore and label it in** [**Supervisely labeling tool**](https://dev.supervisely.com/ecosystem/annotation\_tools/dicom-labeling-tool):

<figure><img src="https://user-images.githubusercontent.com/79905215/212951761-97facd6d-143e-4edc-8568-6c1c63471f99.png" alt=""><figcaption></figcaption></figure>

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

print(f"Volume name:", volume_info_by_id.name)
```

**Output:**

```python
# Volume name: NRRD_1.nrrd
```

### Get single volume info by name

**Source code:**

```python
volume_info_by_name = api.volume.get_info_by_name(dataset.id, name="MRHead.nrrd")

print(f"Volume name:", volume_info_by_name.name)
```

**Output:**

```python
# Volume name: NRRD_1.nrrd
```

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

Get slices from current volume. In this example we will get sagittal slices.

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

## Download slice from Supervisely

Download slice as NumPy from Supervisely by ID

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

## Save slice to local directory

✅ There is a built-in function **`supervisely.image.write`** which reads file extension from path and saves image (slice) with the desired format in local directory.

Example:

```python
import supervisely as sly

sly.image.write("folder/slice.nrrd", image_np) # save as NRRD
sly.image.write("folder/slice.jpg", image_np) # save as JPG
```

#### Save slice as NRRD

Recommended way to save slice as NRRD file to preserve image quality (pixel depth)

**Source code:**

```python
# save slice as NRRD file
save_dir = "src/download/"
nrrd_slice_path = os.path.join(save_dir, 'slice.nrrd')

sly.image.write(nrrd_slice_path, image_np)
```

### Save slice as JPG

**Source code:**

```python
# save slice as jpg
save_dir = "src/download/"
image_slice_path = os.path.join(save_dir, 'slice.jpg')

sly.image.write(jpg_slice_path, image_np)
```

#### Note:

In case you save slice using `nrrd` library, it is [recommended](https://pynrrd.readthedocs.io/en/stable/background/index-ordering.html) to use `C-order` indexing.

```python
save_dir = "src/download/"
slice_path = os.path.join(save_dir, 'slice.nrrd')

nrrd.write(slice_path, image_np, index_order='C')
```
