# App development: FAQ

In this section, we will answer the most common questions about developing apps for Supervisely.

## How can we reset the app's UI after each use?
This depends on the complexity of the app's UI. If it's simple (e.g. it has not so many widgets), you can just add a [Button](https://developer.supervisely.com/app-development/widgets/controls/button) widget and manually set everything you need.<br>
But if it's complex, you need to know about two things in our engine.
Basically everything in the UI is stored in [DataJson](https://github.com/supervisely/supervisely/blob/master/supervisely/app/content.py#L166) object and [StateJson](https://github.com/supervisely/supervisely/blob/master/supervisely/app/content.py#L128) object. They are both Singletons.<br>
In `DataJson` we usually store some permanent widget settings: width, styles, etc. While in the `StateJson` we usually store dynamic stuff like selected items and so on. They are both simple dictionaries on steroids with the same parent class. So what you need to do, is to define their "default" state and apply it when needed.<br>
Usually we use the [send_changes()](https://github.com/supervisely/supervisely/blob/master/supervisely/app/content.py#L120) method, when we need to apply the changes.<br>
<br>
But there is another approach. We have a very special widget called [ReloadableArea](https://developer.supervisely.com/app-development/widgets/layouts-and-containers/reloadablearea).<br>
With it you can dynamically create or remove new widgets in the app. So the head-on approach to solving the problem would be simple pre-define some layout, and when it's not needed anymore delete it and create a new one from template. This way you can not only reset the UI, but completely create a new one.