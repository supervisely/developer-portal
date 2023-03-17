# Point Cloud Episodes

![3D Episodes labeling interface](../../.gitbook/assets/3d\_episodes\_interface.png)

## Project Structure Example

```
<PROJECT_NAME>
├── key_id_map.json (optional)                
├── meta.json     
├── <EPISODE_NAME_1>                       
│ ├── annotation.json           
│ ├── frame_pointcloud_map.json     
│ ├── pointcloud                    
│ │   ├── scene_1.pcd           
│ │   ├── scene_2.pcd   
│ │   └── ...                
│ └── related_images (optional)        
│     ├── scene_1_pcd               
│     │ ├── scene_1_cam0.png       
│     │ ├── scene_1_cam0.png.json  
│     │ ├── scene_1_cam1.png       
│     │ ├── scene_1_cam1.png.json  
│     │ └── ... 
│     ├── scene_2_pcd               
│     │ ├── scene_2_cam0.png       
│     │ ├── scene_2_cam0.png.json  
│     │ ├── scene_2_cam1.png       
│     │ ├── scene_2_cam1.png.json  
│     │ └── ... 
│     └── ...      
├── <EPISODE_NAME_2>                       
│ ├── annotation.json               
│ ├── frame_pointcloud_map.json     
│ ├── pointcloud                    
│ │   ├── scene_1.pcd                 
│ │   └── ...               
│ └── related_images (optional)          
│     ├── scene_1_pcd               
│     │ ├── scene_1_cam0.png       
│     │ ├── scene_1_cam0.png.json    
│     │ └── ... 
│     ├── scene_2_pcd               
│     │ ├── scene_2_cam0.png       
│     │ ├── scene_2_cam0.png.json  
│     │ └── ... 
│     └── ...                    
└── <EPISODE_NAME_...>                       
```

## Main concepts

**Point cloud Episodes Project**

Point cloud Episodes (PCE) Project consists of one or many sequences of frames. Each sequence is called an episode/dataset in Supervisely.

PCE also includes Sensor fusion feature that supports video camera sensor in the Labeling Tool UI.

**Project Meta (meta.json)**

Project Meta contains the essential information about the project - Classes and Tags. These are defined project-wide and can be used for labeling in every episode inside the current project.

**Episodes/Datasets (\<EPISODE\_NAME\_1>, \<EPISODE\_NAME\_2>, ...)**

Episodes are the second level folders inside the project, they host a sequence of frames (pointclouds), related photo context (images) and annotations.

**Items/Point clouds (pointcloud)**

Every `.pcd` file in a sequence has to be stored inside a `pointcloud` folder of episodes.

| Key | Value                                                     |
| --- | --------------------------------------------------------- |
| x   | The x coordinate of the point.                            |
| y   | The y coordinate of the point.                            |
| z   | The z coordinate of the point.                            |
| r   | The red color channel component. An 8-bit value (0-255).  |
| g   | The green color channel component. An 8-bit value (0-255) |
| b   | The blue color channel component. An 8-bit value (0-255)  |

All of the positional coordinates (x, y, z) are in meters. Supervisely supports all PCD encodings: ASCII, binary, binary\_compressed.

