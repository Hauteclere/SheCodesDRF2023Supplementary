# `virtualenv` Walkthrough

`virtualenv` is a virtual environment manager for Python. It helps you set up individual sandboxes for each of your Python projects so that you can install packages specific to each codebase, without cluttering up your computer's global environment or running into version conflicts.

There are other similar tools that do the same thing - in the last unit you used a package called `venv` for the same task. `virtualenv` and `venv` are very similar.

- [`virtualenv` Walkthrough](#virtualenv-walkthrough)
  - [A Note On Running Python Modules](#a-note-on-running-python-modules)
  - [Installation](#installation)
    - [Updating Pip](#updating-pip)
    - [Installing/Upgrading `virtualenv`](#installingupgrading-virtualenv)
  - [Environment Setup](#environment-setup)
    - [Create The Environment](#create-the-environment)
    - [Make Sure That `venv/` Is In Your `.gitignore`](#make-sure-that-venv-is-in-your-gitignore)
    - [Create A Requirements File](#create-a-requirements-file)
  - [Using The Environment](#using-the-environment)
    - [Turn It On](#turn-it-on)
      - [On Windows:](#on-windows)
      - [On Mac/Linux:](#on-maclinux)
    - [Install Libraries](#install-libraries)
    - [Deactivate the Environment](#deactivate-the-environment)


## A Note On Running Python Modules
The standard way to do run a Python module in the terminal is to invoke the `python3` interpreter by name, use the `-m` flag to indicate a module, and then name the installed module. That looks like this:

```Bash
python3 -m <module_name_here>
```
Some modules can be "*added to the PATH*" when they're installed. `pip` and `virtualenv` are examples of these. When a module is added to the PATH, it is available to be invoked by name without the python interpreter. So, if `pip` is on my PATH, I can just type:

```Bash
pip install <some_module_name>
```
Instead of the slightly more complicated:
```Bash
python3 -m pip install <some_module_name>
```
These commands do the same thing, **unless** the module in question isn't on the PATH, or there are multiple installations of Python on the computer. In those edge cases, things can get weird.

In this walkthrough I'll always explicitly invoke the Python interpreter, but if you see other examples online where people leave that bit off, now you'll know why. If the shorter commands work for you, go for it!

## Installation
Before you can use `virtualenv` to manage virtual environments, you need to make sure you have it installed. Let's do that now - you should only have to do this once, unless you make change the machine you're working on.

### Updating Pip
First let's check that `pip` is up to date. Remember, `pip` is the library manager we use to install Python libraries; we'll need it to get `virtualenv`, and we'll also need it to install other packages in each environment.

Open your terminal. (On Windows, you should select `run as administrator`.) Then run the following command:
```Bash
python3 -m pip install --upgrade pip
```

- If you get an error saying Python is not installed, you can find guides for installing it [here](https://wsvincent.com/install-python/)
- If you get an error saying `pip` is not installed, take a look [here](https://pip.pypa.io/en/stable/installation/) at how to install it.

### Installing/Upgrading `virtualenv`
Next try running the command 
```Bash
python3 -m pip install --upgrade virtualenv
```

This should set you up with everything you need to manage virtual environments with `virtualenv`

## Environment Setup
When you first begin a project, you'll need to create a virtual environment to manage the packages for that project. This is something you'll only need to do once for each project.

Navigate to the root folder of the project in the terminal. If you've cloned a repo, look for a file called `requirements.txt` - that's where you should create the virtual environment. If you can't find a requirements file or you're starting a project from scratch, the best place is the outer directory you've created to store all your project files - the same place you initialise your `git` repo.

### Create The Environment
Setting up a virtual environment takes just one command: 

```Bash
python3 -m virtualenv venv
```
Here we are supplying the name `venv` for the environment - you could technically pick any name you like, but `venv` is traditional.

### Make Sure That `venv/` Is In Your `.gitignore`
You should make sure that the `venv/` folder that last command created is listed in your `.gitignore` file, because it shouldn't be included in your version control.

### Create A Requirements File
As projects evolve, they generally accumulate more libraries that need to be included to make them run.

To keep track of the dependencies that have been rolled into the project, we'll use a file called `requirements.txt`. It will contain a list of libraries that have been installed, and can be used to install all the dependencies all at once when someone clones the project repo down to their machine.

Create this file now. You should call it `requirements.txt`, since that's the traditional name.

## Using The Environment
Each time you work on the project, you'll use the environment to manage your installed packages.

### Turn It On
Every time you open the terminal to work on your project, you should make sure that the environment is activated before beginning to code. To do this, navigate to the root folder (the one containing the `venv/` file) and run the following command:

#### On Windows:
```CMD
./venv/Scripts/activate
```

#### On Mac/Linux:
```Bash
source ./venv/bin/activate
```

In most terminal shells, you should notice that the shell prompt changes slightly to indicate that you are working inside the virtual environment. Sometimes this looks like the text `(venv)` being added to the start of the prompt.

### Install Libraries
To install libraries in your environment, just use pip as normal. Ex:

```Bash
python3 -m pip install --upgrade <your_package_name_here>
```
Once the library is installed you should record it in your `requirements.txt`. As long as you are in the root folder, you can do this by running:

```Bash
python3 -m pip freeze > requirements.txt
```
Libraries that you install while the environment is active will only be available while the environment is active. If you forget to reactivate the environment the next time you come back to the project, you'll get errors!

### Deactivate the Environment
Once you're done working on the project you can normally just close the terminal. But if you want to swap over to another project or you have some other reason to turn off your environment, you can just type the following single word in the terminal:

```Bash
deactivate
```
