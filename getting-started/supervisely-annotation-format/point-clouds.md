# Point Cloud Annotation

<figure><img src="https://github.com/supervisely/developer-portal/raw/main/.gitbook/assets/3d_pointclouds_interface.png" alt=""><figcaption><p>3D Point Cloud labeling interface</p></figcaption></figure>

## Project Structure Example

```
<PROJECT_NAME>  
├── key_id_map.json (optional)              
├── meta.json     
├── <DATASET_NAME_1>                         
│ ├── pointcloud                    
│ │   ├── scene_1.pcd           
│ │   ├── scene_2.pcd   
│ │   └── ...                
│ ├── related_images (optional)               
│ │   ├── scene_1_pcd               
│ │   │ ├── scene_1_cam0.png       
│ │   │ ├── scene_1_cam0.png.json  
│ │   │ ├── scene_1_cam1.png       
│ │   │ ├── scene_1_cam1.png.json  
│ │   │ └── ... 
│ │   ├── scene_2_pcd               
│ │   │ ├── scene_2_cam0.png       
│ │   │ ├── scene_2_cam0.png.json  
│ │   │ ├── scene_2_cam1.png       
│ │   │ ├── scene_2_cam1.png.json  
│ │   │ └── ... 
│ │   └── ...      
│ └── ann
│     ├── scene_1.pcd.json
│     ├── scene_2.pcd.json
│     └── ...     
├── <DATASET_NAME_2>                     
│ ├── pointcloud                    
│ │   ├── scene_1.pcd
│ │   └── ...                
│ ├── related_images (optional)               
│ │   ├── scene_1_pcd               
│ │   │ ├── scene_1_cam0.png       
│ │   │ ├── scene_1_cam0.png.json  
│ │   │ ├── scene_1_cam1.png       
│ │   │ ├── scene_1_cam1.png.json  
│ │   │ └── ... 
│ │   └── ...      
│ └── ann
│     ├── scene_1.pcd.json
│     └── ...                      
└── <DATASET_NAME_...>                       
```

## Main concepts

**Point cloud Project**

Point cloud Project consists of one or many datasets of point clouds.

It also includes Sensor fusion feature that supports video camera sensor in the Labeling Tool UI.

**Project Meta (meta.json)**

Project Meta contains the essential information about the project - Classes and Tags. These are defined project-wide and can be used for labeling in every dataset inside the current project.

**Datasets (\<DATASET\_NAME\_1>, \<DATASET\_NAME\_2>, ...)**

Datasets are the second level folders inside the project, they host subsets of point cloud scenes, related photo context (images) and annotations.

**Items/Point clouds (pointcloud)**

Every `.pcd` file in a sequence has to be stored inside a `pointcloud` folder of datasets.

| Key | Value                                                     |
| --- | --------------------------------------------------------- |
| x   | The x coordinate of the point.                            |
| y   | The y coordinate of the point.                            |
| z   | The z coordinate of the point.                            |
| r   | The red color channel component. An 8-bit value (0-255).  |
| g   | The green color channel component. An 8-bit value (0-255) |
| b   | The blue color channel component. An 8-bit value (0-255)  |

All the positional coordinates (x, y, z) are in meters. Supervisely supports all PCD encoding: ASCII, binary, binary\_compressed.

