# Iterate over a local project

In this article, we will learn how to iterate through a project in [Supervisely format](https://developer.supervisely.com/api-references/supervisely-annotation-json-format), which is stored locally on your machine. It is one of the most frequent operations in Superviely Apps and python automation scripts. You will see how easy it is to get all the necessary information from the project, as well as how quickly you can visualize the contents of the project even without internet access.

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/iterate-over-local-project): source code, Visual Studio code configuration, and a shell script for creating venv.
{% endhint %}

In this guide we will go through the following steps:

\*\*\*\* [**Step 1.**](iterate-over-a-local-project.md#1.-demo-project) Get a [demo project](https://ecosystem.supervisely.com/projects/lemons-annotated) with labeled lemons and kiwis.

\*\*\*\* [**Step 2.**](iterate-over-a-local-project.md#2.-download-project-in-supervisely-format) Download the demo project to your local machine in Supervisely format using [Export to Supervisely format](https://ecosystem.supervisely.com/apps/export-to-supervisely-format) app in Supervisely Ecosystem.

\*\*\*\* [**Step 3.**](iterate-over-a-local-project.md#3.-python-script) Run [python script](https://github.com/supervisely-ecosystem/iterate-over-local-project/blob/master/main.py).

### 1. Demo project

If you don't have any projects in Supervisely format on your local machine, go to the ecosystem and add the demo project 🍋 **`Lemons annotated`** to your current workspace.

![Add demo project "Lemons annotated" to your workjspace](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/249098761-1a3652a0-c6b3-423e-ad5e-25d614b3cc2b.png)

### 2. Download project in Supervisely format

Go to the ecosystem and launch the app [Export to Supervisely format](https://ecosystem.supervisely.com/apps/export-to-supervisely-format). Select the demo project, you created in the previous step (or use any existing project in Supervisely), and download the result archive to your local machine.

![Run Export to Supervisely format app](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/249098782-5c08cbc0-6305-4185-8476-571d35cf95ba.png)

![Download the result archive](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/249098794-d1a5dc52-2b3f-440a-b29e-0127cbe8b5f3.png)

Extract the archive to any folder and check that it has the following structure:

```
. 📦 (project root)
├── 📂 ds1 (dataset name)
│   ├── 📂 ann
│   │   ├── 📜 IMG_0748.jpeg.json
│   │   ├── 📜 IMG_1836.jpeg.json
│   │   ├── 📜 IMG_2084.jpeg.json
│   │   ├── 📜 IMG_3861.jpeg.json
│   │   ├── 📜 IMG_4451.jpeg.json
│   │   └── 📜 IMG_8144.jpeg.json
│   └── 📂 img
│       ├── 🖼️ IMG_0748.jpeg
│       ├── 🖼️ IMG_1836.jpeg
│       ├── 🖼️ IMG_2084.jpeg
│       ├── 🖼️ IMG_3861.jpeg
│       ├── 🖼️ IMG_4451.jpeg
│       └── 🖼️ IMG_8144.jpeg
└── 📜 meta.json
```

The project root directory contains the `meta.json` file with the project meta information and directories for each dataset. Each dataset directory contains two subdirectories: `img` with images and `ann` with annotations in Supervisely format.

### 3. Python script

To start debugging you need to:

1. Clone the [repo](https://github.com/supervisely-ecosystem/iterate-over-local-project)
2. Create [venv](https://docs.python.org/3/library/venv.html) by running the script [`create_venv.sh`](https://github.com/supervisely-ecosystem/iterate-over-local-project/blob/master/create_venv.sh)
3. Set the correct path to the downloaded and extracted project on your local machine in the script [`main.py`](https://github.com/supervisely-ecosystem/iterate-over-local-project/blob/master/main.py)

#### Source code:

```python
import os
import json

import supervisely as sly
from tqdm import tqdm

input = "./lemons-fs"
output = "./results"
os.makedirs(output, exist_ok=True)

# Creating Supervisely project from local directory.
project = sly.Project(input, sly.OpenMode.READ)
print("Opened project: ", project.name)
print("Number of images in project:", project.total_items)

# Showing annotations tags and classes.
print(project.meta)

# Iterating over classes in project, showing their names, geometry types and colors.
for obj_class in project.meta.obj_classes:
    print(
        f"Class '{obj_class.name}': geometry='{obj_class.geometry_type}', color='{obj_class.color}'",
    )

# Iterating over tags in project, showing their names and colors.
for tag in project.meta.tag_metas:
    print(f"Tag '{tag.name}': color='{tag.color}'")

print("Number of datasets (aka folders) in project:", len(project.datasets))

progress = tqdm(project.datasets, desc="Processing datasets")
for dataset in project.datasets:
    # Iterating over images in dataset, using the paths to the images and annotations.
    for item_name, image_path, ann_path in dataset.items():
        print(f"Item '{item_name}': image='{image_path}', ann='{ann_path}'")

        ann_json = json.load(open(ann_path))
        ann = sly.Annotation.from_json(ann_json, project.meta)

        img = sly.image.read(image_path)  # rgb - order

        for label in ann.labels:
            # Drawing each label on the image.
            label.draw(img)

        res_image_path = os.path.join(output, item_name)
        sly.image.write(res_image_path, img)

        # Or alternatively draw annotation (all labels at once) preview with
        # ann.draw_pretty(img, output_path=res_image_path)

        progress.update(1)
```

#### Output

The script above produces the following output:

```
Opened project:  lemons-fs
Number of images in project: 6
ProjectMeta:
Object Classes
+-------+--------+----------------+--------+
|  Name | Shape  |     Color      | Hotkey |
+-------+--------+----------------+--------+
|  kiwi | Bitmap |  [255, 0, 0]   |        |
| lemon | Bitmap | [81, 198, 170] |        |
+-------+--------+----------------+--------+
Tags
+----------+------------+-----------------+--------+---------------+--------------------+
|   Name   | Value type | Possible values | Hotkey | Applicable to | Applicable classes |
+----------+------------+-----------------+--------+---------------+--------------------+
+----------+------------+-----------------+--------+---------------+--------------------+

Class 'kiwi': geometry='<class 'supervisely.geometry.bitmap.Bitmap'>', color='[255, 0, 0]'
Class 'lemon': geometry='<class 'supervisely.geometry.bitmap.Bitmap'>', color='[81, 198, 170]'
Number of datasets (aka folders) in project: 1

Item 'IMG_4451.jpeg': image='./lemons-fs/ds1/img/IMG_4451.jpeg', ann='./lemons-fs/ds1/ann/IMG_4451.jpeg.json'
Item 'IMG_0748.jpeg': image='./lemons-fs/ds1/img/IMG_0748.jpeg', ann='./lemons-fs/ds1/ann/IMG_0748.jpeg.json'
Item 'IMG_1836.jpeg': image='./lemons-fs/ds1/img/IMG_1836.jpeg', ann='./lemons-fs/ds1/ann/IMG_1836.jpeg.json'
Item 'IMG_3861.jpeg': image='./lemons-fs/ds1/img/IMG_3861.jpeg', ann='./lemons-fs/ds1/ann/IMG_3861.jpeg.json'
Item 'IMG_2084.jpeg': image='./lemons-fs/ds1/img/IMG_2084.jpeg', ann='./lemons-fs/ds1/ann/IMG_2084.jpeg.json'
Item 'IMG_8144.jpeg': image='./lemons-fs/ds1/img/IMG_8144.jpeg', ann='./lemons-fs/ds1/ann/IMG_8144.jpeg.json'
```

As a result of running the script there also will be created a directory `results`, which will contain the images with drawn annotations.

![Result images with drawn annotations](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/249098806-9b6aa95e-81f1-41eb-8a94-f87655362785.png)
