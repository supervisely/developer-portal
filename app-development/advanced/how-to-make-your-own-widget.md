---
description: Guide explains how to create and add your own widget to Supervisely SDK
---

# Introduction
In this tutorial you will learn how to create your own widget, add it to Supervisely SDK and make small demo.

## How to create your own widget
1. Clone SKD repo:

    ```bash
    git clone https://github.com/supervisely/supervisely
    ```

2. Clone repo with widgets samples:

    ```bash
    git clone https://github.com/supervisely-ecosystem/ui-widgets-demos
    ```

3. Create SDK folder symlink in ui-widgets-demos

    ```bash
    ln -s ./supervisely/supervisely ./ui-widgets-demos/supervisely
    ```

4. Create folder in `supervisely/app/widgets/<your_widget_folder>` like that
    ```
    your_widget_folder
    |____ __init__.py
    |____ template.html
    |____ script.js   # optional: you can add javascript file
    |____ your_widget.py
    ```
5. Declare a new class with inheritance from Widget in `your_widget.py`
    ```python
    from supervisely.app.widgets import Widget
    from supervisely.app.content import StateJson, DataJson 


    class YourWidget(Widget):
        # ------ Required methods ------
        def __init__(
                self,
                data_1: list 
                data_2: dict 
            ):
            self._data_1 = data_1
            self._data_2 = data_2
            self._some_state_attribute_1 = True 
            self._some_state_attribute_2 = 42 

            super(YourWidget, self).__init__(widget_id=widget_id, file_path=__file__)

            # ------ Optional | Add your javascript file ------
            script_path = "./sly/css/app/widgets/your_widget_folder/script.js" # change <your_widget_folder> 
            JinjaWidgets().context["__widget_scripts__"][self.__class__.__name__] = script_path

        def get_json_data(self):
            """ This method will be used in template.html to get widget data """
            return {
                "data_1": self._data_1,
                "data_2": self._data_2,
            } 

        def get_json_state(self):
            """ This method will be used in template.html to get widget state """
            return {
                "some_state_attribute_1": self._some_state_attribute_1,
                "some_state_attribute_2": self._some_state_attribute_2,
            } 

        # ------ Optional methods | Just for example ------
        def your_method_for_change_data(self):
            # make changes in widget data
            self._data_1 = self._data_1 + [1, 2, 3, 4, 5]
            self._data_2 = {'a': 11}
            # save this changes of widget data in local storage(RAM) 
            DataJson()[self.widget_id]['data_1'] = self._data_1
            DataJson()[self.widget_id]['data_2'] = self._data_2
            # sync this changes to remote server 
            DataJson().send_changes()
        
        def your_method_for_change_state(self):
            # change state attribute
            self._some_state_attribute_1 = False
            self._some_state_attribute_2 = 0.1
            # save this changes of widget data in local storage(RAM) 
            StateJson()[self.widget_id]["some_state_attribute_1"] = self._some_state_attribute_1
            StateJson()[self.widget_id]["some_state_attribute_2"] = self._some_state_attribute_2
            # sync this changes to remote server 
            StateJson().send_changes()
    ```

6. Construct `template.html` for your widget using other widgets from SDK or any HTML elements
    ```html
    <div>
        <!-- Elements from SDK had the "sly" prefix -->
        <sly-field :title="data.{{{widget.widget_id}}}.data_1" :description="state.{{{widget.widget_id}}}.some_state_attribute_1">
            <div>
                {{data.{{{widget.widget_id}}}.data_2}}
            </div>
            <div>
                {{state.{{{widget.widget_id}}}.some_state_attribute_2}}
            </div>
        </sly-field>
        
        <!-- Just simple HTML element with Javascript function from your scipt.js -->
        <button
            :value="data.{{{widget.widget_id}}}.data_1"
            @click="myFunction()" 
        ></button>

        <!-- Also simple HTML element, but from UI library of HTML elements - https://element.eleme.io -->
        <el-button :value="data.{{{widget.widget_id}}}.data_1"></el-button>
    <div>
    ```
