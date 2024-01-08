# Progress Bar tqdm

## Introduction

In this tutorial we will show you how to use [tqdm](https://github.com/tqdm/tqdm) [![GitHub Org's stars](https://img.shields.io/github/stars/tqdm/tqdm?style=social)](https://github.com/tqdm/tqdm) module inside methods of Supervisely SDK in a seamless manner.

{% hint style="info" %}
🔥 With this update, any sly.Progress object can be easily replaced with tqdm, allowing you to seamlessly integrate your progress tracking with the powerful features of tqdm. Say goodbye to headaches!
{% endhint %}

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-tqdm): source code.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

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

import supervisely as sly
from tqdm import tqdm

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

![Example 1a](https://user-images.githubusercontent.com/78355358/234921276-11775b9c-4d9c-45ca-96a1-67d2b7782c75.gif)

{% hint style="info" %}
When running locally, the fancy-looking tqdm progress bar will be displayed in the console, while in production, JSON-looking lines with relevant information will be logged and fancy-looking progress bar will be shown in Workspace Tasks.
{% endhint %}

![Example 1b](https://user-images.githubusercontent.com/78355358/234921675-04e71707-8d5d-48d7-81c5-1d198c94055d.gif)

![This is how progress bar might look like in the Workspace Tasks](https://user-images.githubusercontent.com/78355358/234922284-c0796602-4ea5-423a-9756-9bc113491a58.gif)

### Example 2. Download image project and upload it into Team files using tqdm progress bar.

**Source code:**

Download your project with previously initiialized `project_id`

```python
    n_count = api.project.get_info_by_id(project_id).items_count
    p = tqdm(desc="Downloading", total=n_count)

    sly.download(api, project_id, 'your/local/dir/', progress_cb=p)
```

**Output:**

![Example 2a](https://user-images.githubusercontent.com/78355358/234924224-4f0b2dac-78a1-418e-a235-37ca60d8b9f4.gif)

Then, you can upload downloaded directory to Team files:

**Source code:**

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

![Example 2b](https://user-images.githubusercontent.com/78355358/234924484-c7fdb17f-41fc-4284-a251-866dda36d532.gif)

### Example 3 (advanced). Use native sly.Progress functions for downloading.

Let's reproduce previous example with Supervisely's native Progress bar.

**Source code:**

```python
    n_count = api.project.get_info_by_id(project_id).items_count
    p = sly.Progress("Downloading", n_count)

    sly.download(api, project_id, 'your/local/dir/', progress_cb=p)
```

**Output:**

![Example 3a](https://user-images.githubusercontent.com/78355358/234925098-dcff5061-5981-434e-a966-ef59d3c050de.gif)

You will get files in progress.

Then, you can upload downloaded directory to Team files:

**Source code:**

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

![Example 3b](https://user-images.githubusercontent.com/78355358/234925456-621c800b-0ab5-4111-8ed2-1c48a20ba577.gif)

{% hint style="info" %}
You can swap equivalent arguments from `sly.Progress` while initializing `tqdm`. For example, the `desc` argument can be replaced with `message`, and `total` can be replaced with `total_cnt`. Additionally, both `unit="B"` and `unit_scale=True` can be replaced with `is_size=True`.
{% endhint %}
