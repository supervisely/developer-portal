# Individual Video Annotations

For each video file, we store the annotations in a separate json file named `image_name.image_format.json` with the following file structure:

Example:

![cuboid\_3d example](../../.gitbook/assets/video\_frame.png)

```json
{
    "size": {
        "height": 1080,
        "width": 1920
    },
    "description": "",
    "key": "c8168b43ae1b45c38930f456df9d0f2b",
    "tags": [],
    "objects": [
        {
            "key": "198f727d40c749eebcacc4aed299b39a",
            "classTitle": "rect",
            "tags": [],
            "labelerLogin": "alexxx",
            "updatedAt": "2020-08-23T12:06:11.963Z",
            "createdAt": "2020-08-23T12:06:11.963Z"
        }
    ],
    "frames": [
        {
            "index": 0,
            "figures": [
                {
                    "key": "65f21690780e43b49863c3cbd07eab3a",
                    "objectKey": "198f727d40c749eebcacc4aed299b39a",
                    "geometryType": "rectangle",
                    "geometry": {
                        "points": {
                            "exterior": [
                                [
                                    266,
                                    420
                                ],
                                [
                                    847,
                                    845
                                ]
                            ],
                            "interior": []
                        }
                    },
                    "labelerLogin": "alexxx",
                    "updatedAt": "2020-08-23T12:06:13.544Z",
                    "createdAt": "2020-08-23T12:06:13.544Z"
                }
            ]
        }
    ],
    "framesCount": 375
}
```

**Fields definitions:**

* `size` - string - is equal to image(frame) size
* `description` - string - (optional) -  this field is used to store the text we want to assign to the video. In the labeling interface it corresponds to the 'data' filed.
* `tags` - list of strings that will be interpreted as video tags
* `key` - string, unique key for a given video (used in key\_id\_map.json to get the video ID)
* `objects` - list of objects that may be present on the video
* `frames` - list of frames of which the video consists. List contains only frames with an object from the 'objects' field
  * `index` - integer - number of the current frame
  * `figures` - integer -  list of objects which the current frame contains
* `framesCount` - integer - total number of frames in the video
* `objectKey` - string - unique key for a given object (used in key\_id\_map.json)
* `labelerLogin` - string - the name of a user who created the current figure
* `geometryType` - "cuboid\_3d" - class shape
* `geometry` - a dictionary containing indicators of location, rotation and dimensions of cuboids

**Fields definitions for objects field:**

* `key` - string, a unique key for the given object (used in key\_id\_map.json to get the object ID)
* `classTitle` - string - the title of a class. It's used to identify the class shape from the `meta.json` file
* `tags` - list of strings that will be interpreted as object tags
* `labelerLogin` - string - the name of the user that added this figure to the project

**Fields description for figures field:**

* `key` - string, a unique key for the given figure (used in key\_id\_map.json to get the figure ID)
* `objectKey` - string, a unique key for the given object (used in key\_id\_map.json to get the object ID).&#x20;
* `geometryType` - "rectangle" -class shape
* `geometry` - geometry of the object
* `classTitle` - string - the title of a class. It's used to identify the class shape from the `meta.json` file
* `labelerLogin` - string - the name of the user that added this figure to the current frame

## Key id map file

Key\_id\_map.json file is optional. It is created when annotating the video inside Supervisely interface and sets the correspondence between the unique identifiers of the video, object and the frame on which the object is located. If you annotate manually, you do not need to create this file. This will not affect the work being done.

Json format of key\_id\_map.json:

```json
{
    "tags": {},
    "objects": {
        "198f727d40c749eebcacc4aed299b39a": 20520
    },
    "figures": {
        "65f21690780e43b49863c3cbd07eab3a": 503130811
    },
    "videos": {
        "c8168b43ae1b45c38930f456df9d0f2b": 157876296
    }
}
```

Fields definitions:

* `objects` - dictionary, where the key is a unique string, generated inside Supervisely environment to set correspondence of current object in annotation, and values are unique integer ID corresponding to the current object
* `figures` - dictionary, where the key is a unique string, generated inside Supervisely environment to set correspondence of object on current frame in annotation, and values are unique integer ID corresponding to the current frame
* `videos` - dictionary, where the key is unique string, generated inside Supervisely environment to set correspondence of video in annotation, and value is a unique integer ID corresponding to the current video
* `tags` - dictionary, where the keys are unique strings, generated inside Supervisely environment to set correspondence of tag on current frame in annotation, and values are a unique integer ID corresponding to the current tag

## Common tasks

In this article, we will cover the most common tasks that you can perform with individual video annotations using Supervisely Python SDK.

#### Retrieve video annotation from the API

Don't forget to install the package first:

```bash
pip install supervisely
```

```python
import supervisely as sly

video_id = 123

# First, we'll download the annotation in JSON format.
ann_json = api.video.annotation.download(video_id)

# We recommend to work with sly.VideoAnnotation object, not with raw JSON.
# To convert JSON to sly.VideoAnnotation, you'll need the sly.ProjectMeta.
ann = sly.VideoAnnotation.from_json(ann_json, project_meta, sly.KeyIdMap())
```

Now, you have sly.VideoAnnotation object and you can work with it.

#### Upload video annotation to the API

```python
# Let's assume that you have sly.VideoAnnotation object and video_id.

api.video.annotation.append(video_id, ann)
```

It's pretty simple to complete this task, but now let's make something more complex.

#### Add label (figure) to the video annotation

In this case, we need to add a new label to the video annotation. But let's imagine that we want to create a new empty annotation and add a new label to it.

```python
import supervisely as sly

# First, we'll need to create a new sly.ObjClass.
obj_class = sly.ObjClass(name="new bbox", geometry_type=sly.Rectangle)
geometry = sly.Rectangle(top=10, left=10, bottom=50, right=50)

# Let's imagine we want to add out bounding box to the frame 30.
frame_idx = 30

# And not it's time to create a new VideoObject and a VideoFigure.
video_object = sly.VideoObject(obj_class)
video_figure = sly.VideoFigure(video_object, geometry, frame_idx)

# Prepare a new VideoObjectCollection and a new Frame.
objects =  sly.VideoObjectCollection([video_object])
frame = sly.Frame(frame_idx, figures=[video_figure])
frames = [frame]

# Now we can create a new VideoAnnotation and add the new Frame to it.
video_size = (1080, 1920)
frames_count = 375

new_ann = sly.VideoAnnotation(video_size, frames_count, objects=objects, frames=frames)

# So, our new annotation is ready. But we can't upload it to the server yet, because
# we've created a new ObjClass here and still not added it to the ProjectMeta. Let's do it now.

# Add the new object class to the project meta.
new_project_meta = project_meta.add_obj_class(obj_class)

# Upload the new project meta to the server.
api.project.update_meta(project_id, new_project_meta)

# Finally, we can upload the new annotation to the server.
api.video.annotation.append(video_id, new_ann)
```

That's it! The annotation was replaced on the server with the new one. So, let's take a short summary of what we did:
1. We created a new object class.
2. We created a new label with the new object class and added it to the annotation.
3. We added the new object class to the ProjectMeta.
4. We updated the project meta on the server.
5. We uploaded our new annotation to the server.

When working with annotations, pay attention to make the project meta up-to-date.