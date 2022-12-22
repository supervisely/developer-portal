# Basic operations with videos

## Introduction

In this tutorial we will focus on working with videos using Supervisely SDK.

You will learn how to:

1. upload one or more videos to Supervisely dataset.
2. get information about video by id or name.
3. get all videos from Supervisely dataset.
4. download video from Supervisely.
5. download one or more frames of video and save to local directory as images.
6. download one or more frames of video as RGB NumPy matrix.

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-video): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-video) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-video.git

cd tutorial-video

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

<figure><img src="https://user-images.githubusercontent.com/93247833/209009648-0bfc39c5-586a-495c-b5ab-bf6d8dc99f07.png"" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `src/main.py`.

### Import libraries

```python
import os
from dotenv import load_dotenv
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

## **Part 1.** Create new project and dataset

Create new project

**Source code:**

```python
project = api.project.create(
    workspace_id, "Animals", type=ProjectType.VIDEOS, change_name_if_conflict=True
)

print(f"Project ID: {project.id}")
```

**Output:**

```python
# Project ID: 15599
```

Create new dataset

**Source code:**

```python
dataset = api.dataset.create(project.id, "Birds")

print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 53465
```

## **Part 2.** Upload videos from local directory to Supervisely.

### **Part 2.1.** Upload single video.

**Source code:**

```python
original_dir = "src/videos/original"
path = os.path.join(original_dir, "Penguins.mp4")
meta = {"my-field-1": "my-value-1", "my-field-2": "my-value-2"}

video = api.video.upload_path(
    dataset.id,
    name="Penguins",
    path=path,
    meta=meta,  # optional: you can add metadata to video.
)

print(f'Video "{video.name}" uploaded to Supervisely with ID:{video.id}')
```

**Output:**

```python
# Video "Penguins" uploaded to Supervisely platform with ID:17536611
```

<figure><img src="https://user-images.githubusercontent.com/93247833/209021729-7514437a-1fca-4259-a57c-afcf1c3ce935.png" alt=""><figcaption></figcaption></figure>

### **Part 2.2.** Upload list of videos.

✅ Supervisely API allows uploading multiple videos in a single request. The code sample below sends fewer requests and it leads to a significant speed-up of our original code.

**Source code:**

```python
names = ["Flamingo.mp4", "Swans.mp4", "Toucan.mp4"]
paths = [os.path.join(original_dir, name) for name in names]

upload_info = api.video.upload_paths(dataset.id, names, paths)

print(f"{len(upload_info)} videos successfully uploaded to  Supervisely platform")
```

**Output:**

```python
# 3 videos successfully uploaded to Supervisely platform
```

<figure><img src="https://user-images.githubusercontent.com/93247833/209021518-eb37a2a6-0dd6-4d1e-a6df-fc9848b50389.png" alt=""><figcaption></figcaption></figure>

## **Part 3.** Getting information about videos

### **Part 3.1.** Single video

Get information about video from Supervisely by id.

**Source code:**

```python
video_info = api.video.get_info_by_id(video.id)

print(video_info)
```

**Output:**

```python
# VideoInfo(
#     id=17536607,
#     name="Penguins",
#     hash="6aTUVnuyfGqIuMxH8l1t1yvkmn/9iuWbHUOE7iebhYk=",
#     link=None,
#     team_id=435,
#     workspace_id=654,
#     project_id=15748,
#     dataset_id=53751,
#     path_original="/h5un6l2bnaz1vj8a9qgms4-public/videos/U/O/LF/T7tiPVoBUyFaM.mp4",
#     frames_to_timecodes=[],
#     frames_count=413,
#     frame_width=640,
#     frame_height=360,
#     created_at="2022-12-21T23:09:54.191Z",
#     updated_at="2022-12-21T23:09:54.191Z",
#     tags=[],
#     file_meta={},
#     custom_data={},
#     processing_path="1/1703793",
# )
```

You can also get information about video from Supervisely by name.

**Source code:**

```python
video_info_by_name = api.video.get_info_by_name(dataset.id, video.name)

