---
description: >-
  Step-by-step tutorial explains how to integrate custom object detection neural
  network into Supervisely platform
---

# Object detection

## Introduction

This tutorial will teach you how to integrate your custom object detection model into Supervisely by using ObjectDetectionTrainDashboad class.

Full code of object detecting sample app can be found [here](https://github.com/supervisely-ecosystem/object-detection-training-template)

![training\_dashboard](../../../.gitbook/assets/training\_dashboard.png)



## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/object-detection-training-template) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/object-detection-training-template
cd object-detection-training-template
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Start debugging `src/main.py`



## Integrate your model

The integration of your own NN with ObjectDetectionTrainDashboad is really simple:

**Step 1.** Define pytorch dataset

<details>

<summary>Example</summary>

```python
import json
import random 
from typing import Literal, List, Dict, Optional, Any, Tuple, Union

import numpy as np
import imgaug.augmenters as iaa
import supervisely as sly
from torch.utils.data import Dataset

###########################
### 1. Redefine your dataset and methods or use them as is
###########################
class CustomDataset(Dataset):
    def __init__(self, 
                 items: sly.ImageInfo, 
                 project_meta: sly.ProjectMeta, 
                 classes: List[str], 
                 input_size: Union[List, Tuple], 
                 transforms: Optional[iaa.Sequential] = None):
        self.items = items
        self.project_meta = project_meta
        self.classes = classes
        self.input_size = input_size
        self.transforms = transforms
    
    def __getitem__(self, index: int):
        image = sly.image.read(self.items[index].img_path)
        with open(self.items[index].ann_path, 'r') as f:
            ann = sly.Annotation.from_json(json.loads(f.read()), self.project_meta)

        if self.transforms:
            res_meta, image, ann = sly.imgaug_utils.apply(self.transforms, self.project_meta, image, ann)

        meta, image, ann = sly.imgaug_utils.apply(
            iaa.Sequential([
                iaa.Resize({"height": self.input_size[1], "width": self.input_size[0]})
            ]), 
            self.project_meta, 
            image, 
            ann
        )

        # our simple model can work only with 1 bbox per image
        # you need to change code if your model more complicated
        label = random.choice(ann.labels)
        class_id = self.classes.index(label.obj_class.name)
        bbox = label.geometry.to_bbox()
        bbox = np.array([bbox.left, bbox.top, bbox.right, bbox.bottom])
    
        image = np.rollaxis(image, 2)
        return image, class_id, bbox

    def __len__(self):
        return len(self.items)
```

</details>

**Step 2.** Define pytorch object detection model

<details>

<summary>Example</summary>

```python
from typing import Literal, List, Dict, Optional, Any, Tuple, Union

import torch.nn as nn
import torch.nn.functional as F
from torchvision import models

###########################
### 2. Define neural network model
###########################
class CustomModel(nn.Module):
    def __init__(self):
        super(CustomModel, self).__init__()
        resnet = models.resnet34()
        layers = list(resnet.children())[:8]
        self.features1 = nn.Sequential(*layers[:6])
        self.features2 = nn.Sequential(*layers[6:])
        self.classifier = nn.Sequential(nn.BatchNorm1d(512), nn.Linear(512, 4))
        self.bb = nn.Sequential(nn.BatchNorm1d(512), nn.Linear(512, 4))
        
    def forward(self, x):
        x = self.features1(x)
        x = self.features2(x)
        x = F.relu(x)
        x = nn.AdaptiveAvgPool2d((1,1))(x)
        x = x.view(x.shape[0], -1)
        return self.classifier(x), self.bb(x)
```

</details>

**Step 3.** Define subclass **`ObjectDetectionTrainDashboad`** and implement **`train`** method

<details>

<summary>Example</summary>

```python
import os
from typing import Literal, List, Dict, Optional, Any, Tuple, Union

import torch
import torch.nn.functional as F
import supervisely as sly
from torch.utils.data import DataLoader

import src.sly_globals as g
from src.dashboard import ObjectDetectionTrainDashboad

###########################
### 3. Define dashboard
###########################
class CustomTrainDashboard(ObjectDetectionTrainDashboad):
    def train(self):
        # getting selected classes from UI
        classes = self._classes_table.get_selected_classes()
        # getting configuration of splits from UI
        train_set, val_set = self.get_splits()
        # getting hyperparameters from UI
        hparams = self.get_hyperparameters()
        # initialized with selected hyperparameters pytorch optimizer will be returned 
        optimizer = self.get_optimizer(hparams)
        device = hparams.general.device
        # extra hparam to scale loss
        C = hparams.general.C
        
        # getting selected augmentation from UI
        transforms = self.get_transforms()
        train_dataset = CustomDataset(
            train_set, 
            project_meta=g.project_fs.meta, 
            classes=classes, 
            input_size=hparams.general.input_image_size,
            transforms=transforms, 
        )
        val_dataset = CustomDataset(
            val_set, 
            project_meta=g.project_fs.meta, 
            classes=classes, 
            input_size=hparams.general.input_image_size
        )
        train_loader = DataLoader(
            train_dataset, 
            batch_size=hparams.general.batch_size, 
            shuffle=True,
            num_workers=hparams.general.workers_number
        )
        val_loader = DataLoader(
            val_dataset, 
            batch_size=hparams.general.batch_size,
            num_workers=hparams.general.workers_number
        )

        # it will return None if pretrained model weights isn't selected in UI
        if self.pretrained_weights_path:
            self.model.load_state_dict(torch.load(self.pretrained_weights_path))
            sly.logger.info('Model checkpoint successfully loaded.')
        
        model.to(device)
        with self.progress_bar(message=f"Training...", total=hparams.general.number_of_epochs) as pbar:
            self.model.train()
            # change training and eval loops for your model if needed
            for epoch in range(hparams.general.number_of_epochs):
                train_total_samples = 0
                train_sum_loss = 0
                train_correct = 0 
                for batch_idx, (images, classes, bboxes) in enumerate(train_loader):
                    batch_size = images.shape[0]
                    images = images.to(device).float()
                    classes = classes.to(device)
                    bboxes = bboxes.to(device).float()

                    pred_classes, pred_bboxes = self.model(images)
                    loss_class = F.cross_entropy(pred_classes, classes, reduction="sum")
                    loss_bb = F.l1_loss(pred_bboxes, bboxes, reduction="none").sum(1)
                    loss_bb = loss_bb.sum()
                    loss = loss_class + loss_bb / C
    
                    optimizer.zero_grad()
                    loss.backward()
                    optimizer.step()
                    train_total_samples += batch_size
                    train_sum_loss += loss.item()
                    
                    _, pred = torch.max(pred_classes, 1)
                    train_correct += pred.eq(classes).sum().item()
                train_loss = train_sum_loss / train_total_samples
                train_accuracy = train_correct / train_total_samples
                
                if hasattr(hparams.intervals, 'validation'):
                    if epoch % hparams.intervals.validation == 0:
                        model.eval()
                        val_total_samples = 0
                        val_sum_loss = 0
                        val_correct = 0 
                        for batch_idx, (images, classes, bboxes) in enumerate(val_loader):
                            batch_size = images.shape[0]
                            images = images.to(device).float()
                            classes = classes.to(device)
                            bboxes = bboxes.to(device).float()

                            pred_classes, pred_bboxes = self.model(images)
                            loss_class = F.cross_entropy(pred_classes, classes, reduction="sum")
                            loss_bb = F.l1_loss(pred_bboxes, bboxes, reduction="none").sum(1)
                            loss_bb = loss_bb.sum()
                            loss = loss_class + loss_bb / C

                            val_sum_loss += loss.item()
                            val_total_samples += batch_size

                            _, pred = torch.max(pred_classes, 1)
                            val_correct += pred.eq(classes).sum().item()
                        val_loss = val_sum_loss / val_total_samples
                        val_accuracy = val_correct / val_total_samples

                if hasattr(hparams.intervals, 'сheckpoints'):
                    if epoch % hparams.intervals.сheckpoints == 0:
                        temp_checkpoint_path = os.path.join(g.checkpoints_dir, f'model_epoch_{epoch}.pth')
                        torch.save(self.model.state_dict(), temp_checkpoint_path)
                        self.log('add_text', tag='Main logs', text_string=f"Model saved at:\n{temp_checkpoint_path}")

                if epoch % hparams.intervals.logging == 0:
                    # common method to logging your values to dashboard
                    # you log values only by your own logger if it setted just call it by class name
                    # self.loggers.YOUR_LOGGER.add_scalar(tag='Loss/train', scalar_value=train_loss, global_step=epoch)
                    self.log('add_scalar', tag='Loss/train', scalar_value=train_loss, global_step=epoch)
                    self.log('add_scalar', tag='Loss/val', scalar_value=val_loss, global_step=epoch)
                    self.log('add_scalar', tag='Accuracy/train', scalar_value=train_accuracy, global_step=epoch)
                    self.log('add_scalar', tag='Accuracy/val', scalar_value=val_accuracy, global_step=epoch)
                
                self.log('add_text', tag='Main logs', text_string=f"Epoch: {epoch}\t|\tTrain loss: {train_loss:.3f}\t|\tVal loss: {val_loss:.3f}\t|\tTrain accuracy: {train_accuracy:.3f}\t|\tVal accuracy: {val_accuracy:.3f}")
                pbar.update(1)
            pbar.set_description_str("Training has been successfully finished")
```

</details>

**Step 4.** Configure your dashboard using parameters and run the app. That's all. 😎

<details>

<summary>Example</summary>

```python
###########################
### 4. Run dashboard app
###########################
dashboard = CustomTrainDashboard(
    # REQUIRED
    # your neural network to train
    model=model, 
    # These titles will be used for logging values while training as part of tags
    # self.log('add_scalar', tag='Loss/train', scalar_value=train_loss, global_step=epoch)
    plots_titles=['Loss', 'Accuracy'],
    # OPTIONAL
    # This row will add an additional hyperparam to the UI. 
    # You will have access to this hyperparam inside CustomTrainDashboard as
    # hparams = self.get_hyperparameters()
    # C = hparams['general']['C']
    extra_hyperparams={
        'general': [
            dict(key='C',
                title='Bbox loss scale', 
                description='Divide bbox_loss for this value', 
                content=InputNumber(1000, min=1, max=100000, size='small')),
        ],
    },
)
app = dashboard.run()
```

</details>



## How to customize the dashboard?

### Configuration via parameters

This section provide detailed information about parameters for ObjectDetectionTrainDashboad initialize and how to change it.

```python
class ObjectDetectionTrainDashboad:
    def __init__(
            self, 
            # ↓↓↓ required ↓↓↓
            model,
            plots_titles
            # ↓↓↓ optional ↓↓↓ 
            pretrained_weights 
            hyperparameters_categories
            extra_hyperparams
            hyperparams_edit_mode
            show_augmentations_ui
            extra_augmentation_templates
            download_batch_size
            loggers
        ):
        ...
```

**pretrained\_weights**: `Dict` - it defines the table of pretraned model weights in UI

<details>

<summary>Details</summary>

If the provided path doesn't exist in the local filesystem at `sly_globals.checkpoints_dir`, it will be downloaded from Team files.

You can read more about `sly_global` in the [Additional notes](training-dashboard.md#additional-notes) section

Example

```python
pretrained_weights = {
    'columns': ['Name', 'Description', 'Path'], # table headers
    'rows': [
        # The path can be local path, team files path or url
        ['Unet', 'Vanilla Unet', 'data/checkpoints/unet.pth'], # local path
        ['Unet-11', 'VGG16', '/data/checkpoints/unet11.pth'], # team files path
        ['Unet-16', 'VGG11', 'https://your_file_server/unet16.pth'] # url (in the future releases)
    ]
}
```

The "Pretrained weights" tab will appear in the model settings card automatically.

![](../../../.gitbook/assets/custom\_weights\_tab.png)

</details>

**hyperparameters\_categories**: `List` - list of tabs names in hyperparameters UI.

<details>

<summary>Details</summary>

Default: `['general', 'checkpoints', 'optimizer', 'intervals', 'scheduler']`

These names also will be used as parent keys for hyperparams from corresponding tabs. You can add/delete tabs by this parameter in hyperparameters card.

Example

```python
dashboard = CustomTrainDashboard(
    ...
    hyperparameters_categories = ['general', 'intervals']
)
```

Before

![](../../../.gitbook/assets/hparams\_tabs.png)

After

![](../../../.gitbook/assets/hparams\_tabs\_2.png)

</details>

**extra\_hyperparams**: `Dict` - they will be added at the end of list hyperparams in the tab by passed tab name, which used as parent key.

<details>

<summary>Details</summary>

Extra hyperparam structure

```python
{
    'any_tab_name': [
        {
            'any_key': str, # by key you will get access to widget value. hparams.any_tab_name.any_key
            'title': str, 
            'description': str,
            'content': sly.Widget # any widget from sly.app.widgets with "get_value" method
        }
    ]
}
```

`any_tab_name` should be unique string.

`any_key` should be unique string for corresponding tab, but it can be repeatet on a another tab. See example below.

`content` work correctly only with `sly.app.widgets`, which have `get_value` method.

In other cases you have two options:

* implement `get_value` method for your widget
* modify `get_hyperparameters` method for support custom widgets

Example:

```python
extra_hyperparams={
    # adding "addition_hparam1" and "addition_hparam2" to "general" tab
    'general': [
        dict(key='addition_hparam_1',
            title='Addition hyperparameter 1', 
            description='Some description about this hyperparameter', 
            content=InputNumber(1, min=1, max=1000, size='small')),
        dict(key='addition_hparam_2',
            title='Addition hyperparameter 1', 
            description='Some description about this hyperparameter', 
            content=InputNumber(6, min=2, max=10, step=2, size='small')),
    ],
    # adding duplicated "addition_hparam1" to "checkpoints" tab
    'checkpoints': [
        dict(key='addition_hparam_1',
            title='Addition hyperparameter 1', 
            description='Some description about this hyperparameter', 
            content=InputNumber(0.0001, min=0.0001, step=0.0001, size='small')),
    ],
}

dashboard = CustomTrainDashboard(
    ...
    extra_hyperparams = extra_hyperparams
)
```

The General tab ![](../../../.gitbook/assets/extra\_hparams\_1.png)

The Checkpoints tab ![](../../../.gitbook/assets/extra\_hparams\_2.png)

</details>

**hyperparams\_edit\_mode**: `String` - the ways to define hyperparameters.

<details>

<summary>Details</summary>

Default: \`'ui'\`

Supported values: \[`'ui', 'raw', 'all'`]

`ui` - only 🟢 section will be shown.

`raw` - only 🔴 section will be shown.

`all` - 🟢 + 🔴 sections will be shown together.

![](../../../.gitbook/assets/raw\_hyperparams.png)

The hyperparams from UI will overwrite hyperparams with the same names from the text editor widget.

For example, if you declare `hparam_1` with "general" as the parent key in extra\_hyperparams or in hyperparameters\_ui method

```python
'general': [
    dict(key='hparam_1',
        title='Hyperparameter 1', 
        description='Some description', 
        content=InputNumber(100, min=100, max=1000, size='small')),
    ]
```

and declare the same in the text editor widget

```yaml
general:
    hparam_1: 0.1
```

then when you will call `get_hyperparameters` method, the `hparam_1` value will be equal to `100`, not `0.1`.

</details>

**show\_augmentations\_ui**: `Bool` - show/hide flag for augmentations card

Default: `True`

**extra\_augmentation\_templates**: `List` - these augmentations templates will be added to beginning of the list for selector in augmentations card:

<details>

<summary>Details</summary>

You can create your own augmentations template `.json` using [ImgAug Studio app](https://dev.supervise.ly/ecosystem/apps/imgaug-studio).

Example:

```python
AUG_TEMPLATES = [
    # label - just title for selector option
    # value - local path
    {'label': 'My aug 1', 'value':'aug_templates/light.json'},
    {'label': 'My aug 2', 'value':'aug_templates/light_corrupt.json'},
    {'label': 'My aug 3', 'value':'aug_templates/medium.json'},
]
```

If you will set hyperparams\_edit\_mode to `raw` or `all`, this additional widget will be shown.

![](../../../.gitbook/assets/extra\_augs.png)

</details>

**download\_batch\_size**: `int` - How much data to download per batch. Increase this value for speedup download on big projects.

Default: 100

**loggers**: `List` - additional user loggers

<details>

<summary>Details</summary>

Example:

```python
from torch.utils.tensorboard import SummaryWriter

class CSVWriter:
    def __init__(self, log_dir):
        # your code
        pass
    def add_scalar(tag, scalar_value, global_step):
        # your code
        pass

my_csv_logger = CSVWriter(g.csv_log_dir)
my_tensorboard = SummaryWriter(g.tensorboard_runs_dir)
loggers=[my_csv_logger, my_tensorboard]
```

You can log value for all loggers by calling common method.

All passed loggers should have the called method.

```python
self.log(method='add_scalar', tag='Loss/train', scalar_value=train_loss, global_step=epoch)
```

If you want to log value for specific logger, then use `self.loggers.YOUR_LOGGER_CLASS`

```python
self.loggers.SummaryWriter.add_scalar(tag='Loss/train', scalar_value=train_loss, global_step=epoch)
```

</details>



### Configuration via methods re-implemetation

#### How to change all hyperparameters in the hyperparameters card?

All what you neeed is just re-define `hyperparameters_ui` method in subclass of `ObjectDetectionTrainDashboad`

<details>

<summary>Example</summary>

```python
class CustomTrainDashboard(ObjectDetectionTrainDashboad):
    def hyperparameters_ui(self):
        hparams_widgets = {}
        # adding widgets to "my_hparam_tab" in IU. "my_hparam_tab" will be used as parent key.
        if 'my_hparam_tab' in self._hyperparameters_categories:
            hparams_widgets['my_hparam_tab'] = [
                # adding hparam 
                dict(key='key_for_hparam',
                    title='Name for added hyperparameter', 
                    description='Any description for added hyperparameter', 
                    # any widget from sly.app.widgets with "get_value" method or 
                    # you should redefine "get_hyperparameters" method to handle with these widgets
                    content=InputNumber(10, min=1, max=100000, size='small')),
                # this string required for adding additional widgets to "my_hparam_tab" 
                *self._extra_hyperparams.get('my_hparam_tab', [])
            ]
# Add new tab name to the list for displaying them in UI
hparams_tabs = ['my_hparam_tab']
dashboard = CustomTrainDashboard(
    ...
    hyperparameters_categories = hparams_tabs
)
```

</details>



## Additional notes

Environment variable `SLY_APP_DATA_DIR` in `src.globals` is used to provide access to app files when the app will be finished. If something went wrong in your training process at any moment - you won't lose checkpoints and other important artifacts. They will be available by SFTP.

By default [object detection training template app](https://github.com/supervisely-ecosystem/object-detection-training-template/blob/d1278af846d9bd30bd39ca4138a3a8f90870955e/src/sly\_globals.py#L25) use this directoties structure from `src/sly_globals`:

```python
|object-detection-training-template
├─ project_dir # project training data destination folder
└─ data_dir # All training artefacts. This dir will be saved in Team files at `remote_data_dir` at the end of training process.
  ├─ checkpoints_dir #  Model checkponts will be saved here. This dir included in `data_dir`.
  └─ tensorboard_runs_dir # This dir will be created if tensorboard ResultsWriter was passed in loggers list 
```

`remote_data_dir` = `f"/train_dashboard/{project.name}/runs/{time.strftime('%Y-%m-%d %H:%M:%S')}"` - the destination dir in Team files for all training artefacts.