The PCD file format description can be found [here](https://pointclouds.org/documentation/tutorials/pcd\_file\_format.html)

**Items Annotation (annotation.json)**

Point cloud Episode Annotation contains the information for the entire episode including labels on all point clouds (frames) in the episode and objects. The mapping between frame numbers and point cloud names is specified in the file `frame_pointcloud_map.json` which guarantees the order.

An episode contains a list of objects that are used to track labels between frames. The list of objects is defined for the entire episode

Figures represents individual labels on frames. Label contains information about the geometry, frame number and object that it belongs to.

```json
[
    {
    "description": "",
    "key": "e9f0a3ae21be41d08eec166d454562be",
    "tags": [],
    "objects": [
        {
            "key": "6663ca1d20c74bea83bd48c24568989d",
            "classTitle": "car",
            "tags": []
        }],
    "framesCount": 48,
    "frames": [
         {
            "index": 0,
            "figures": [
               {
                "key": "cb8e067dadfc423aa8575a0c4e62de33",
                "objectKey": "6663ca1d20c74bea83bd48c24568989d",
                "geometryType": "cuboid_3d",
                "geometry": {
                    "position": {
                        "x": -10.863547325134277,
                        "y": -93.57706451416016,
                        "z": -4.598618030548096
                    },
                    "rotation": {
                        "x": 0,
                        "y": 0,
                        "z": 3.250733629393711
                    },
                    "dimensions": {
                        "x": 1.978,
                        "y": 4.607,
                        "z": 1.552
                        }
                      }
                    }
            ]
         },
         {
            "index": 1,
            "figures": [               
               {
                "key": "71e0fe52dc4f4f6aaf059ad095f43c1f",
                "objectKey": "6663ca1d20c74bea83bd48c24568989d",
                "labelerLogin": "username",
                "updatedAt": "2021-11-11T17:19:11.448Z",
                "createdAt": "2021-11-11T16:53:03.670Z",
                "geometryType": "cuboid_3d",
                "geometry": {
                    "position": {
                        "x": -11.10418701171875,
                        "y": -91.33098602294922,
                        "z": -4.5446248054504395
                    },
                    "rotation": {
                        "x": 0,
                        "y": 0,
                        "z": 3.24780199600921
                    },
                    "dimensions": {
                        "x": 1.978,
                        "y": 4.607,
                        "z": 1.552
                    }
                }
              }
            ]
         }
    ]
    }
]
```

**Optional fields and loading** These fields are optional and are not needed when loading the project. The server can automatically fill in these fields while project is loading.

* `id` - unique identifier of the current object
* `classId` - unique class identifier of the current object
* `labelerLogin` - string - the name of user who created the current figure
* `createdAt` - string - date and time of figure creation
* `updatedAt` - string - date and time of the last figure update

Main idea of `key` fields and `id` you can see below in [Key id map file](point-cloud-episodes.md#key-id-map-file) section.

**Fields definitions:**

* `description` - string - (optional) - this field is used to store the text to assign to the sequence.
* `key` - string, unique key for a given sequence (used in key\_id\_map.json to get the sequence ID)
* `tags` - list of strings that will be interpreted as episode tags
* `objects` - list of objects that may be present on the episode
* `frames` - list of frames of which the episode consists. List contains only frames with an object from the 'objects' field
  * `index` - integer - number of the current frame
  * `figures` - list of figures in the current frame.
* `framesCount` - integer - total number of frames in the episode
* `geometryType` - "cuboid\_3d" or other 3D geometry - class shape

**Fields definitions for `objects` field:**

* `key` - string - unique key for a given object (used in key\_id\_map.json)
* `classTitle` - string - the title of a class. It's used to identify the class shape from the `meta.json` file
* `tags` - list of strings that will be interpreted as object tags (can be empty)

**Fields description for `figures` field:**

* `key` - string - unique key for a given figure (used in key\_id\_map.json)
* `objectKey` - string - unique key to link figure to object (used in key\_id\_map.json)
* `geometryType` - "cuboid\_3d" or other 3D geometry -class shape
* `geometry` - geometry of the object

**Description for `geometry` field (cuboid\_3d):**

* `position` 3D vector of box center coordinates:
  * **x** - forward in the direction of the object
  * **y** - left
  * **z** - up
* `dimensions` is a 3D vector that scales a cuboid from its local center along x,y,z:
  * **x** - width
  * **y** - length
  * **z** - height
* `rotation` is a 3D Vector that rotates a cuboid along an axis in world space:
  * **x** - pitch
  * **y** - roll
  * **z** - yaw (direction)

### Cuboid direction vector

Rotation values bound inside \[**-pi** ; **pi** ] When `yaw = 0` box direction will be strict `+y`

## Key id map file

The basic idea behind key-id-map is that it maps the unique identifiers of entities from Supervisely to local entities keys. It is needed for such local data manipulations as cloning entities and reassigning relations between them. Examples of entities in `key_id_map.json`: datasets (videos/episodes), tags, objects, figures.

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
        "e9f0a3ae21be41d08eec166d454562be": 42656
    }
}
```

* `objects` - dictionary, where the key is a unique string, generated inside Supervisely environment to set mapping of current object in annotation, and values are unique integer ID related to the current object
* `figures` - dictionary, where the key is a unique string, generated inside Supervisely environment to set mapping of object on current frame in annotation, and values are unique integer ID related to the current frame
* `videos` - dictionary, where the key is unique string, generated inside Supervisely environment to set mapping of episode (dataset) in annotation, and value is a unique integer ID related to the current sequence
* `tags` - dictionary, where the keys are unique strings, generated inside Supervisely environment to set mapping of tag on current frame in annotation, and values are a unique integer ID related to the current tag
* **Key** - [generated by python3 function `uuid.uuid4().hex`](https://docs.python.org/3/library/uuid.html#uuid.uuid4). The unique string. All key values and id's should be unique inside single project and can not be shared between frames\sequences.
* **Value** - returned by server integer identifier while uploading object / figure / sequence / tag

## Format of frame\_pointcloud\_map.json

This file stores mapping between point cloud files and annotation frames in the correct order.

```json
{
    "0" : "frame1.pcd",
    "1" : "frame2.pcd",
    "2" : "frame3.pcd" 
}
```

**Keys** - frame order number\
**Values** - point cloud name (with extension)

## Photo context image annotation file

```json
    {
        "name": "host-a005_cam4_1231201437716091006.jpeg",
        "entityId": 2359620,
        "meta": {
            "deviceId": "CAM_BACK_LEFT",
            "timestamp": "2019-01-11T03:23:57.802Z",
            "sensorsData": {
                "extrinsicMatrix": [
                    -0.8448329028461443,
                    -0.5350302199120708,
                    0.00017334762588639086,
                    -0.012363736761232369,
                    -0.0035124448582330757,
                    0.005222293412494302,
                    -0.9999801949951969,
                    -0.16621728572112304,
                    0.5350187183638307,
                    -0.8448167798004226,
                    -0.006291229448121315,
                    -0.3527897896721229
                ],
                "intrinsicMatrix": [
                    882.42699274,
                    0,
                    602.047851885,
                    0,
                    882.42699274,
                    527.99972239,
                    0,
                    0,
                    1
                ]
            }
        }
    }