print(f"Video name - '{video_info_by_name.name}'")
```

**Output:**

```python
# Video name - 'Penguins'
```

### **Part 3.2.** Multiple videos

Get information about video from Supervisely by id.

**Source code:**

```python
video_info_list = api.video.get_list(dataset.id)

print(f"{len(video_info_list)} videos information received.")
```

**Output:**

```python
# 4 videos information received.
```

## **Part 4.** Download video.

Download video from Supervisely to local directory by id.
**Source code:**

```python
save_path = os.path.join(result_dir, f"{video_info.name}.mp4")

api.video.download_path(video_info.id, save_path)

print(f"Video has been successfully downloaded to '{save_path}'")
```

**Output:**

```python
# Video has been successfully downloaded to 'src/videos/result/Penguins.mp4'
```

<figure><img src="https://user-images.githubusercontent.com/93247833/209016452-051d824f-7bdd-4c74-a52a-5fb838aca676.png" alt=""><figcaption></figcaption></figure>

## **Part 5.** Download frames

### **Part 5.1.** Download single frame

Download frame of video as image from Supervisely to local directory.

**Source code:**

```python
frame_idx = 15
file_name = "frame.png"
save_path = os.path.join(result_dir, file_name)

api.video.frame.download_path(video_info.id, frame_idx, save_path)

print(f"Video frame has been successfully downloaded as image to '{save_path}'")
```

**Output:**

```python
# Video frame has been successfully downloaded as image to 'src/videos/result/frame.png'
```

You can also download video frame as RGB NumPy matrix.

**Source code:**

```python
video_frame_np = api.video.frame.download_np(video_info.id, frame_idx)
print(f"Video frame downloaded as RGB NumPy matrix. Frame shape: {video_frame_np.shape}")
```

**Output:**

```python
# Video frame downloaded as RGB NumPy matrix. Frame shape: (360, 640, 3)
```

### **Part 5.2.** Download multiple frames in a single request

Download multiple frames in a single requests from Supervisely and save as images to local directory.

**Source code:**

```python
frame_indexes = [5, 10, 20, 30, 45]
save_paths = [os.path.join(result_dir, f"frame_{idx}.png") for idx in frame_indexes]

api.video.frame.download_paths(video_info.id, frame_indexes, save_paths)

print(f"{len(frame_indexes)} images has been successfully downloaded to '{save_path}'")
```

**Output:**

```python
# 5 images has been successfully downloaded to 'src/videos/result/frame.png'
```

Download multiple frames in a single requests from Supervisely as RGB NumPy matrix.

**Source code:**

```python
video_frames_np = api.video.frame.download_nps(video_info.id, frame_indexes)

print(f"{len(video_frames_np)} video frames downloaded in RGB NumPy matrix.")
```

**Output:**

```python
# 5 video frames downloaded as RGB NumPy matrix.
```

## **Part 6.** Remove videos from Supervisely

### **Part 6.1.** Remove one video

Remove video from Supervisely by id

**Source code:**

```python
api.video.remove(video_info.id)
print(f"Video (ID: {video_info.id}) successfully removed.")
```

**Output:**

```python
# Video (ID: 17536607) has been successfully removed.
```

### **Part 6.1.** Remove list of videos

Remove list of videos from Supervisely by ids

**Source code:**

```python
videos_to_remove = api.video.get_list(dataset.id)
remove_ids = [video.id for video in videos_to_remove]
api.video.remove_batch(remove_ids)
print(f"{len(videos_to_remove)} videos successfully removed.")
```

**Output:**

```python
# 3 videos have been successfully removed.
```

### Information about available codecs, extensions, and containers

> **Note:**
> Only basic video codecs are available in the Community Edition, for additional video codecs you can try the Enterprise Edition.

**The Enterprise Edition** allows you to use the full range of extensions, containers and codecs listed below without any limits.

- extensions: _.avi, .mp4, .3gp, .flv, .webm, .wmv, .mov, .mkv_
- containers: _mp4, webm, ogg, ogv_
- codecs: _h264, vp8, vp9_

In the Community Edition, it is recommended to use _vp9, h264_ codecs with _mp4_ container.
