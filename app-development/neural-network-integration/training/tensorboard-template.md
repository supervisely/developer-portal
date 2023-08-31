---
description: >-
  Step-by-step tutorial explains how to use custom training script and log
  results in Tensorboard
---

# Tensorboard template

## Introduction

This tutorial will teach you how to integrate your custom training script into Supervisely Ecosystem. The following procedure can be used with any Neural Network architecture and for any Computer Vision task.

It is the simplest integration with of NN training with Supervisely, that do not require any special modifications of your source codes.

📗 GitHub source code of tensorboard training template can be found [here](https://github.com/supervisely-ecosystem/training-tensorboard-template).

{% hint style="info" %}
Note: use this template as a baseline. You can modify any of its parts, for example `run.sh` or `src/train.py`. In case of questions, please contact technical support.
{% endhint %}

The high level overview of the procedure is the following:

1. Take input directory (`--input-dir`) with training data in Supervisely format (see `python3 src/train.py` command at `run.sh`).
2. Transform labeled data (in Supervisely format) to any format you need
3. Train your model (use your training script almost without modifications).
4. Save artifacts (checkpoints and tensorboard metrics) to the output directory (`--output-dir`).
5. After the training all artefacts will be automatically uploaded to Supervisely platform to Team Files.

Full code of training tensorboard template can be found [on github](https://github.com/supervisely-ecosystem/training-tensorboard-template).

Note, that you can always load your previous logs just by simply specifying `HISTORY_DIR` in `run.sh`. Here how it will look like in the tensorboard interface:

![Tensorboard logs with history logs of previous runs. 'output/.' stands here for the current run](https://user-images.githubusercontent.com/78355358/236162006-5dceeb9a-39fa-46a7-9834-eb5c4c1cba89.gif)



## Let's debug template locally

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

```python
SERVER_ADDRESS="https://app.supervise.ly"
API_TOKEN="4r47N.....blablabla......xaTatb" 
```

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/training-tensorboard-template) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/training-tensorboard-template
cd training-tensorboard-template
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code .
```

**Step 4.** Change variables in `local.env` to your values. PROJECT\_ID - id of the project with training data, TEAM\_ID - id of the team where the project is located. [Learn more here.](../../../getting-started/environment-variables.md)

```python
PROJECT_ID=12208 # ⬅️ change it
TEAM_ID=449 # ⬅️ change it
SLY_APP_DATA_DIR="/home/<user>/test-dir" # ⬅️ change it
```

{% hint style="info" %}
Note: the `SLY_APP_DATA_DIR` variable represents a synced data directory that connects locally stored files in a container with the Team files directory. This allows the data to be viewed and copied on the remoted directory in Team files. This directory serves as a backup for the training artefacts in case the training script suddenly crashes. You can view the saved data in `Team Files` -> `Supervisely agents` -> `<chosen node>` ('Main node' by default) -> `app-data` -> `training-tensorboard-template`.
{% endhint %}

**Step 5.** Check self-explanatory `run.sh` script to get the idea how app works. You can modify it the way you need. The resulted directory with output artefacts data will have the following path: `"/my-training/$PROJECT_ID-$PROJECT_NAME/$TASK_ID"`. Note that you can always change the `DST_DIR` in the `run.sh` to suit your needs in any way.

You should also note that in case if you do not have any history logs. (i.e. `*.tfevents.*` files), the script will automatically ignore non-existence of the history folder (`HISTORY_DIR`). It means that you do not need to bother about additional `run.sh` customization!

<details>

<summary>App entrypoint: run.sh script</summary>

```bash
# !/bin/bash
set -e # This will cause the python script to exit immediately if any command exits with a non-zero status.

if [ "$ENV" = "development" ]
then
    source ~/supervisely.env 
    source local.env 
    export SERVER_ADDRESS 
    export API_TOKEN
fi 

INPUT_DIR_LOCAL="/tmp/training_data/"                   # local training data
OUTPUT_DIR_LOCAL="$SLY_APP_DATA_DIR/output/"            # local output artefacts data
# Note: variable $SLY_APP_DATA_DIR is for synced_data_dir which mirrors artefacts data on teamfiles
PROJECT_NAME=$(supervisely project get-name -id $PROJECT_ID)
HISTORY_DIR="/my-training/"                             # teamfiles history logs data
HISTORY_DIR_LOCAL="$SLY_APP_DATA_DIR/history/"          # local history logs data
DST_DIR="/my-training/$PROJECT_ID-$PROJECT_NAME/$TASK_ID" # teamfiles destination directory for output artefacts data

# download project 
supervisely project download -id $PROJECT_ID --dst $INPUT_DIR_LOCAL

# download history artefacts
supervisely teamfiles download -id $TEAM_ID --src "$HISTORY_DIR" --dst "$HISTORY_DIR_LOCAL" --filter ".tfevents." -i

# run tensorboard
nohup tensorboard --logdir_spec output:"$OUTPUT_DIR_LOCAL",history:"$HISTORY_DIR_LOCAL" --port 8000 --host 0.0.0.0 --reload_multifile=true --load_fast=false --path_prefix=$BASE_URL &> output & sleep 5 

# training script
python3 src/train.py --input-dir "$INPUT_DIR_LOCAL" --output-dir "$OUTPUT_DIR_LOCAL"

# upload artefacts
supervisely teamfiles upload -id $TEAM_ID --src "$OUTPUT_DIR_LOCAL" --dst "$DST_DIR"
# set final Team files dir in Workspace tasks
supervisely task set-output-dir -id $TASK_ID --team-id $TEAM_ID  --dir "$DST_DIR"

# cleaning the space on agent
echo "Deleting "$SLY_APP_DATA_DIR" contents"
rm -rf "$SLY_APP_DATA_DIR/*"
```

</details>

**Step 6.** Configure your training script

Modify `src/train.py` with your own training loop:

<details>

<summary>Python training script example</summary>

```python

import argparse
import os
import time
import random
import torch
from torch.utils.tensorboard import SummaryWriter
import supervisely as sly


def train(input_dir: str, output_dir: str) -> None:
    """
    train model on input_dir, log metrics to tensorboard, save artefacts to output_dir
    """

    print(f"Input directory with training data: {input_dir}")
    # hint: transform data in supervisely format to the format your training script understands

    print(f"Training started, artefacts will be saved to {output_dir} ...")
    os.makedirs(output_dir, exist_ok=True)

    # Start a TensorBoard writer
    writer = SummaryWriter(output_dir)

    iters = 150
    steepness = random.uniform(0.1, 10.0)
    progress = sly.Progress(message="Training...", total_cnt=iters)
    for step in range(iters):
        time.sleep(0.1)  # imitates training process
        loss = 1.0 / (steepness * (step + 1))

        print(f"Step [{step}]: loss={loss:.4f}")
        writer.add_scalar("Loss", loss, step)  # Log smth to TensorBoard

        # save fake checkpoint every 30 iterations
        if step != 0 and step % 30 == 0:
            torch.save(
                {"iter": step, "model_state_dict": {"a": "b"}, "loss": loss},
                os.path.join(output_dir, f"{step:05d}.pt"),
            )

        progress.iter_done_report()  # log to view progress bar in Supervisely

    # Close the TensorBoard writer
    writer.close()
    print("Training finished")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Training tensorboard template")
    parser.add_argument("--input-dir", "-i", required=True, help="Input dir with training data")
    parser.add_argument("--output-dir", "-o", required=True, help="Dir for training artefacts")

    args = parser.parse_args()
    train(args.input_dir, args.output_dir)

```

</details>

**Step 7.** Start debugging.

```bash
export ENV="development" && ./run.sh
```

**Step 8.** Watch tensorboard while training.

Tensorboard is available in browser using address [http://localhost:8000/](http://localhost:8000/)

**Step 9.** Open output artefacts in Team files.

You can always examine your logs by simply using [Tensorboard metrics viewer app](https://ecosystem.supervise.ly/apps/tensorboard-logs-viewer). To do that, just follow this steps (learn more in app readme):

1. Choose your folder in Team Files containing tensorboard logs or the file itself
2. Right-click on the object and click on three-dot menu. Then, choose `Run App -> Tensorboard`. Click `Run`.
3. After running, the tensorboard server will be available with `Open` button in workspace. Click on it.
4. That's it! Now you can view your tensorboard logs.

**Step 10.** Release your private app

Just run the following command in the root directory of you app. Learn more in [corresponding tutorial](../../basics/add-private-app.md).

```bash
# use supervisely cli
supervisely release
```

**Step 11.** Run app on your Supervisely instance

Choose your project and click on three-dot menu. Then, choose `Run App -> Training tensorboard template` and, if you need, specify selected `Advanced Settings`. Click `Run`.

**Step 12.** Open Tensorboard while training

Wait until your project will be downloaded and your tensorboard logging server will start. You can open it in `Workspace Tasks` interface with clicking `Open` button.

{% hint style="info" %}
In case of sudden crash, you can view saved data in `'Team Files' -> Supervisely agents -> <chosen node> ('Main node' by default) -> 'app-data' -> 'training-tensorboard-template'`
{% endhint %}

**Step 13.** Open link with output artefacts in Team files

After successful task ending, Tensorboard server stops and there will be a direct link to a Team files folder. You can always examine your logs by simply using [Tensorboard metrics viewer app](https://ecosystem.supervise.ly/apps/tensorboard-logs-viewer).