6. Prepare `script.js` for your widget.
    ```javascript
    const myFunction = function (name) {
        console.log('Hello world')
    }
    ```

    >📗 See [this example](https://github.com/supervisely/supervisely/tree/master/supervisely/app/widgets/video_player) if you want to add Javascript file and create your own Vue JS component.

7. Import new widget as part of `widgets` module. Just add import in `supervisely/app/widgets/__init__.py`
    ```python
    # imports for other widgets as example
    from supervisely.app.widgets.radio_tabs.radio_tabs import RadioTabs
    from supervisely.app.widgets.train_val_splits.train_val_splits import TrainValSplits
    from supervisely.app.widgets.editor.editor import Editor
    
    # import for your widget
    from supervisely.app.widgets.your_widget_folder.your_widget_py_file import YourWidgetClass
    ```

8. Create folder with example of your widget in `ui-widgets-demos/NEW widgets/<your_widget_example_folder>` like that
    ```
    your_widget_folder
    |____src
      |____main.py
    ```

    Example of `main.py`
    ```python
    import os
    import supervisely as sly
    from dotenv import load_dotenv
    from supervisely.app.widgets import (
        Card, 
        YourWidget #  <-- import your widget
        )

    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

    api = sly.Api()

    your_widget = YourWidget(data_1=[...], data_2={'c': 0.3355})
    card = Card(title="My card", content=your_widget)
    app = sly.Application(layout=card)
    ```
9. To see the demo of your widget, you need to run the server. You got 2 options for that:
    
    a) Just run the server from command line
        
    `python -m uvicorn "NEW widgets.grid_plot.src.main:app" --host 0.0.0.0 --port 8000 --ws websockets --reload`

    b) If you are VSCode user, then just edit `ui-widgets-demos/.vscode/launch.json` as showed below and run the server from "run & debug" menu

    ```json
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Uvicorn",
                "type": "python",
                "request": "launch",
                "module": "uvicorn",
                "args": [
                    <!-- change "your_widget_example_folder" to the name of the folder with your widget -->
                    "NEW widgets.your_widget_example_folder.src.main:app",
                    "--host",
                    "0.0.0.0",
                    "--port",
                    "8000",
                    "--ws",
                    "websockets",
                    "--reload"
                ],
                "jinja": true,
                "justMyCode": true,
                "env": {
                    "PYTHONPATH": "${workspaceFolder}:${PYTHONPATH}",
                    "LOG_LEVEL": "DEBUG",
                }
            }
        ]
    }
    ```
10. After all steps, test your widget in browser http://0.0.0.0:8000 and if it ok make a commit & push & create the pull request in both repo(SDK and ui-widget-demos).
    
    Send the link to pull-request to Maxim Kolomeychenko(@Max) in Slack. He will run a new SDK build. 

    **Success!**


## Simple widget example

`supervisely/app/widgets/textarea/textarea.py`
```python
from supervisely.app import DataJson, StateJson
from supervisely.app.widgets import Widget


class TextArea(Widget):
    def __init__(
        self,
        value: str = None,
        placeholder: str = "Please input",
        rows: int = 2,
        autosize: bool = True,
        readonly: bool = False,
        widget_id=None,
    ):
        self._value = value
        self._placeholder = placeholder
        self._rows = rows
        self._autosize = autosize
        self._readonly = readonly
        super().__init__(widget_id=widget_id, file_path=__file__)

    def get_json_data(self):
        return {
            "value": self._value,
            "placeholder": self._placeholder,
            "rows": self._rows,
            "autosize": self._autosize,
            "readonly": self._readonly,
        }

    def get_json_state(self):
        return {"value": self._value}

    def set_value(self, value):
        self._value = value
        StateJson()[self.widget_id]["value"] = value
        StateJson().send_changes()

    def get_value(self):
        return StateJson()[self.widget_id]["value"]

    def is_readonly(self):
        return DataJson()[self.widget_id]["readonly"]

    def enable_readonly(self):
        self._readonly = True
        DataJson()[self.widget_id]["readonly"] = True
        DataJson().send_changes()

    def disable_readonly(self):
        self._readonly = False
        DataJson()[self.widget_id]["readonly"] = False
        DataJson().send_changes()
```

`supervisely/app/widgets/textarea/template.html`
```html
<el-input
  type="textarea"
  :placeholder="data.{{{widget.widget_id}}}.placeholder"
  :rows="data.{{{widget.widget_id}}}.rows"
  :autosize="data.{{{widget.widget_id}}}.autosize"
  v-model="data.{{{widget.widget_id}}}.value">
</el-input>
```

`supervisely/app/widgets/__init__.py`
```python
from supervisely.app.widgets.textarea.textarea import TextArea
```

`NEW widgets/textarea/src/main.py`
```python
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, TextArea

load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()

text_area = TextArea(value="Some text in the text area.", autosize=False)
card = Card(title="My card", content=text_area)
app = sly.Application(layout=card)
```
