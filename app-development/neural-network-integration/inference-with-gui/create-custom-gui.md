# Create custom user interface


- we provide template, but you can create custom GUI
- our template inherited from BaseInferenceGUI class. you can inherit from this or from our template to save base functionality
- BaseInferenceGUI methods description: get_device(), get_ui(), set_deployed() and props: serve_button and download_progress.
- initialize_gui() method in Inference
- example: Serve MMDetection