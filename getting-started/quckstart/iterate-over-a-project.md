---
description: >-
  In this article, we will learn how to iterate through a project with annotated
  data in python. It is one of the most frequent operations in Superviely Apps
  and python automation scripts.
---

# Iterate over a project

{% hint style="info" %}
Everything you need to reproduce this tutorial is on [GitHub](https://github.com/supervisely-ecosystem/iterate-over-project): source code, Visual Studio code configuration, and a shell script for creating [venv](https://docs.python.org/3/library/venv.html).
{% endhint %}

### Demo project

If you don't have any projects yet, go to the ecosystem and add the demo project 🍋 **`Lemons annotated`** to your current workspace.

![Add demo project "Lemons annotated" to your workjspace](https://user-images.githubusercontent.com/12828725/180640631-8636ac88-a8f7-4f72-90bb-84438d12f247.png)

### .env file

Create a file at `~/supervisely.env`. Learn more about environment variables [here](environment-variables.md). The content should look like this:

```python
# default python variables
PYTHONUNBUFFERED=1
LOG_LEVEL="debug"

# your API credentials, learn more here: https://developer.supervise.ly/getting-started/basics-of-authentication
SERVER_ADDRESS="https://app.supervise.ly"
API_TOKEN="4r47N.....blablabla......xaTatb" 

# change the Project ID to your value
modal.state.slyProjectId=12208
```

### Python script

{% hint style="info" %}
This script illustrates only the basics. If your project is huge and has **hundreds of thousands of images** then it is not so efficient to download annotations one by one. It is better to use batch methods to reduce the number of API requests and significantly speed up your code. Learn more in [the optimizations section](iterate-over-a-project.md#undefined) below.
{% endhint %}

Just clone the [repo](https://github.com/supervisely-ecosystem/iterate-over-project), create [venv](https://github.com/supervisely-ecosystem/iterate-over-project/blob/master/create\_venv.sh) and start debugging:

```python
import os
import supervisely as sly
from dotenv import load_dotenv


load_dotenv(os.path.expanduser("~/supervisely.env"), verbose=True)
api = sly.Api.from_env()

project_id = int(os.environ["modal.state.slyProjectId"])
project = api.project.get_info_by_id(project_id)
if project is None:
    raise KeyError(f"Project with ID {project_id} not found in your account")
print(f"Project info: {project.name} (id={project.id})")

# get project meta - list of annotation classes and tags
meta_json = api.project.get_meta(project.id)
project_meta = sly.ProjectMeta.from_json(meta_json)
print(project_meta)

datasets = api.dataset.get_list(project.id)
print(f"There are {len(datasets)} datasets in project")

for dataset in datasets:
    print(f"Dataset {dataset.name} has {dataset.items_count} images")
    images = api.image.get_list(dataset.id)
    for image in images:
        ann_json = api.annotation.download_json(image.id)
        ann = sly.Annotation.from_json(ann_json, project_meta)
        print(f"There are {len(ann.labels)} objects on image {image.name}")
```

### Output

The script above produces the following output:

```
Project info: Lemons (Annotated) (id=12208)
ProjectMeta:
Object Classes
+-------+--------+----------------+--------+
|  Name | Shape  |     Color      | Hotkey |
+-------+--------+----------------+--------+
|  kiwi | Bitmap |  [255, 0, 0]   |        |
| lemon | Bitmap | [81, 198, 170] |        |
+-------+--------+----------------+--------+
Tags
+------+------------+-----------------+--------+---------------+--------------------+
| Name | Value type | Possible values | Hotkey | Applicable to | Applicable classes |
+------+------------+-----------------+--------+---------------+--------------------+
+------+------------+-----------------+--------+---------------+--------------------+

There are 1 datasets in project
Dataset ds1 has 6 images
There are 3 objects on image IMG_1836.jpeg
There are 4 objects on image IMG_8144.jpeg
There are 4 objects on image IMG_3861.jpeg
There are 3 objects on image IMG_0748.jpeg
There are 5 objects on image IMG_4451.jpeg
There are 7 objects on image IMG_2084.jpeg
```

### Optimizations

Coming soon.
