# Deploy Model

## Introduction

**`DeployModel`** widget in Supervisely is a comprehensive user interface element that allows users to deploy or connect to neural network models in three different modes:

- **Connect**: Connect to an already deployed model session
- **Pretrained**: Deploy a pretrained model from the Supervisely ecosystem
- **Custom**: Deploy a custom model from your training experiments

This widget streamlines the model deployment workflow and provides a unified interface for managing model sessions in Supervisely apps.
![deploy-model](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1766070037491.png)

## Function signature

```python
DeployModel(
    api=None,
    team_id=None,
    modes=None,
    widget_id=None,
)
```

## Parameters

| Parameters  |                        Type                        |                    Description                    |
| :---------: | :------------------------------------------------: | :-----------------------------------------------: |
|    `api`    |                       `Api`                        | Supervisely API instance for server communication |
|  `team_id`  |                       `int`                        |           Team ID for model operations            |
|   `modes`   | `List[Literal["connect", "pretrained", "custom"]]` | List of deployment modes to enable in the widget  |
| `widget_id` |                       `str`                        |                 ID of the widget                  |

### api

Supervisely API instance for communicating with the Supervisely server. If not provided, will use the default API instance.

**type:** `Api`

**default value:** `None` (will create new Api instance)

### team_id

Team ID for model deployment operations. If not provided, will use the team ID from environment variables.

**type:** `int`

**default value:** `None` (will use `sly.env.team_id()`)

### modes

List of deployment modes to enable in the widget. Can include "connect", "pretrained", and/or "custom". If not provided, all three modes will be available.

**type:** `List[Literal["connect", "pretrained", "custom"]]`

**default value:** `None` (all modes enabled)

### widget_id

ID of the widget for internal use.

**type:** `str`

**default value:** `None`

## Methods and attributes

### Public Methods

|          Method           |     Returns      | Description                                                                                                                          |
| :-----------------------: | :--------------: | ------------------------------------------------------------------------------------------------------------------------------------ |
|        `deploy()`         |    `ModelAPI`    | Deploy or connect to a model based on current mode and selection. Returns ModelAPI instance for interacting with the deployed model. |
|         `stop()`          |      `None`      | Stop the currently deployed model session. Shuts down the model and cleans up resources.                                             |
|      `disconnect()`       |      `None`      | Disconnect from the model without stopping it. The model continues running but this widget releases the connection.                  |
| `get_deploy_parameters()` | `Dict[str, Any]` | Get current deployment parameters as a dictionary including mode, agent_id, and mode-specific parameters.                            |

### Status and UI Control Methods

|            Method            |                Parameters                 | Description                                                                                                                          |
| :--------------------------: | :---------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------ |
|     `set_model_status()`     | `status: Literal[...]`, `extra_text: str` | Set the deployment status message. Status can be: "deployed", "stopped", "deploying", "connecting", "connected", "error", or "hide". |
|     `set_session_info()`     |             `task_info: Dict`             | Display session information for deployed model including session ID, app name, and timestamp.                                        |
|      `set_model_info()`      |             `session_id: int`             | Set model information widget with session details.                                                                                   |
|     `reset_model_info()`     |                   None                    | Reset model information widget to initial state.                                                                                     |
| `set_model_message_by_tab()` |              `tab_name: str`              | Update model info message based on active tab.                                                                                       |
|      `disable_modes()`       |                   None                    | Disable all mode selection and agent selector during deployment.                                                                     |
|       `enable_modes()`       |                   None                    | Enable mode selection and agent selector after deployment completes.                                                                 |
|        `show_stop()`         |                   None                    | Show stop/disconnect buttons and hide deploy/connect buttons.                                                                        |

### Utility Methods

|       Method       |      Returns       | Description                                                                     |
| :----------------: | :----------------: | ------------------------------------------------------------------------------- |
|  `get_modules()`   | `List[ModuleInfo]` | Get list of available serving modules from ecosystem with framework categories. |
| `get_frameworks()` |    `List[str]`     | Get list of available framework names from serving modules.                     |

### Properties

