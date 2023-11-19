# Overview

Although model training applications can be complex, they typically adhere to a simple structure consisting of two main components: the training code and the user interface. The UI is a collection of Supervisely widgets that allow the user to pick data, adjust training parameters, and watch the training process. More information about widgets can be found [in this section](/app-development/widgets/README.md). 

The implementation of the model training process varies significantly depending on the specific model architecture. Supervisely's applications code provides you with the freedom to delve into different implementation approaches for model training: [HRDA](https://github.com/supervisely-ecosystem/hrda/blob/master/src/main.py), [YOLOv8](https://github.com/supervisely-ecosystem/yolov8/tree/master/train), [MMDetection](https://github.com/supervisely-ecosystem/train-mmdetection-v3/tree/master/src).

Let's now delve into some crucial components of applications using the following code and explore practical use cases of SDK methods and functions that will empower you to craft robust and functional applications.


```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app import widgets

# PART 1: BASE UI
models = widgets.RadioTable(columns=["Name"], rows=[["Model_1", "Model_2"]])
start_btn = widgets.Button(text="Start app")

layout = widgets.Container(
    widgets=[
        models,
        start_btn
    ]
)

# PART 2: SETUP
app = sly.Application(layout=layout)
api = sly.Api.from_env()

if not sly.is_production():
    # set more appropriate checkpoints_dir for local debug
    os.environ["SLY_APP_DATA_DIR"] = "./artifacts"
    load_dotenv("debug.env")

TEAM_ID = sly.env.team_id()
TASK_ID = sly.env.task_id()

# Using this checkpoints_dir is important in order to avoid data loss
checkpoints_dir = sly.app.get_synced_data_dir()
my_train_app_name = "MyTrainApp"


# PART 3: TRAIN FUNCTIONS
def get_model_instance_by_name():
    name = models.get_selected_row()
    # Initialize model with default params by name
    ...
    return model


def train_process(model, data, app: sly.Application):
    # pre-train preparing
    ...
    # start train cycle
    for epoch in epochs:
        for batch in data:
            # train on batch
            ...
            # check if app receive stop signal
            if app.is_stopped():
                raise app.StopException
        # Save checkpoint
        model_path = os.path.join(checkpoints_dir, f"checkpoint_{epoch}.pth")
        model.save(model.state_dict(), model_path)
    last_checkpoint = os.path.join(checkpoints_dir, f"checkpoint_last.pth")
    model.save(model.state_dict(), last_checkpoint)


def upload_artifacts():
    out_path = api.file.upload_directory(
        team_id=TEAM_ID,
        local_dir=checkpoint_dir,
        remote_dir=f"/my_train_app_name/{TASK_ID}"
    )
    return out_path


# PART 4: APP LOGIC

# start train on button click
@start_btn.click
def start_train():
    model = get_model_instance_by_name()
    with app.handle_stop(graceful=True):
        train_process(model, data, app)

    # The following code will only be executed if:
    # 1. training loop successfuly finished (like normal python code - execution line by line)
    # 2. if STOP event recieved during training and `graceful=True`
    if sly.is_production():
        out_path = upload_artifacts()
        
        # allows you to display the path in the output
        last_checkpoint = os.path.join(checkpoints_dir, f"checkpoint_last.pth")
        file_info = g.api.file.get_info_by_path(TEAM_ID, last_checkpoint)
        api.task.set_output_directory(g.api.task_id, file_info.id, out_path)
    
        # clean agent memory
        sly.fs.remove_dir(checkpoints_dir)
    
    # stop application
    app.stop()
```

# Use cases and best practice

## Base UI

To incorporate widgets into the application, wrap them within a `supervisely.app.widgets.Container` object and pass it as the layout parameter to the `sly.Application` constructor.

## Checkpoints 

### Description

Throughout the training process, multiple checkpoints of the model are commonly saved: the best-performing checkpoint, the most recent checkpoint or several of the most recent checkpoints. Upon successful training completion, all relevant files are transferred to Team Files. In the event of training interruptions due to unforeseen circumstances, such as GPU memory limitations or server issues, we desire the capability to resume training from previously generated checkpoints.

### Tools

While developers are responsible for properly storing training artifacts in Team Files, interim files can be temporarily stored in the `sly.app.get_synced_data_dir()` directory. This path within the Docker container is linked to a corresponding host folder within the Agent Docker container. Consequently, all checkpoints remain accessible throughout the training process at `Team Files/Supervisely Agents/{agent_name}/app_data`.

![Saved checkpoints for running train applications](https://github.com/supervisely/developer-portal/assets/87002239/804451a4-d79e-4f70-b3f6-7fd3540dc7f4)

{% hint style="info" %}
Files saved to this directory will be automatically purged upon successful program completion. Only in the event of training errors, the agent will refrain from deleting these files. These files can be subsequently removed through the Agent panel in Team Clusters.
{% endhint %}

The value of `sly.app.get_synced_data_dir()` is determined by the `SLY_APP_DATA_DIR` environment variable. In *PART 2* of the code, we utilize distinct `SLY_APP_DATA_DIR` paths for production and local debugging scenarios.

### Example

The `train_process()` function utilizes `sly.app.get_synced_data_dir()` to save checkpoints for each epoch, as well as the final checkpoint. In *PART 4*, upon the completion or graceful cancellation (using graceful=True) of the training process, the script initiates the uploading of artifacts to the Team Files. The `api.task.set_output_directory()` function is employed to display the training path in the output after training concludes. 

![Correct output after app completion](https://github.com/supervisely/developer-portal/assets/87002239/54763281-c3b3-400b-9941-cc981eeffec3)

Additionally, it is advisable to clear the checkpoints directory with `sly.fs.remove_dir(checkpoints_dir)` before shutting down to avoid data leakage, even though the agent is likely to automatically clean up. 

## Application stop

### Description 

Halting a Supervisely application can be accomplished in three distinct ways: by terminating the running Docker container via an agent request, known as a force stop, by terminating the internal process within the container through a request to the `sly.Application` object, or by allowing the app to complete its tasks and then terminate (`app.stop()` command in *PART 4*). The following guidelines must be followed for the stopping process to be error-free.

1. Non-main threads should either be explicitly terminated on stop or configured to run in daemon mode. 
2. The execution of training and validation cycles must be interruptible at batch boundaries.
3. The `app.stop()` must **always** be called at the end.

Disregarding these guidelines may lead to scenarios where the application fails to terminate upon request or generates erroneous logs, despite the application's logic being sound.

![Non zero status](https://github.com/supervisely/developer-portal/assets/87002239/fb9e2c39-71d3-419f-ac0b-4150b798c3c2)

### Tools

To guarantee a smooth application stoppage, consider using the following class methods from `app = sly.Application()` object:

- `app.is_stopped()` method returns True if the application has received a signal to stop;
- `app.call_before_shutdown(func)` method adds a callback that will be called before the application terminates;
- `app.StopException` error is used to exit from a training function;
- `app.handle_stop(graceful=True)` context manager allows you to run functions suppressing the app.StopException error. If `graceful=True`, then the part of the code that was limited by the context manager will be skipped on error, the remaining part will be executed until the application is stopped using the `app.stop()` method. Otherwise, the application will be stopped immediately after the function exits.

### Example

The training function is initiated within the `app.handle_stop(graceful=True)` context manager. Therefore, if a *STOP* task is received, the `train_process()` function will be interrupted after batch processing. However, the shutdown process will be on hold until `app.stop()` is called. This setup ensures that our application successfully uploads all saved checkpoints to Team Files before shutting down.