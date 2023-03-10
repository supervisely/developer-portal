# Basic operations with videos

## Introduction

In this tutorial we will focus on working with videos using Supervisely SDK.

You will learn how to:

1. [upload one or more videos to Supervisely dataset.](#upload-videos-from-local-directory-to-supervisely)
2. [get information about videos by id or name.](#get-information-about-videos)
3. [download video from Supervisely.](#download-video)
4. [get video metadata](#get-video-metadata)
5. [download one or more frames of video and save to local directory as images.](#download-video-frames-as-images)
6. [download one or more frames of video as RGB NumPy matrix.](#download-video-frames-as-rgb-numpy-matrix)
7. [remove videos from Supervisely.](#remove-videos-from-supervisely)
8. [choose from the available codecs, extensions and containers.](#information-about-available-codecs-extensions-and-containers)
9. [exract frames from videos correctly using the OpenCV library.](#how-to-exract-frames-from-videos-correctly-using-the-opencv-library)

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
WORKSPACE_ID=654 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209327856-e47fb82b-c207-48fc-bb36-1fe795d45f6f.png" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `src/main.py`.

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

In this tutorial, you will need an workspace ID that you can get from environment variables. [Learn more here](../environment-variables.md#workspace_id)

```python
workspace_id = sly.env.workspace_id()
```

### Create new project and dataset

Create new project.

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

Create new dataset.

**Source code:**

```python
dataset = api.dataset.create(project.id, "Birds")

print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 53465
```

## Upload videos from local directory to Supervisely

### Upload single video.

**Source code:**

```python
original_dir = "src/videos/original"
path = os.path.join(original_dir, "Penguins.mp4")

video = api.video.upload_path(
    dataset.id,
    name="Penguins",
    path=path
)

print(f'Video "{video.name}" uploaded to Supervisely with ID:{video.id}')
```

**Output:**

```python
# Video "Penguins" uploaded to Supervisely platform with ID:17539140
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209328524-3887105e-e694-493c-9a51-91d44d6ce636.png" alt=""><figcaption></figcaption></figure>

### Upload list of videos.

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

<figure><img src="https://user-images.githubusercontent.com/79905215/209328566-3e8c95c8-9c0d-4b4c-8cdd-fd11d0cee9bd.png" alt=""><figcaption></figcaption></figure>

## Get information about videos

### Single video.

Get information about video from Supervisely by id.

**Source code:**

```python
video_info = api.video.get_info_by_id(video.id)

print(video_info)
```

**Output:**

```python
# VideoInfo(
#     id=17539140,
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

### Get all videos from dataset.

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

## Download video

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

<figure><img src="https://user-images.githubusercontent.com/79905215/209328639-f2456969-c171-49ec-b880-590a6fc9de81.png" alt=""><figcaption></figcaption></figure>

## Get video metadata

### Get video metadata from file

**Source code:**

```python
video_path = "src/videos/result/Penguins.mp4"
file_info = sly.video.get_info(video_path)
pprint(file_info)
```

**Output:**

```python
{
  "duration": 16.52,
  "formatName": "mov,mp4,m4a,3gp,3g2,mj2",
  "size": "1599101",
  "streams": [
    {
      "codecName": "h264",
      "codecType": "video",
      "duration": 16.52,
      "framesCount": 413,
      "framesToTimecodes": [],
      "height": 360,
      "index": 0,
      "rotation": 0,
      "startTime": 0,
      "width": 640
    }
  ]
}
```

### Get video metadata from server

**Source code:**

```python
video_info = api.video.get_info_by_id(video.id)
pprint(video_info.file_meta)
```

**Output:**

```python
{
  "codecName": "h264",
  "codecType": "video",
  "duration": 16.52,
  "formatName": "mov,mp4,m4a,3gp,3g2,mj2",
  "framesCount": 413,
  "framesToTimecodes": [],
  "height": 360,
  "index": 0,
  "mime": "video/mp4",
  "rotation": 0,
  "size": "1599101",
  "startTime": 0,
  "streams": [],
  "width": 640
}
```

## Download video frames as images

Download single frame of video as image from Supervisely to local directory.

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

## Download video frames as RGB NumPy matrix

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

## Remove videos from Supervisely

### Remove one video.

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

### Remove list of videos.

Remove list of videos from Supervisely by ids.

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

## Information about available codecs, extensions, and containers.

> **Note:**
> Only basic video codecs are available in the Community Edition, for additional video codecs you can try the Enterprise Edition.

**The Enterprise Edition** allows you to use the full range of extensions, containers and codecs listed below without any limits.

- extensions: _.avi, .mp4, .3gp, .flv, .webm, .wmv, .mov, .mkv_
- containers: _mp4, webm, ogg, ogv_
- codecs: _h264, vp8, vp9_

In the Community Edition, it is recommended to use _vp9, h264_ codecs with _mp4_ container.


## How to exract frames from videos correctly using the OpenCV library.

In case you need to exract frames from videos, you should be aware of one important detail of the OpenCV library.
According to [issue #15499](https://github.com/opencv/opencv/issues/15499), different versions of the OpenCV library have different values of the `CAP_PROP_ORIENTATION_AUTO` flag. This may cause that VideoCapture to ignore video orientation metadata.
To avoid incorrect extraction of frames from videos, it is recommended to directly define flag `CAP_PROP_ORIENTATION_AUTO`:

```python
cap = cv2.VideoCapture(filepath)

# set CAP_PROP_ORIENTATION_AUTO flag for VideoCapture
cap.set(cv2.CAP_PROP_ORIENTATION_AUTO, 1)

while cap.isOpened():
    success, frame = cap.read()
    if not success:
        print("Can't receive frame (stream end?). Exiting ...")
        break
    # cv2.imshow(f"{frame.shape[:2]}", frame)
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

# Release everything if job is finished
cap.release()
# cv2.destroyAllWindows()
```
