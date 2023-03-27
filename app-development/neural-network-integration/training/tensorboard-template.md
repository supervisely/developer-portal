---
description: >-
  Step-by-step tutorial explains how to use custom training script and log results in Tensorboard
---

# Object detection

## Introduction

This tutorial will teach you how to integrate your custom training script into Supervisely ecosystem by running provided shell script and logging training output artefacts with Tensorboard.

Full code of training tensorboard template can be found [here](https://github.com/supervisely-ecosystem/training-tensorboard-template)

<!-- ![training-tensorboard\_template]() -->

***

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/training-tensorboard-template) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/training-tensorboard-template
cd training-tensorboard-template
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Start debugging with command `ENV="development" && ./run.sh`. 

***

## Integrate your script

Follow this simple steps to successfuly run your custom training script:

**Step 1.** Configure your training script

Modify `src/train.py` with your own training loop:

<details>

<summary>Example</summary>

```python

def train(input_dir: str, output_dir: str) -> None:
    """
    Gets data from input_dir, trains a model, generates artefacts as output data,
    and logs the training process. Generated artefacts backed up using synced_dir.
    """
    print(f"Opening data from {input_dir}...")

    files = os.listdir(input_dir)
    print('Number of objects in input directory:', len(files))

    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    # Start a TensorBoard writer
    writer = SummaryWriter(output_dir)

    print("Training model...")
    print(f"Generating output artifacts in {output_dir}...")

    # initializing loop with dummy data...
    n_iter = 30
    progress = sly.Progress(message='Training...', total_cnt=n_iter) # progress bar in workspace tasks output

    for step in range(n_iter):

        time.sleep(5) # imitates training process
        loss = 1.0 / (step + 1)

        # Log the data to TensorBoard
        writer.add_scalar('Loss', loss, step)
        print(f"Step [{step}]: loss={loss:.4f}")


        file_path = os.path.join(output_dir, f'step_{str(step).zfill(len(str(n_iter)))}.txt')
        # create artefacts
        with open(file_path, 'w') as f:
            f.write(f"Step [{step}]: loss={loss:.4f}")

        progress.iter_done_report() # update progress bar

    # Close the TensorBoard writer
    writer.close()

    print(f"Artefacts generated in {output_dir}!")
```

</details>


**Step 2.** Choose project with training data

Choose your project and click on three-dot menu. Then, choose `Run App -> Training tensorboard template` and, if you need, specify selected `Advanced Settings`. Click `Run`.

<!-- ![training-tensorboard\_template]() -->

**Step 3.** Open Tensorboard while training

Wait until your project will be downloaded and your tensorboard logging server will start. You can open it in `Workspace Tasks` interface with clicking `Open` button.

{% hint style="info" %}
In case of sudden crash, you can view saved data in `'Team Files' -> Supervisely agents -> <chosen node> ('Main node' by default) -> 'app-data' -> 'training-tensorboard-template'`
{% endhint %}

<!-- ![training-tensorboard\_template]() -->

**Step 4.** Open link with output artefacts in Team files

After successful task ending, Tensorboard server stops and there will be a direct link to a teamfiles folder. You can always examine your logs by simply using Tensorboard logs viewer app. To do that, just follow this steps:

1. Choose your folder in Team Files containing tensorboard logs or the file itself
2. Right-click on the object and click on three-dot menu. Then, choose `Run App -> Tensorboard Logs Viewer`. Click `Run`.
3. After running, the tensorboard server will be available with `Open` button in workspace. Click on it.
4. That's it! Now you can view your tensorboard logs.

