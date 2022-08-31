---
description: >-
  In this part, we will introduce you to app handlers and tell you what they are
  for.
---

# Part 3 — APP Handlers \[Handle Events and Errors]

### Table of contents

1. [Handle HTML events](part-3-app-handlers-handle-events-and-errors.md#step-1-handle-html-events)
2. [Dialog window instead of an error](part-3-app-handlers-handle-events-and-errors.md#step-2-dialog-window-instead-of-an-error)
3. [Results](part-3-app-handlers-handle-events-and-errors.md#step-3-results)

### Step 1 — Handle HTML events

How do I make the IU buttons work?\
It's very simple.\
In the HTML file, I will create a button that invokes the command on **click**:

**src/gui.html** (partially)

```html
<div>
	<el-button
		type="success"
	    	@click="command('normal_handler')">
        		Call normal handler
	 </el-button>
</div>
```

In Python code I will handle this command using callback handler

**src/main.py** (partially)

```
@app.callback('normal_handler')
def normal_handler(api: sly.Api, task_id, context, state, app_logger):
```

This callback is triggered if the command name matches the name of the callback parameter.\
**Arguments that come to the input:**

* api — api the object of the user who called the callback
* task\_id — app task\_id
* context — information about the environment in which the application is running
* state — state of all widgets in Python dict format
* app\_logger — sly\_logger with task\_id

### Step 2 — Dialog window instead of an error

Sometimes we want the application not to crash after an error occurs.\
Therefore, we can use `app.ignore_errors_and_show_dialog_window()` handler:

**src/main.py** (partially)

```python
@app.callback('error_handler')
@app.ignore_errors_and_show_dialog_window()
def error_handler(api: sly.Api, task_id, context, state, app_logger):
    logger.info('normal handler called')
    raise SystemError('there could be an error message here')
```

### Step 3 — Results

{% embed url="https://www.youtube.com/watch?v=U2XtONhiZaw" %}
App Handlers
{% endembed %}
