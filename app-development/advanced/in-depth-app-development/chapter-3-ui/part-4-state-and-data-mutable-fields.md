---
description: >-
  In this part, you will learn how to change the values of state and data
  fields. And how they can be used in the application.
---

# Part 4 — State and Data \[Mutable Fields]

### Table of contents

1. [Mutable state && data fields](part-4-state-and-data-mutable-fields.md#step-1-mutable-state-and-and-data-fields)
2. [GET && SET field](part-4-state-and-data-mutable-fields.md#step-2-get-and-and-set-field)
3. [Results](part-4-state-and-data-mutable-fields.md#step-3-results)

### Step 1 — Mutable state && data fields

Remember the Part 6 State Machine?

{% hint style="info" %}
Let me remind you: in the example from Part 6,\
we wrote the timer value in the **`state.timerValue`** field.\
Then we did not change the state value while the program was running.
{% endhint %}

**But what if we want to change the value of the timer?**

For these purposes, we have `two types of fields` available:

1. **state** — for storing lightweight values
2. **data** — for storing heavyweight values

<img src="https://github.githubassets.com/images/icons/emoji/unicode/26a0.png" alt="warning" data-size="line"> **In order to use mutable fields while the application is running:**

1. All required fields must be initialized before calling `app.run`
2. Fields keys must be written in `CamelCase` register
3. `data` fields can't read the values of widgets,\
   so for widgets you need to use `state`

**src/main.py** (partially)

```python
def main():
	data = {}
	state = {}

	# initialize required fields here
	state['timerValue'] = 0
	data['timeLeft'] = None

	app.run(data=data, state=state)
```

### Step 2 — GET && SET field

To `get` or `set` field, we need to refer to the `api` of the application.\
Here's a simple example:

**src/main.py** (partially)

```python
app = sly.AppService()
app_api = app.public_api
logger = sly.logger


@app.callback('start_timer')
def start_timer(api: sly.Api, task_id, context, state, app_logger):
    timer_value = app_api.app.get_field(task_id=task_id,
                                        field='state.timerValue')  # getting field
    ...
    app_api.app.set_field(task_id=task_id,
                          field='data.timeLeft',
                          payload=0)   # setting field
```

### Step 3 — Results

Let's run the app and take a look at our improved timer!

{% embed url="https://www.youtube.com/watch?v=W-598G9BH4Y" %}
State and Data
{% endembed %}

\
