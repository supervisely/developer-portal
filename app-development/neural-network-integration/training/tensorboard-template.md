---
description: >-
  Step-by-step tutorial explains how to use custom training script and log results in Tensorboard
---

# Training tensorboard template

## Introduction

This tutorial will teach you how to integrate your custom training script into Supervisely ecosystem by running provided shell script and logging training output artefacts with Tensorboard.

Full code of training tensorboard template can be found [here](https://github.com/supervisely-ecosystem/training-tensorboard-template)

<!-- ![training-tensorboard\_template]() -->

***

## Run template locally

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

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
code -r .
```

**Step 4.** Modify `local.env` file with your project ID. [Learn more here.](https://developer.supervise.ly/getting-started/environment-variables)

```python
PROJECT_ID=12208 # ⬅️ change it
TEAM_ID=449 # ⬅️ change it
SLY_APP_DATA_DIR="/home/<user>/test-dir" # ⬅️ change it
```

{% hint style="info" %}

Note: variable SLY_APP_DATA_DIR is for synced data directory which mirrors artefacts data on Team files. It will be used for backup of of training artefacts in case of sudden crash of the training script.

{% endhint %}

**Step 5.** Configure your training script

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


**Step 6.** Start debugging.

```bash
export ENV="development" && ./run.sh
```

**Step 7.** Watch tensorboard while training.

Tensorboard is available in browser using address (http://localhost:8000/)

**Step 8.** Open output artefacts in Team files.

You can always examine your logs by simply using Tensorboard logs viewer app. To do that, just follow this steps:

1. Choose your folder in Team Files containing tensorboard logs or the file itself
2. Right-click on the object and click on three-dot menu. Then, choose `Run App -> Tensorboard Logs Viewer`. Click `Run`.
3. After running, the tensorboard server will be available with `Open` button in workspace. Click on it.
4. That's it! Now you can view your tensorboard logs.

## Run template as private app

Follow this steps to successfully run your code as app in Supervisely ecosystem:

**Step 1.** Clone [repository](https://github.com/supervisely-ecosystem/training-tensorboard-template) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/training-tensorboard-template
cd training-tensorboard-template
./create_venv.sh
```

**Step 2.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 3.** Configure your training script

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

**Step 4.** Release private app with command-line-interface (CLI)

Commit your changes to your personal branch and publish it in VSCode. Then, execute the following command in the terminal to release an app. By default this command will pack and release files in the current folder.

```bash
# use supervisely cli
supervisely release
```

You will be asked for release description. After that you will see a summary message and confirmation request. Then if there is no errors you will see "App release successfully!" message.

![release from other branch](https://user-images.githubusercontent.com/61844772/225957782-2c6557e4-93ed-4ab2-a40e-4268b7110976.png)

You can provide release version and release description by providing `--release-version` and `--release-description` options to the CLI

Your app will appear in section `🔒 private apps` in Ecosystem.

![private apps](https://user-images.githubusercontent.com/12828725/205959921-7d631cb5-c1fc-4b0c-99d5-f2260c96708d.png)

Thus you can quickly do releases of your app. All releases will be available on the application page. Just select the release in modal window in advanced section before running the app. The latest release is selected by default.

![app versions](https://user-images.githubusercontent.com/12828725/205960656-615803f0-c081-4086-b7ba-45f4bbc60cb6.png)



**Step 5.** Choose project with training data

Choose your project and click on three-dot menu. Then, choose `Run App -> Training tensorboard template` and, if you need, specify selected `Advanced Settings`. Click `Run`.

<!-- ![training-tensorboard\_template]() -->

**Step 6.** Open Tensorboard while training

Wait until your project will be downloaded and your tensorboard logging server will start. You can open it in `Workspace Tasks` interface with clicking `Open` button.

{% hint style="info" %}
In case of sudden crash, you can view saved data in `'Team Files' -> Supervisely agents -> <chosen node> ('Main node' by default) -> 'app-data' -> 'training-tensorboard-template'`
{% endhint %}

<!-- ![training-tensorboard\_template]() -->

**Step 7.** Open link with output artefacts in Team files

After successful task ending, Tensorboard server stops and there will be a direct link to a Team files folder. You can always examine your logs by simply using Tensorboard logs viewer app. To do that, just follow this steps:

1. Choose your folder in Team Files containing tensorboard logs or the file itself
2. Right-click on the object and click on three-dot menu. Then, choose `Run App -> Tensorboard Logs Viewer`. Click `Run`.
3. After running, the tensorboard server will be available with `Open` button in workspace. Click on it.
4. That's it! Now you can view your tensorboard logs.