```

**Fields description:**

* name - string - Name of image file
* entityId (OPTIONAL) - integer >= 1 ID of the Point Cloud in the system, that photo attached to. Doesn't required while uploading.
* deviceId - string - Device ID or name.
* timestamp - (OPTIONAL) - string - Time when the frame occurred in ISO 8601 format
* sensorsData - Sensors data such as Pinhole camera model parameters. See wiki: [Pinhole camera model](https://en.wikipedia.org/wiki/Pinhole\_camera\_model) and [OpenCV docs for 3D reconstruction](https://docs.opencv.org/2.4/modules/calib3d/doc/camera\_calibration\_and\_3d\_reconstruction.html).
  * intrinsicMatrix - Array of number - 3x3 flatten matrix (dropped last zeros column) of intrinsic parameters in row-major order, also called camera matrix. It's used to denote camera calibration parameters. See [Intrinsic parameters](https://en.wikipedia.org/wiki/Camera\_resectioning#Intrinsic\_parameters).
  * extrinsicMatrix - Array of number - 4x3 flatten matrix (dropped last zeros column) of extrinsic parameters in row-major order, also called joint rotation-translation matrix. It's used to denote the coordinate system transformations from 3D world coordinates to 3D camera coordinates. See [Extrinsic\_parameters](https://en.wikipedia.org/wiki/Camera\_resectioning#Extrinsic\_parameters).

## Related apps

1. [Import Supervisely pointcloud episodes](https://ecosystem.supervise.ly/apps/import-pointcloud-episode) app.

![](https://i.imgur.com/JRM9WXO.png)

1. [Export Supervisely pointcloud episodes](https://ecosystem.supervise.ly/apps/export-pointcloud-episode) app.

![](https://i.imgur.com/cnXCPVx.png)

## Example projects

1. [Demo LYFT 3D dataset annotated](https://app.supervise.ly/ecosystem/projects/demo-lyft-3d-dataset-annotated) - demo sample from [Lyft](https://level-5.global/data) dataset with labels.

![](https://user-images.githubusercontent.com/97401023/192003812-1cefef97-29e3-40dd-82c6-7d3cf3d55585.png)

1. [Demo LYFT 3D dataset](https://app.supervise.ly/ecosystem/projects/demo-lyft-3d-dataset) - demo sample from [Lyft](https://level-5.global/data) dataset without labels.

![](https://user-images.githubusercontent.com/97401023/192003862-102de613-d365-4043-8ca0-d59e3c95659a.png)

1. [Demo KITTI pointcloud episodes annotated](https://app.supervise.ly/ecosystem/projects/demo-kitti-3d-episodes-annotated) - demo sample from [KITTI 3D](https://www.cvlibs.net/datasets/kitti/eval\_tracking.php) dataset with labels.

![](https://user-images.githubusercontent.com/97401023/192003917-71425add-e985-4a9c-8739-df832324be2f.png)

1. [Demo KITTI pointcloud episodes](https://app.supervise.ly/ecosystem/projects/demo-kitti-3d-episodes) - demo sample from [KITTI 3D](https://www.cvlibs.net/datasets/kitti/eval\_tracking.php) dataset without labels.

![](https://user-images.githubusercontent.com/97401023/192003975-972c1803-b502-4389-ae83-72958ddd89ad.png)
