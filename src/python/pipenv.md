## Table of contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Run an environment](#run-an-environment)
  - [New environment](#new-environment)
      - [Default virtualenv path](#default-virtualenv-path)
      - [Custom virtualenv path](#custom-virtualenv-path)
  - [Created environment](#created-environment)
      - [Show environments](#show-environments)
      - [Activate created virtual environment](#activate-created-virtual-environment)
- [Pipfile.lock: environment and packages](#pipfile.lock-environment-and-packages)
  - [Create a Pipfile.lock](#create-a-pipfile.lock)
      - [Install packages](#install-packages)
      - [Uninstall packages](#uninstall-packages)
      - [Lock environment](#lock-environment)
- [Exit virtual environment](#exit-virtual-environment)
  - [Error already activated](#error-already-activated)
- [Other commands](#other-commands)
- [Resources](#resources)

## Introduction

Pipenv is a command line tool to manage python virtual environments and pip installations.

## Installation

```bash
pip install pipenv
```

## Run an environment

### New environment

#### Default virtualenv path

With the following command, the desired python version is set too.

```bash
pipenv shell --python 3.7.2
```

Note. Its important to save the path of the environment in order to check later all available environments. Example:

```bash
✔ Successfully created virtual environment!                                                                    
Virtualenv location: /home/nonroot/.local/share/virtualenvs/nonroot-LcccbqV7                                   
Creating a Pipfile for this project…                                                                           
Launching subshell in virtual environment…                                                                     
 . /home/nonroot/.local/share/virtualenvs/nonroot-LcccbqV7/bin/activate                                        
nonroot@c3942b253696:~$  . /home/nonroot/.local/share/virtualenvs/nonroot-LcccbqV7/bin/activate
```

The location of the virtual environment can be checked too with the following command before exit the environment:

```bash
pipenv --venv
```

#### Custom virtualenv path

You can set the path of the venv:

```bash
export PIPENV_VENV_IN_PROJECT=1
pipenv shell --python 3.7.2
```

The venv will be created at the current location inside a .venv folder. Example: /home/nonroot/Documents/project/.venv/bin/activate

Remove the option:

```bash
unset PIPENV_VENV_IN_PROJECT
```

### Created environment

You can work with a created virutal environment.

#### Show environments

The location of the environments is showed when the first one was created. Example:

```bash
ls /home/nonroot/.local/share/virtualenvs/
```

#### Activate created virtual environment

If you are in the same path as your Pipfile and Pipfile.lock files:

```bash
pipenv shell
```

If you are in a different path, the path to the environment must be provided. Example:

```bash
source /home/nonroot/.local/share/virtualenvs/nonroot-LcccbqV7/bin/activate
```

## Pipfile.lock: environment and packages

### Create a Pipfile.lock

The Pipfile.lock allows you to save the state of the environment and use later in other environment.

You have to install or uninstall packages and the lock the Pifile.lock file.

#### Install packages

After start a pipenv shell:

```bash
pipenv install flask==0.12.1
```

Install from requirements files:

```bash
pipenv install -r requirements.txt
```

##### Install from pipenv

Omitting the dev section: 

```bash
pipenv install
```

Including the dev section:

```bash
pipenv install --dev
```

If the Pipfile.lock exists, it will be used for the installation instead Pipfile. 

You can omit the Pipfile or Pipfile.lock when installing packages:

- To omit the Pipfile.lock file: add `--skip-lock` at the end of the previous commands (`pipenv lock` command won't be exectued).
- To omit the Pipfile file: add `--ignore-pipfile` at the end of the previous commands.

#### Uninstall packages

Only one package:

```bash
pipenv uninstall numpy
```

All packages:

```bash
pipenv uninstall --all
```

#### Lock environment

After install the packages (or uninstall them), we will create or update the Pipfile.lock file with all the information of the virtual environment.

```bash
pipenv lock
```

The Pipfile.lock file must not be modified, to do that, use the pipenv shell.

## Exit virtual environment

```bash
exit
```

If you are in a docker container, use the following option to avoid close the container:

```bash
CTRL-d
```

### Error already activated

If you exit the environment using `deactivate` instead `exit`, you will have this error when you activate it again with `pipenv shell`:

```bash
Shell for UNKNOWN_VIRTUAL_ENVIRONMENT already activated.
No action taken to avoid nested environments.
```

The solution is simply to run the `exit` command (even though you are currently outside the vitual environment) and you will be able to activate the environment again:

```bash
exit
pipenv shell
```

Resource: <https://stackoverflow.com/questions/49944871/deactivate-a-pipenv-environment>.

## Other commands

Location of the virtual environment:

```bash
pipenv --venv
```

## Resources

- Activate and deactivate a virtualtual environment

  <https://github.com/pypa/pipenv/issues/84>

- Custom virtualenv path

  <https://github.com/pypa/pipenv/issues/1049>

- Documentation

  <https://pipenv-es.readthedocs.io/es/latest/>

- Pipenv install

  <https://pipenv.kennethreitz.org/en/latest/basics/#pipenv-install>

- Tutorial

  <https://realpython.com/pipenv-guide/#reader-comments>
