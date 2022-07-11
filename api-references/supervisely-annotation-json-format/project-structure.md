# Project Structure

In Supervisely all data and annotations are stored inside individual projects which themselves consist of datasets with files in them, and Project Meta - series of classes and tags.

When downloaded, each project is converted into a folder that stores `meta.json` file containing Project Meta, dataset folders with the individual annotation files (and optionally the original data files) in them. This allows you to seamlessly cycle data between Supervisely and local storage with the use of `Supervisely Format` import plugin, if you so require.

This structure remains the same for every type of project in Supervisely.

![](../../.gitbook/assets/project\_structure.png)

**Project Folder**

****