The PCD file format description can be found [here](https://pointclouds.org/documentation/tutorials/pcd\_file\_format.html)

**Items Annotations (ann)**

Point cloud Annotations refer to each point cloud and contains information about labels on the point clouds in the datasets.

A dataset has a list of `objects` that can be shared between some point clouds.

The list of `objects` is defined for the entire dataset, even if the object's figure occurs in only one point cloud.

`Figures` represents individual labels, attached to one single frame and its object.

```
{
    "description": "",
    "key": "e9f0a3ae21be41d08eec166d454562be",
    "tags": [],
    "objects": [
        {
            "key": "ecb975d70735486b90fe4fdd2be77e3b",
            "classTitle": "Car",
            "tags": [],
            "labelerLogin": "admin",
            "updatedAt": "2022-05-04T00:32:30.872Z",
            "createdAt": "2022-05-04T00:32:30.872Z"
        }
    ],
    "figures": [
        {
            "key": "abbaec8785c1468585f6210c62bb2374",
            "objectKey": "ecb975d70735486b90fe4fdd2be77e3b",
            "geometryType": "cuboid_3d",
            "geometry": {
                "position": {
                    "x": 58.756710052490234,
                    "y": 4.623323917388916,
                    "z": -0.4174150526523591
                },
                "rotation": {
                    "x": 0,
                    "y": 0,
                    "z": -1.77
                },
                "dimensions": {
                    "x": 1.59,
                    "y": 4.28,
                    "z": 1.45
                }
            },
            "labelerLogin": "admin",
            "updatedAt": "2022-05-04T00:32:37.432Z",
            "createdAt": "2022-05-04T00:32:37.432Z"
        }
    ]
}
```

**Optional fields and loading** These fields are optional and are not needed when loading the project. The server can automatically fill in these fields while project is loading.

* `id` - unique identifier of the current object
* `classId` - unique class identifier of the current object
* `labelerLogin` - string - the name of user who created the current figure
* `createdAt` - string - date and time of figure creation
* `updatedAt` - string - date and time of the last figure update

Main idea of `key` fields and `id` you can see below in [Key id map file](point-clouds.md#key-id-map-file) section.

**Fields definitions:**

* `description` - string - (optional) - this field is used to store the text to assign to the sequence.
* `key` - string, unique key for a given sequence (used in key\_id\_map.json to get the sequence ID)
* `tags` - list of strings that will be interpreted as point cloud tags
* `objects` - list of objects that may be present on the dataset
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
* `dimensions` is a 3D vector that scales a cuboid from its local center along x, y, z:
  * **x** - width
  * **y** - length
  * **z** - height
* `rotation` is a 3D Vector that rotates a cuboid along an axis in world space:
  * **x** - pitch
  * **y** - roll
  * **z** - yaw (direction)

Rotation values bound inside \[**-pi** ; **pi**] When `yaw = 0` box direction will be strict `+y`

## Key id map file

The basic idea behind key-id-map is that it maps the unique identifiers of entities from Supervisely to local entities keys. It is needed for such local data manipulations as cloning entities and reassigning relations between them. Examples of entities in `key_id_map.json`: datasets (videos), tags, objects, figures.

```
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
* `videos` - dictionary, where the key is unique string, generated inside Supervisely environment to set mapping of dataset in annotation, and value is a unique integer ID related to the current sequence
* `tags` - dictionary, where the keys are unique strings, generated inside Supervisely environment to set mapping of tag on current frame in annotation, and values are a unique integer ID related to the current tag
* **Key** - [generated by python3 function `uuid.uuid4().hex`](https://docs.python.org/3/library/uuid.html#uuid.uuid4). The unique string. All key values and ID's should be unique inside single project and can not be shared between frames\sequences.
* **Value** - returned by server integer identifier while uploading object / figure / sequence / tag

## Format of frame\_pointcloud\_map.json

This file stores mapping between point cloud files and annotation frames in the correct order.

```
{
    "0" : "frame1.pcd",
    "1" : "frame2.pcd",
    "2" : "frame3.pcd" 
}
```

**Keys** - frame order number\
**Values** - point cloud name (with extension)

## Photo context image annotation file

```
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

1\. [Import Point Cloud Project](https://ecosystem.supervisely.com/apps/import-pointcloud-project) app.

[![](https://user-images.githubusercontent.com/97401023/193620195-6481801e-0fc5-4ac3-858f-cb3a294defac.png)](https://user-images.githubusercontent.com/97401023/193620195-6481801e-0fc5-4ac3-858f-cb3a294defac.png)

2\. [Export pointclouds project in Supervisely format](https://ecosystem.supervisely.com/apps/export-pointclouds-project-in-supervisely-format) app.

[![](https://user-images.githubusercontent.com/97401023/193619296-df4ea2b2-e26c-42c2-b98a-bbe578c67fdb.png)](https://user-images.githubusercontent.com/97401023/193619296-df4ea2b2-e26c-42c2-b98a-bbe578c67fdb.png)

## Example projects

1\. [Demo pointcloud project](https://ecosystem.supervisely.com/projects/demo-pointcloud-project)

[![](https://user-images.githubusercontent.com/97401023/193617265-431aa000-ae57-4beb-aa9b-8ba31d755b74.png)](https://user-images.githubusercontent.com/97401023/193617265-431aa000-ae57-4beb-aa9b-8ba31d755b74.png)

2\. [Demo pointcloud project with labels](https://ecosystem.supervisely.com/projects/demo-pointcloud-project-annotated)

[![](https://user-images.githubusercontent.com/97401023/193617359-2b929837-901e-4d98-92b8-cecb32d8f3af.png)](https://user-images.githubusercontent.com/97401023/193617359-2b929837-901e-4d98-92b8-cecb32d8f3af.png)
