# Workflow Automation with CLI

{% hint style="warning" %}
### Beta. Release coming soon.
{% endhint %}

This section of the documentation will help you understand how to use CLI tool to effectively communicate with instance. The commands in this module will allow you to manage projects, upload files, etc.

# Commands

## Login

Authorize on a Supervisely instance to access the API to execute commands. Overwrite existing `.env` file and override environment variables. A backup file will be created for the `.env` file. Backup files will be stored for the last 5 authorizations.

```bash
supervisely instance login [OPTIONS]
```

### Command options:
- `-s` / `--server-address` - Server address.
- `-l` / `--login` - User login.
- `-p` / `--password` - User password.

### Usage examples

**1. Full interactive mode from scratch**
   
   ```bash
   supervisely instance login

   Select Instance type:  (Use arrow keys)
     Community Edition
   » Enterprise Edition

   Enter server address here >> thiscompany.supervisely.com  # this step will be skipped for selection "Community Edition"
   Enter login here >> user_1
   Enter password here >> ************
   User with login user_1 successfully authenticated on https://thiscompany.supervisely.com
   ```

**2. Check authorization**

   In case you have already been authorized and have valid data, information about which user on which server is authorized will be shown.

   ```bash
   supervisely instance login
   
   User with login user_1 successfully authenticated on https://thiscompany.supervisely.com
   ```

**3. Overwrite data / one-command authorization**

   If you want to change users and keep that authorization, or just want to authorize non-interactively, then use all options at once.

   ```bash
   supervisely instance login -s https://thiscompany.supervisely.com -l user_1 -p password
   
   User with login user_1 successfully authenticated on https://thiscompany.supervisely.com
   ```

## Upload entity files

Upload entity files from a specified directory into a new or existing project or dataset.

```bash
supervisely instance $entity_type upload [OPTIONS]
```
`$entity_type` - command group name

### Possible entity types:
 - `images`
 - `videos`
 - `volumes`
 - `point-clouds`
 - `point-cloud-episodes`

### Command options:
- `-d` / `--dirs` - Path to the local directories with files. It is possible to specify multiple directories, e.g. '-d /path/to/dir1 -d /path/to/dir2'. `required`
- `-tid` / `--team-id` - Team ID.
- `-wid` / `--workspace-id` - Workspace ID.
- `-pid` / `--project-id` - ID of an existing project.
- `-did` / `--dataset-id` - ID of an existing dataset.
- `-pn` / `--project-name` - Name of the project to be created.
- `-dn` / `--dataset-name` - Name of the dataset to be created.
- `-ep` / `--existing-project` - This option is only used in interactive mode, where you choose which of the available projects to upload files to. It is not possible to use with '--project-id', because by using '--project-id' you are already indicating that you are going to upload data into a specific existing project.
- `-ed` / `--existing-dataset` - This option is only used in interactive mode, where you choose which of the available datasets to upload files to. It is not possible to use with '--dataset-id', because by using '--dataset-id' you are already indicating that you are going to upload data into a specific existing dataset.

{% hint style="info" %}
If there are no credentials in the environment variables or no `supervisely.env` file, then the login method will be initiated whenever the file upload command is initialized. The entered data will only be used to execute the current command. To avoid entering this data each time, use the `login` command first.
{% endhint %}

### Usage examples

**1. Upload via Interactive mode**
   
   ```bash
   supervisely instance $entity_type upload -d /path/to/dir
   ```

   The team and workspace selection wizard will be initialized. Use the up and down arrow keys on the keyboard to make a selection, and the Enter key to confirm the selection.
   
   ```bash
   Select Team:  (Use arrow keys)
     team_name_1
   » team_name_2
     team_name_3
   ```

   The next steps will prompt you to enter the name of the project and the dataset. It is necessary to enter the name of the project, in turn it is not necessary to enter the name of the dataset, in this case the name of the dataset will be the name of the first directory listed.

   ```bash
   Creating new project. Please enter poject name here >> Project_1
   Creating new dataset. Please press Enter to use the first dir name or write dataset name here >> Dataset_1
   ```
   
   After the files are successfully uploaded, information with a link to the project will be displayed.

   ```bash
    Uploading images to project [Project_1]: 100%|████████| 6/6 [00:00<00:00, 6.76it/s]
    ===============================================

    To access the project, please follow this link:
    https://app.supervisely.com/projects/1111/datasets

    ===============================================
   ```

**2. Upload via Semi-predefined mode**
   
   You can specify any of the IDs to skip some wizard steps. 
   
   For example, if you specify only the workspace ID, you will go straight to the step of entering the project name.

   ```bash
   supervisely instance $entity_type upload -d /path/to/dir -wid 100

   User with login user_1 successfully authenticated on https://app.supervisely.com
   Creating new project. Please enter poject name here >> Project_1
   Creating new dataset. Please press Enter to use the first dir name or write dataset name here >> Dataset_1
   ...
   ```

   If you specify a workspace ID and a dataset name, you will only go through the step of entering the project name.

   ```bash
   supervisely instance $entity_type upload -d /path/to/dir -wid 100 -dn Dataset_1
   
   User with login user_1 successfully authenticated on https://app.supervisely.com
   Creating new project. Please enter poject name here >> Project_1
   ...
   ```
   In case you upload something into an existing project and want to select the project interactively - you need to specify option `-ep`

   ```bash
   supervisely instance $entity_type upload -d /path/to/dir -wid 100 -ep -dn Dataset_1
   
   User with login user_1 successfully authenticated on https://app.supervisely.com
   Select Project:  (Use arrow keys)
   » Project_1
     Project_2
     Project_3
   ...
   ``` 

   And if you add the `-pn` parameter to the command above, then the results will be filtered and only projects with the `-pn` argument will be displayed. 
 
   ```bash
   supervisely instance $entity_type upload -d /path/to/dir -wid 100 -ep -pn Proj -dn Dataset_1
   
   User with login user_1 successfully authenticated on https://app.supervisely.com
   There were found 3 projects with similar names.
   Select Project:  (Use arrow keys)
   » Projector
     Project_1
     test_Project_one
   ...
   ``` 

   In the above example, if no matches are found, you will see a list of all existing projects in the current workspace.

**3. Upload via One-command mode**
   
   If you are going to automate some processes with scripts that will use CLI tool, it is possible to set all parameters for successful uploading.
   
   In this case, be sure to initiate the login command before calling commands that communicate with the instance.

   ```bash
   supervisely instance $entity_type upload -d /path/to/dir_1 -wid 100 -pn Project_4 -dn Dataset_1
   # here you can write a code to extract project ID from stdout
   supervisely instance $entity_type upload -d /path/to/dir_2 -pid 1111 -dn Dataset_2
   ...
   ```