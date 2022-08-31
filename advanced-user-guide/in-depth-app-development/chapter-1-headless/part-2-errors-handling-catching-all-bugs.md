---
description: In this part, we will work with bugs. We will catch them.
---

# Part 2 — Errors handling \[Catching all bugs]

### Table of contents

1. [Supervisely logger](part-2-errors-handling-catching-all-bugs.md#step-1-supervisely-logger)
2. [Use \[try: except\] with traceback](part-2-errors-handling-catching-all-bugs.md#step-2-use-try-except-with-traceback)
3. [Results](part-2-errors-handling-catching-all-bugs.md#step-3-results)

### Step 1 — Supervisely logger

Supervisely has a logger. It is a wrapper over Python's built-in [logging](https://docs.python.org/3/howto/logging.html) package.

First thing we need to do is install Supervisely Python SDK to your environment:

`pip install supervisely`

Let's take the code from Part 1 as a basis and add a logger:

```python
import supervisely_lib as sly

worlds = ['Westeros', 'Azeroth', 'Middle Earth', 'Narnia']

sly_logger = sly.logger

for world in worlds:
    sly_logger.info(f'Hello {world}!')
```

### Step 2 — Use \[try: except] with traceback

To handle errors, you can use the `[try: except]` construction

**src/main.py**

```python
import supervisely_lib as sly

worlds = ['Westeros', 'Azeroth', 'Middle Earth', 'Narnia']

sly_logger = sly.logger

try:
    for world in worlds:
        sly_logger.info(f'Hello {world}!')

    if 'Our World' not in worlds:
        raise ValueError(f"Can't find Our World")

except Exception as ex:
    sly_logger.warning(ex)
```

### Step 3 — Results

{% embed url="https://www.youtube.com/watch?v=MMcNsW3wI_I" %}
Error Handling
{% endembed %}
