# Enterprise Module

This module ...

# Usage

The instance you want to manage is defined by the `workdir`. Workdir - is a directory where all the data necessary for running the instance is stored. By default, it is `/opt/supervisely/`. This also means that you can run several instances on the same machine by using different workdirs.

When you run a CLI command, the CLI tries to find the workdir in the following order. You can also provide a workdir explicitly using the `-w`/`--workdir` option:
1. current directory
2. default directory `/opt/supervisely/`
3. parents of current directory

{% hint style="info" %}
directory considered as workdir if it contains `.supervisely/config.json` file
{% endhint %}

The daemon uses a configuration files to store instance settings. The configuration files are located at `.supervisely` subdirectory in the workdir and are created automatically when you run the `init` command. What those files are:
1. `config.json` - instance settings
2. `vars.yml` - variables for instance
3. `docker-compose.yml` - docker-compose configuration for instance


# Commands

## Init

To set up the instance, use the "init" command. This command installs daemon as a system service, creates a configuration file for the instance and upgrades the instance if needed.

```bash
supervisely [OPTIONS] init
```

command options:
- `-l` / `--license` - path to license file or license string. Is not necessary if config file already contains license.
- `-w` / `--workdir` - path to workdir. See # Usage.
- `--show-daemon-logs` - add this flag to include daemon logs in the output. Useful for debugging.
- `--log-file` - path to log file. If not specified, logs will be written to stdout.


## Set license

To set new license, use the "set-license" command.


```bash
supervisely set-license [OPTIONS] [license string or path to license file]
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.


## Update

To update configuration file, use the "update" command. This fetches a new configuration from the web and updates the configuration file.

```bash
supervisely [OPTIONS] update
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.


## Backup

To create a backup of configuration and data for the instance, use the "backup" command. This command creates a backup archive and stores it in the workdir.

```bash
supervisely [OPTIONS] backup
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.


## Upgrade

Upgrade your instance by using the "upgrade" command. This command fetches a new configuration from the web, downloads the latest Docker images required to run the instance, and restarts the instance.

```bash
supervisely [OPTIONS] upgrade
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.
- `--skip-backup` - add this flag to skip backup creation before upgrade.

## Login

To log in to Docker registry, use the "login" command. Credentials are stored in the config.

```bash
supervisely [OPTIONS] login
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.


## Uninstall

To uninstall the instance, use the "uninstall" command. This command stops the instance containers and deletes data.

```bash
supervisely [OPTIONS] uninstall
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.

## SQL

To run SQL query on the instance database, use the "sql" command. This command runs the query and prints the result.

```bash
supervisely [OPTIONS] sql [query]
```

command options:
- `-w` / `--workdir` - path to workdir. See # Usage.