|  Property   |          Type           | Description                                                                             |
| :---------: | :---------------------: | --------------------------------------------------------------------------------------- |
| `model_api` |   `ModelAPI or None`    | Current ModelAPI instance for the deployed/connected model. None if no model is active. |
|   `modes`   | `Dict[str, DeployMode]` | Dictionary of available deployment modes (Connect, Pretrained, Custom).                 |
|  `layout`   |        `Widget`         | The main layout widget (RadioTabs if multiple modes, otherwise single mode container).  |

## Deployment Modes

### Connect Mode

Connect to an already deployed model session. The widget displays a table of all active model sessions in your team.

**Features:**

- Lists all deployed models with session ID, app name, framework, and model name
- Refresh button to update the list of available sessions
- Select and connect to any active session

### Pretrained Mode

Deploy a pretrained model from the Supervisely ecosystem.

![Pretrained Mode](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1766071088726.png)

**Features:**

- Browse pretrained models by framework
- View model details including parameters and performance metrics
- Select agent for deployment
- Automatic model deployment

### Custom Mode

Deploy a custom model from your training experiments.

![Custom Mode](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1766070070962.png)

**Features:**

- Browse your training experiments
- Select specific checkpoints
- View experiment details and metrics
- Deploy custom trained models

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/misc/deploy_model/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/misc/deploy_model/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Button, Text, DeployModel, Flexbox
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
team_id = sly.env.team_id()
```

### Initialize DeployModel widget

```python
# Initialize DeployModel widget with all modes
deploy_model = DeployModel(
    api=api,
    team_id=team_id,
    modes=["connect", "pretrained", "custom"]
)
```

### Create additional UI elements

```python
# Create info text to display deployment parameters
info_text = Text("", status="text")
info_text.hide()

# Create buttons to interact with the widget
get_params_button = Button("Get Deploy Parameters", icon="zmdi zmdi-info")
load_state_button = Button("Load State (Connect to YOLOv8)", icon="zmdi zmdi-download")

buttons_container = Container(
    [get_params_button, load_state_button],
    direction="horizontal",
    gap=10
)
```

### Create app layout

```python
# Create main card
card = Card(
    title="Deploy Model",
    description="Deploy or connect to a model using different modes",
    content=Container([deploy_model, buttons_container, info_text], gap=20),
)

layout = Container(widgets=[card], direction="vertical")
app = sly.Application(layout=layout)
```

### Add button handlers

```python
@get_params_button.click
def show_deploy_params():
    """Show current deployment parameters"""
    try:
        params = deploy_model.get_deploy_parameters()
        info_text.set(f"Deploy Parameters: {params}", status="success")
        info_text.show()
    except Exception as e:
        info_text.set(f"Error getting parameters: {str(e)}", status="error")
        info_text.show()


@load_state_button.click
def load_example_state():
    """Load example state - select pretrained model"""
    try:
        # Example: Load state to select a pretrained YOLOv8n model
        example_state = {
            "mode": "pretrained",
            "agent_id": None,
            "framework": "yolov8",
            "model_name": "yolov8n-seg"
        }
        deploy_model.load_from_json(example_state)
        info_text.set("Loaded example state: YOLOv8n segmentation model", status="success")
        info_text.show()
    except Exception as e:
        info_text.set(f"Error loading state: {str(e)}", status="error")
        info_text.show()
```

## Usage Examples

### Deploy a pretrained model

```python

# Programmatically deploy
model_api = deploy_model.deploy()
print(f"Model deployed with session ID: {model_api.task_id}")
```

### Connect to existing session

```python
# User selects session from table and clicks Connect
# Or programmatically:
deploy_model.load_from_json({
    "mode": "connect",
    "session_id": 12345
})
model_api = deploy_model.deploy()
```

### Deploy custom model

```python
# Load specific experiment
deploy_model.load_from_json({
    "mode": "custom",
    "agent_id": 10,
    "train_task_id": 67890
})
```

### Get deployment parameters

```python
params = deploy_model.get_deploy_parameters()
```

### Stop deployed model

```python
# User clicks Stop button or programmatically:
deploy_model.stop()
```

### Disconnect without stopping

```python
# Disconnect from model but leave it running
deploy_model.disconnect()
```
