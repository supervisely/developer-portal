# Overview

Although model training applications can be complex, they typically adhere to a simple structure consisting of two main components: the training code and the user interface. The UI is a collection of Supervisely widgets that allow the user to pick data, adjust training parameters, and watch the training process. More information about widgets can be found [in this section](/app-development/widgets/README.md). 

To incorporate widgets into the application, wrap them within a `supervisely.app.widgets.Container` object and pass it as the layout parameter to the `sly.Application` constructor.

```python
import supervisely as sly
from supervisely.app import widgets

# SOME CODE

layout = widgets.Container(
    widgets=[
        # CREATED WIDGET FOR THE APPLICATION
    ]
)
app = sly.Application(layout=layout)
```

The implementation of the model training process varies significantly depending on the specific model architecture. Supervisely's applications code provides you with the freedom to delve into different implementation approaches for model training: [HRDA](https://github.com/supervisely-ecosystem/hrda/blob/master/src/main.py), [YOLOv8](https://github.com/supervisely-ecosystem/yolov8/tree/master/train), [MMDetection](https://github.com/supervisely-ecosystem/train-mmdetection-v3/tree/master/src).

Let's now delve into some crucial components of applications and explore practical use cases of SDK methods and functions that will empower you to craft robust and functional applications.

# Use cases and best practice

## Stoppable application

### Description 

Halting a Supervisely application can be accomplished in two distinct ways: either by terminating the running Docker container via an agent request, known as a force stop, or by terminating the internal process within the container through a request to the `sly.Application` object. The following guidelines must be followed for the stopping process to be error-free.

1. Non-main threads should either be explicitly terminated on stop or configured to run in daemon mode. 
2. The execution of training and validation cycles must be interruptible at batch boundaries.

Disregarding these guidelines may lead to scenarios where the application fails to terminate upon request or generates erroneous logs, despite the application's logic being sound.

![Non zero status](https://github.com/supervisely/developer-portal/assets/87002239/fb9e2c39-71d3-419f-ac0b-4150b798c3c2)

### Tools

To guarantee a smooth application stoppage, consider using the following class methods from `app = sly.Application()` object:

- `app.app_is_stopped()` method returns True if the application has received a signal to stop;
- `app.call_before_shutdown(func)` method adds a callback that will be called before the application terminates;
- `app.StopApp` error is used to exit from a training function;
- `app.run_with_stop_app_suppression(graceful_stop=True)` context manager allows you to run functions suppressing the app.StopApp error. If `graceful_stop=True`, then the part of the code that was limited by the context manager will be skipped on error, the remaining part will be executed until the application is stopped using the `app.stop()` method. Otherwise, the application will be stopped immediately after the function exits.

### Example

Here's an abstract example with explanatory comments. If you want to see the actual functional code, you can do so [here](https://github.com/supervisely-ecosystem/yolov8/blob/f181fcc78a7cdaacf2f7dd088df30f21a25aed1f/train/src/main.py#L1192).

```python
def train_process(model, data, app: sly.Application):
    # pre-train preparing
    ...
    # start train cycle
    for epoch in epochs:
        for batch in data:
            # train on batch
            ...
            # check if app receive stop signal
            if app.app_is_stopped():
                raise app.StopApp


# preparing
app = sly.Application()
...
# start train
with app.run_with_stop_app_suppression(graceful_stop=True):
    train_process(model, data, app)

# the following code will only be executed if `graceful_stop=True`
# or no `STOP` signal recieved
save_artifacts()
app.stop()
```

## Checkpoints 

### Description

Throughout the training process, multiple checkpoints of the model are commonly saved: the best-performing checkpoint, the most recent checkpoint or several of the most recent checkpoints. Upon successful training completion, all relevant files are transferred to Team Files. In the event of training interruptions due to unforeseen circumstances, such as GPU memory limitations or server issues, we desire the capability to resume training from previously generated checkpoints.

### Tools

While developers are responsible for properly storing training artifacts in Team Files, interim files can be temporarily stored in the `sly.app.get_synced_data_dir()` directory. This path within the Docker container is linked to a corresponding host folder within the Agent Docker container. Consequently, all checkpoints remain accessible throughout the training process at `Team Files/Supervisely Agents/{agent_name}/app_data`. 

{% hint style="warning" %}
Files saved to this directory will be automatically purged upon successful program completion. Only in the event of training errors, the agent will refrain from deleting these files. These files can be subsequently removed through the Agent panel in Team Clusters.
{% endhint %}