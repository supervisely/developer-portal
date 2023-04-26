# Using Progress Bar with tqdm

## Introduction

In this tutorial we will show you how to use tqdm module inside methods of Supervisely SDK in a seamless manner.

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-tqdm): source code.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-tqdm) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-tqdm.git

cd tutorial-tqdm

./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code .
```

**Step 4.** Change project ID in `local.env` file by copying the ID from the context menu of the workspace.

<!-- <screen> -->

```
PROJECT_ID=17732 # ⬅️ change value
TEAM=449 # ⬅️ change value
```

**Step 5.** Start debugging `src/main.py`.

### Import libraries

```python
import os
import time
from dotenv import load_dotenv

from tqdm import tqdm

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
project_id = sly.env.project_id()
team_id = sly.env.team_id()
```

## Use tqdm for tracking progress

### Example 1. Use tqdm in the loop.

**Source code:**

```python
batch_size = 10
data = range(100)

with tqdm(total=len(data)) as pbar:
    for batch in sly.batched(data, batch_size):
        for item in batch:
            time.sleep(0.1)
        pbar.update(batch_size)
```

**Output:**

![example_1_1](https://user-images.githubusercontent.com/78355358/234561243-f58d3821-29c1-4ece-bf93-0dabed6d7329.gif)

{% hint style="info" %}

When running locally, the fancy-looking tqdm progress bar will be displayed in the console, while in production, JSON-looking lines with relevant information will be logged.

{% endhint %}

![example_1_2](https://user-images.githubusercontent.com/78355358/234561433-1570eb19-7f78-4f74-bf37-98bc6d1725c2.gif)

### Example 2. Download image project and upload it into Team files using tqdm progress bar.

**Source code:**

Download your project with previously initiialized `project_id`

```python
    n_count = api.project.get_info_by_id(project_id).items_count
    p = tqdm(desc="Downloading", total=n_count)

    sly.download(api, project_id, 'your/local/dir/', progress_cb=p)
```

Then, you can upload downloaded directory to Team files:

```python
    p = tqdm(
        desc="Uploading",
        total=sly.fs.get_directory_size('your/local/dir/'),
        unit="B",
        unit_scale=True,
    )

    api.file.upload_directory(
        team_id,
        'your/local/dir/',
        '/your/teamfiles/dir/',
        progress_size_cb=p,
    )
```

**Output:**

![example_2_1](https://user-images.githubusercontent.com/78355358/234561669-eef09fce-518b-4cd0-9550-63c8da962131.gif)

![example_2_2](https://user-images.githubusercontent.com/78355358/234564625-f9ff0ab2-2775-44c8-b2fc-96a661828298.gif)

### Example 3 (advanced). Use native sly.Progress functions for downloading.

Let's reproduce previous example with Supervisely's native Progress bar.

```python
    n_count = api.project.get_info_by_id(project_id).items_count
    p = sly.Progress("Downloading", n_count)

    sly.download(api, project_id, 'your/local/dir/', progress_cb=p)
```

You will get files in progress.

Then, you can upload downloaded directory to Team files:

```python
    p = sly.Progress(
        "Uploading",
        sly.fs.get_directory_size('your/local/dir/'),
        is_size=True,
    )
    api.file.upload_directory(
        team_id,
        'your/local/dir/',
        '/your/teamfiles/dir/',
        progress_size_cb=p,
    )
```

**Output:**

![example_3_1](https://user-images.githubusercontent.com/78355358/234561956-0002d4f7-fb96-44d5-b346-88f7b83f4714.gif)

![example_3_2](https://user-images.githubusercontent.com/78355358/234562015-2bde0cfa-3fcc-4a6b-a3f5-f3cb7203cc8b.gif)

{% hint style="info" %}

You can swap equivalent arguments from `sly.Progress` while initializing `tqdm`. For example, the `desc` argument can be replaced with `message`, and `total` can be replaced with `total_cnt`. Additionally, both `unit` and `unit_scale` can be replaced with `is_size`."

{% endhint %}
