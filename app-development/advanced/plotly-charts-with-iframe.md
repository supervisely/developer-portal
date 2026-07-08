м---
description: Guide explains how to render Plotly charts in an app GUI using the IFrame widget
---

# Rendering Plotly charts with the IFrame widget

Supervisely's widget library has no native Plotly component, so the common pattern is: build a Plotly figure, export it to a static `.html` file, and display that file through the [IFrame](../widgets/layouts-and-containers/iframe.md) widget.

## How it works, in short

1. Plotly renders a figure to a self-contained HTML file.
2. That file is saved into your app's `static_dir`.
3. The `IFrame` widget points at that file (by its path relative to `static_dir`) and displays it in the GUI.
4. To update the chart, you write a new HTML file and call `iframe.set(...)` again.

## 1. Imports

```python
import os
from pathlib import Path
import numpy as np
import plotly.graph_objects as go
import supervisely as sly
from supervisely.app.widgets import Container, Button, IFrame
```

## 2. Create a folder for generated HTML files

Plotly needs somewhere to write its output, and that location must live inside the app's `static_dir` so Supervisely can serve it.

```python
if not os.path.exists("previews"):
    os.mkdir("previews")

previews_counter = 0  # used to give every render a unique filename
```

## 3. Declare the IFrame widget

Create it once, set a default size, and hide it until there's something to show:

```python
preview_image = IFrame()
preview_image.set_plot_size(height="500px", width="700px")
preview_image.hide()
```

You can also pass size directly in the constructor: `IFrame(height="500px", width="700px")`.

## 4. Register `static_dir` on the `Application`

This is the step that's easy to miss: the folder you write Plotly HTML into must be passed as `static_dir` when creating the app, otherwise `IFrame` has nothing to serve.

```python
app = sly.Application(
    layout=Container(widgets=[preview_image]),
    static_dir=Path("previews"),
)
```

## 5. Build the Plotly figure

Any `plotly.graph_objects.Figure` works. Example adapted from the source file — a 3D scatter plot separating ground vs. non-ground points:

```python
fig = go.Figure(
    data=[
        go.Scatter3d(
            x=nonground_x, y=nonground_y, z=nonground_z,
            mode="markers",
            marker=dict(size=2, opacity=0.8, color="lawngreen"),
            name="nonground",
            showlegend=True,
        ),
        go.Scatter3d(
            x=ground_x, y=ground_y, z=ground_z,
            mode="markers",
            marker=dict(size=2, opacity=0.8, color="red"),
            name="ground",
            showlegend=True,
        ),
    ]
)

fig.update_layout(
    title=dict(text="Ground detection preview", automargin=True, x=0.5),
    margin=dict(l=0, r=0, b=0, t=0),
    scene=dict(xaxis_title="X", yaxis_title="Y", zaxis_title="Z", aspectmode="data"),
)
fig.update_scenes(xaxis_visible=False, yaxis_visible=False, zaxis_visible=False)
```

## 6. Export the figure to HTML and point the IFrame at it

```python
local_preview_path = f"previews/preview_{previews_counter}.html"
fig.write_html(local_preview_path)

preview_image.set(
    path_to_html=f"static/preview_{previews_counter}.html",  # relative path used inside static_dir
    height="500px",
    width="1000px",
)

previews_counter += 1
```

{% hint style="info" %}
`write_html` uses your **local** filesystem path (`previews/preview_0.html`), while `IFrame.set(path_to_html=...)` uses the path **as served under `/static`** (`static/preview_0.html`). `path_to_html` must not start with a leading `/`.
{% endhint %}

Using an incrementing counter (or any unique name) for each file avoids browser caching showing a stale chart.

## 7. Wire it up to a button, with loading state

Putting it all together in a click handler, matching the pattern used in the source app:

```python
preview_button = Button("Preview")

@preview_button.click
def draw_preview():
    preview_image.clean_up()      # clear previous html
    preview_image.show()
    preview_image.loading = True

    # ... compute nonground_x/y/z, ground_x/y/z ...

    fig = go.Figure(data=[...])
    fig.update_layout(...)

    local_preview_path = f"previews/preview_{previews_counter}.html"
    fig.write_html(local_preview_path)

    preview_image.set(
        path_to_html=f"static/preview_{previews_counter}.html",
        height="500px",
        width="1000px",
    )

    preview_image.loading = False
    previews_counter += 1
```

(`global previews_counter` is needed if the counter is declared at module level, as in the original app.)

## Full minimal example

```python
import os
from pathlib import Path
import numpy as np
import plotly.graph_objects as go
import supervisely as sly
from supervisely.app.widgets import Container, Button, IFrame

if not os.path.exists("previews"):
    os.mkdir("previews")

previews_counter = 0

plot_widget = IFrame()
plot_widget.set_plot_size(height="500px", width="700px")
plot_widget.hide()

render_button = Button("Render chart")

app = sly.Application(
    layout=Container(widgets=[render_button, plot_widget]),
    static_dir=Path("previews"),
)

@render_button.click
def render():
    global previews_counter

    plot_widget.clean_up()
    plot_widget.show()
    plot_widget.loading = True

    x = np.random.rand(200)
    y = np.random.rand(200)
    z = np.random.rand(200)

    fig = go.Figure(data=[go.Scatter3d(x=x, y=y, z=z, mode="markers")])
    fig.update_layout(margin=dict(l=0, r=0, b=0, t=0))

    path = f"previews/preview_{previews_counter}.html"
    fig.write_html(path)

    plot_widget.set(
        path_to_html=f"static/preview_{previews_counter}.html",
        height="500px",
        width="700px",
    )

    plot_widget.loading = False
    previews_counter += 1
```

## Tips and common pitfalls

The `path_to_html` given to `IFrame.set()` is relative to `static_dir` and must not start with a forward slash — pass `static/preview_0.html`, not `/static/preview_0.html`.

`static_dir` in `sly.Application(...)` must be the same folder Plotly's `fig.write_html()` writes into, just referenced by its local path (e.g. `Path("previews")` vs. `previews/preview_0.html`).

Give each generated file a unique name (counter, timestamp, or UUID) so `IFrame.set()` reliably shows the newest chart instead of a cached one.

Call `clean_up()` before generating a new chart if you want to visibly clear the widget while the next one is being computed, and toggle `loading = True/False` around the computation to show a spinner.

If the app deletes its working files on shutdown (as the source app does via `clean_data()`), make sure the `previews` folder isn't removed while a chart is still being viewed.

## Sources

[interactive_3d_detection/serving_app/main.py](https://github.com/supervisely-ecosystem/pointcloud_experiments/blob/f0a3608ef4650015808a633e02881badcf08efeb/interactive_3d_detection/serving_app/main.py#L192)

[IFrame widget documentation](../widgets/layouts-and-containers/iframe.md)
