---
description: >-
  In this section, we will customize the modal window. We will use a modal
  window to enter arguments.
---

# Part 2 — States and Widgets \[Customize modal window]

### Table of contents

1. [About states](part-2-states-and-widgets-customize-modal-window.md#step-1-about-states)
2. [Element widgets](part-2-states-and-widgets-customize-modal-window.md#step-2-element-widgets)
3. [Blind it all together](part-2-states-and-widgets-customize-modal-window.md#step-3-blind-it-all-together)
4. [Results](part-2-states-and-widgets-customize-modal-window.md#step-4-results)

### Step 1 — About states

Have you heard about the state machine? It is the same.\
State fields help the application core store lightweight data and pass it to Python as needed.

![warning](https://github.githubassets.com/images/icons/emoji/unicode/26a0.png) To use `states` in our application, we need to initialize them.\
For a modal window application, this does it in **config.json**

**config.json** (partially)

```json
"modal_template_state": {
  "timerValue": "10"
},
```

**Remember the states.**\
We'll need them to pass arguments from HTML to Python.

### Step 2 — Element widgets

[Element widgets](https://element.eleme.io/1.4/#/en-US/component/input-number) are available in our core.\
We can easily add an input box:

**src/modal.html**

```html
<div>
     <h3>Timer value (in seconds):</h3>
     <el-input-number v-model="state.timerValue" :min="1" :max="60"></el-input-number>
</div>
```

Have you forgotten about **states** yet?\
Using the **v-model** parameter, we bind the APP core to the UI.

### Step 3 — Blind it all together

Python code is a regular timer:

**src/main.py**

```python
import supervisely_lib as sly
import time

import os
from dotenv import load_dotenv  # pip install python-dotenv
                                # don't forget to add to requirements.txt!

# Loading env files
load_dotenv("../debug.env")
load_dotenv("../secret_debug.env", override=True)

# Extracting variables
address = os.environ['SERVER_ADDRESS']
token = os.environ['API_TOKEN']

team_id = int(os.environ['context.teamId'])
workspace_id = int(os.environ['context.workspaceId'])

# Extracting world name from modal window
timer_value = int(os.environ['modal.state.timerValue'])


sly_logger = sly.logger
starting_time = time.time()

while time.time() - starting_time < timer_value:
    sly_logger.info(f'Time left {timer_value - (time.time() - starting_time)}')
    time.sleep(1)

sly_logger.warning(f'The timer has rang! DZZZZ')
```

### Step 4 — Results

{% embed url="https://www.youtube.com/watch?v=Y8eR4XRD9AM" %}
States and Widgets
{% endembed %}
