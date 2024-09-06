# SelectCudaDevice

## Introduction

The **`SelectCudaDevice`** widget in Supervisely is a user interface element that allows users to select cuda devices available on their machines. Widget detects devices automatically, and displayes them in a selector alongside their RAM usage values. The widget also has a refresh button which allows to get up-to-date information on available devices and their RAM usage values.
The `SelectCudaDevice` widget has event handler that is triggered when the user selects an option from the dropdown menu.

## Function signature

```python
SelectCudaDevice(
    get_list_on_init = True,
    sort_by_free_ram = False,
    include_cpu_option = False,
    widget_id = None,
)
```

![default](https://github.com/user-attachments/assets/4a971741-3cc8-4844-b116-6abd40c751e2)

## Parameters

|     Parameters      |        Type         |                                         Description                                          |
| :-----------------: | :-----------------: | :------------------------------------------------------------------------------------------: |
| `get_list_on_init`  | `Optional[bool]`    | Whether to retrieve and display the list of CUDA devices upon initialization. Default is `True`. |
| `sort_by_free_ram`  | `Optional[bool]`    | Whether to sort the CUDA devices by their available free RAM. Default is `False`.              |
| `include_cpu_option`| `Optional[bool]`    | Whether to include CPU as an option in the device list. Default is `False`.                   |
| `widget_id`         | `str`               | An optional ID for the widget.                                                                |

### get_list_on_init

Determine if you want to get the device list on widget initialization.

**type:** `Optional[bool]`

**default value:** `True`

Initialize the empty widget, and then manually refresh it to find the devices:

```python
widget = SelectCudaDevice(get_list_on_init=False)

# some code here

widget.refresh()
```

### sort_by_free_ram

If `True`, the device list will automatically be sorted by its free RAM value. Default is `False`.

**type:** `Optional[bool]`

**default value:** `False`

Initialize the widget sorting it by free RAM:

```python
widget = SelectCudaDevice(sort_by_free_ram=True)
```

![sort_by_free_ram](https://github.com/user-attachments/assets/13f47e61-e924-4386-903a-62ae0ecb96fc)

### include_cpu_option

If `True`, selector will include CPU as a device option. Default is `False`.

**type:** `Optional[bool]`

**default value:** `False`

Initialize the widget with CPU as a device option:

```python
widget = SelectCudaDevice(include_cpu_option=True)
```

### widget_id

An optional ID for the widget.

**type:** `str`

**default value:** `None`

Initialize the widget with an ID:

```python
widget = SelectCudaDevice(widget_id="select_cuda_device")
```

## Methods and attributes

|                Methods and Attributes                |                      Description                       |
| :--------------------------------------------------: | :----------------------------------------------------: |
|                  `get_device()`                     |              Get currently selected device.             |
|            `set_device(value: str)`                 |  Set the device to be displayed in the widget (overrides current).|
|                    `refresh()`                      |   Get up-to-date information about available devices and set them as options in the selector.    |


## Mini app example

You can find this example in our GitHub repository:

[](link)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from typing import List
from supervisely.app.widgets import Card, SelectCudaDevice, Container, Text, Button, Flexbox
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize the `SelectCudaDevice` widget

Let's initialize the `SelectCudaDevice` widget with no devices:

```python
cuda_device = SelectCudaDevice(
    get_list_on_init=False,
    include_cpu_option=True,
)
```

Then, we can manually refresh the widget, retrieving the devices information when we need to.

```python
cuda_device.refresh()
```

### Create an event handler for the `SelectCudaDevice` widget

Now, let's create a event handler for the SelectCudaDevice widget, when the value of the widget is changed.

```python
value_changed_info = Text(status="info")
value_changed_info.hide()


@cuda_device.value_changed
def new_device_selected(device: str):
    text = "Value changed: {}".format(device)
    value_changed_info.text = text
    value_changed_info.show()
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Cuda Device",
    content=Container(widgets=[value_changed_info, cuda_device]),
)

layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

![mini_app](https://github.com/user-attachments/assets/13f47e61-e924-4386-903a-62ae0ecb96fc)
