# Video SDK tutorial

## Introduction

In this tutorial we will be focusing on working with videos using Supervisely SDK. We'll go through complete cycle from getting existing videos to working with annotations and metadata.

You will learn how to:

1. get information about a video by id
2. get all the videos in a dataset
3. download/upload video
4. download/upload video batches
5. download a separate frame or frame range
6. download an annotation
7. work with annotations (adding tags/labels)
8. add metadata to a video

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Create [**Virtual Environment**](https://docs.python.org/3/library/venv.html) and prepare template for the project.

> You can try this example for yourself: VSCode project config, original image, and python script for this tutorial are ready on [GitHub](https://github.com/supervisely-ecosystem/supervisely-python-sdk-example).

**Step 3.** Get [**Videos example**](https://dev.supervise.ly/ecosystem/projects/videos-example) project from ecosystem. Videos example is an mini-dataset with 2 labeled videos what you can use in different tasks from labeling to train Neural Networks.

<figure><img src="https://user-images.githubusercontent.com/79905215/208432417-908f4333-9ef6-4f7b-9abd-9f0f9ed22995.png" alt=""><figcaption></figcaption></figure>

**Step 4.** change _project ID_, _dataset ID_ and _video ID_ in `local.env` file by copying the ID from the context menu of the project, dataset and video.

```
modal.state.slyProjectId=15599 # ⬅️ change value
modal.state.slyDatasetId=53465 # ⬅️ change value
modal.state.slyVideoId=17523674 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/93247833/208496631-02b3b9f3-2173-4692-95b2-2e2cd9dda777.png"" alt=""><figcaption></figcaption></figure>
<figure><img src="https://user-images.githubusercontent.com/93247833/208448804-7ff96efe-eb57-422d-ae42-1c3b065553c7.png" alt=""><figcaption></figcaption></figure>
<figure><img src="https://user-images.githubusercontent.com/93247833/208449096-a2cd4c10-f37d-4954-86c6-e39cba9873fa.png" alt=""><figcaption></figcaption></figure>

**Step5.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step6.** Start debugging `src/main.py`.

### Import libraries and Init API client

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get variables from environment

```python
project_id = int(os.environ["modal.state.slyProjectId"])
dataset_id = int(os.environ["modal.state.slyDatasetId"])
video_id = int(os.environ["modal.state.slyVideoId"])
```

## **Part 1.** Get information about the video by id

```python
video_info = api.video.get_info_by_id(video_id)

print(video_info)
# VideoInfo(id=17523674, name='Test Videos_dataset_animals_sea_lion.mp4', ...)
```

## **Part 2.** Get all the videos in the dataset

```python
video_info_list = api.video.get_list(dataset_id)

print(video_info_list)
# [VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...),
#  VideoInfo(id=17523674, name='Test Videos_dataset_animals_sea_lion.mp4', ...)]
```

## **Part 3.** Download the video

- Download the video from Dataset to the local path by ID.

```python
video_info = api.video.get_info_by_id(video_id)
save_path = os.path.join("/home/admin/work/projects/videos/", video_info.name)

api.video.download_path(video_id, save_path)
```

- Download the video with the given ID between the given start and end frames.

```python
start_fr = 5
end_fr= 35

response = api.video.download_range_by_id(video_id, start_fr, end_fr)
```

- Download the video with the given path in Supervisely between the given start and end frames.

```python
start_fr = 5
end_fr= 35
video_info = api.video.get_info_by_id(video_id)
path_sl = video_info.path_original

response = api.video.downalod_range_by_path(path_sl, start_fr, end_fr)
```

- Download the video with the given ID in Supervisely between the given start and end frames on the given path.

```python
start_fr = 5
end_fr= 35
video_info = api.video.get_info_by_id(video_id)
name = video_info.name
save_path = os.path.join('/home/admin/work/projects/videos', name)

result = api.video.download_save_range(video_id, start_fr, end_fr, save_path)
print(result)
# Output: /home/admin/work/projects/videos/MOT16-03.mp4
```

## **Part 4.** Upload the video

### **Part 4.1.** Upload the single video

- Upload the single video from the local directory

```python
local_path = '/home/admin/work/projects/videos/myvideo.mp4'

upload_info = api.video.upload_path(dataset_id=dataset_id, name='My_video', path=local_path)
print(upload_info)
# Output: VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...)
```

- Upload the single video from the link

```python
link = 'https://youtu.be/Mp0BnWEujhA'

upload_info = api.video.upload_link(dataset_id=dataset_id, link=link)
print(upload_info)
# Output: VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...)
```

- Upload the video from given hash to Dataset.

```python
video_info = api.video.get_info_by_id(src_video_id)
hash = video_info.hash
name = video_info.name
new_video_info = api.video.upload_hash(dst_dataset_id, name, hash)
print(new_video_info)
# Output: [
#     198723201,
#     "MOT16-03.mp4",
#     "ob69way77JS/qKdKHfOI/d0oZNdabAlG4m+c7YHtGnM=",
#     null,
#     null,
#     null,
#     466654,
#     "videos/P/H/Pd/rOz7a5usR...9Z91YT.mp4",
#     [
#         0,
#         0.066667,
#         0.133333,
#           ...
#         99.866667,
#         99.933333
#     ],
#     1500,
#     960,
#     540,
#     "2021-03-23T15:51:40.595Z",
#     "2021-03-23T15:51:40.595Z"
# ]
```

### **Part 4.2.** Upload video batches

- Upload a list of videos from the directory

```python
local_path1 = "/home/admin/work/projects/videos/myvideo.mp4"
local_path2 = "/home/admin/work/projects/videos/myvideo2.mp4"

upload_info = api.video.upload_paths(
    dataset_id=dataset_id,
    names=["video1.mp4", "video2.mp4"],
    paths=[local_path1, local_path2],
)
print(upload_info)
# Output: [VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...), VideoInfo(...)]
```

- Upload a list of videos from the links

```python
link1 = "https://youtu.be/SsupAPrvz8U"
link2 = "https://youtu.be/htUaZ8su_M0"

upload_info = api.video.upload_links(
    dataset_id=dataset_id,
    links=[link1, link2],
    names=["video1.mp4", "video2.mp4"],
)
print(upload_info)
# Output: [VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...), VideoInfo(...)]
```

- Upload videos from given hashes to Dataset.

```python
src_dataset_id = 466639
dst_dataset_id = 468620
hashes = []
names = []
metas = []
video_infos = api.video.get_list(src_dataset_id)
# Create lists of hashes, videos names and meta information for each video
for video_info in video_infos:
    hashes.append(video_info.hash)
    # It is necessary to upload videos with the same names(extentions) as in src dataset
    names.append(video_info.name)
    metas.append({video_info.name: video_info.frame_height})

new_videos_info = api.video.upload_hashes(dst_dataset_id, names, hashes, metas)
print(new_videos_info)
# [VideoInfo(id=17523673, name='Test Videos_dataset_cars_cars.mp4', ...),
#  VideoInfo(id=17523674, name='Test Videos_dataset_animals_sea_lion.mp4', ...)]
```

## **Part 5.** Download the annotation

```python
ann = api.video.annotation.download(video_id)

print(ann)
# Output: {
#     "videoId": 17523674,
#     "videoName": "Videos_dataset_cars_cars.mp4",
#     "createdAt": "2021-03-23T13:14:25.536Z",
#     "updatedAt": "2021-03-23T13:16:43.300Z",
#     "description": "",
#     "tags": [],
#     "objects": [],
#     "size": {
#         "height": 2160,
#         "width": 3840
#     },
#     "framesCount": 326,
#     "frames": []
# }
```

```python
api.video.annotation.download_bulk(dataset_id, video_ids)
```

## **Part 6.** Work with annotations and tags

```python
height, width = 500, 700
frames_count = 1

# VideoObjectCollection
obj_class_car = sly.ObjClass('car', sly.Rectangle)
video_obj_car = sly.VideoObject(obj_class_car)
objects = sly.VideoObjectCollection([video_obj_car])

# FrameCollection
fr_index = 7
geometry = sly.Rectangle(0, 0, 100, 100)
video_figure_car = sly.VideoFigure(video_obj_car, geometry, fr_index)
frame = sly.Frame(fr_index, figures=[video_figure_car])
frames = sly.FrameCollection([frame])

# VideoTagCollection
from supervisely.video_annotation.video_tag import VideoTag
from supervisely.video_annotation.video_tag_collection import VideoTagCollection

meta_car = sly.TagMeta('car_tag', sly.TagValueType.ANY_STRING)
vid_tag = VideoTag(meta_car, value='acura')
video_tags = VideoTagCollection([vid_tag])

# Description
descr = 'car example'
video_ann = sly.VideoAnnotation((height, width), frames_count, objects, frames, video_tags, descr)

print(video_ann.to_json())

# Output: {
#     "size": {
#         "height": 500,
#         "width": 700
#     },
#     "description": "car example",
#     "key": "a85b282e5e174e7ebad6f878b6919244",
#     "tags": [
#         {
#             "name": "car_tag",
#             "value": "acura",
#             "key": "540a8212b0344788953996cea220ea8b"
#         }
#     ],
#     "objects": [
#         {
#             "key": "7c74b8a495044ea0ac127f32751c8f5c",
#             "classTitle": "car",
#             "tags": []
#         }
#     ],
#     "frames": [
#         {
#             "index": 7,
#             "figures": [
#                 {
#                     "key": "82dcbf2e3c5f42a99eeea2ad34173793",
#                     "objectKey": "7c74b8a495044ea0ac127f32751c8f5c",
#                     "geometryType": "rectangle",
#                     "geometry": {
#                         "points": {
#                             "exterior": [
#                                 [
#                                     0,
#                                     0
#                                 ],
#                                 [
#                                     100,
#                                     100
#                                 ]
#                             ],
#                             "interior": []
#                         }
#                     }
#                 }
#             ]
#         }
#     ],
#     "framesCount": 1
# }
```

## **Part 7.** Add metadata to a video

- Upload meta to the single video

```python
local_path = '/home/admin/work/projects/videos/myvideo.mp4'

upload_info = api.video.upload_path(
    dataset_id=dataset_id, name="My_video", path=local_path, meta={1: "meta_example"}
)
```


```python
link = 'https://youtu.be/Mp0BnWEujhA'

upload_info = api.video.upload_link(dataset_id=dataset_id, link=link, meta={1: "meta_example"})
```

- Upload metas to the videos.

```python
video_info = api.video.get_info_by_id(video_id)
hashes = [video_info].hash
names = [video_info.name]
metas = [{1: "meta_example"}]
new_video_info = api.video.upload_hashes(dataset_id, names, hashes, metas)
```