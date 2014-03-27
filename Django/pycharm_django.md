# Pycharm

## Details
- Jetbrains PyCharm Version `3.1.1`
- Python version `2.7.5`

## Required Skimming:
- [Features](http://www.jetbrains.com/pycharm/features/)
- [IDE Quickstart](http://www.jetbrains.com/pycharm/quickstart/)
- [Django Quickstart](http://www.jetbrains.com/pycharm/quickstart/django_guide.html)
- Keyboard Shortcuts for [Windows](http://www.jetbrains.com/pycharm/docs/PyCharm_ReferenceCard.pdf) and [OSX](http://www.jetbrains.com/pycharm/docs/PyCharm_ReferenceCard_Mac.pdf)

### Useful Shortcuts:
    + `Cmd + Shift + A` - Find any Action
    + `Option + Enter` - Show intention actions and quick-fixes
    + `Ctrl + Space` - Basic code completion
    + `Cmd-Shift-O` - Open File
    + `Cmd-O` - Open Class
    + `Cmd-Ctrl-O` - Open Symbol
    + Double `Shift` - Search Symbols

## Polls Tutorial
We are following the official [Django Tutorial](https://docs.djangoproject.com/en/1.6/intro/tutorial01/) on PyCharm

### Creating a new project
- Under `Quick Start` click on `Create New Project`
    - `Project Name`: `mysite`
    - `Project Type`: `django`
    - `Interpreter`: Click on `...` followed by python logo with a V on it
        - If you hover on it, it says: `Create Virtual Environment`
        - `name`: `mysite`
        - `location`: Set an appropriate location for all your `virtualenv`s
            - I choose `~/.virtualenvs/`
        - `OK`
        - Once the `virtualenv` is created, click on `Install` and select `Django`, make sure the version is `1.6.2` and install
    - `Application Name`: `polls`
    - Select `Enable Django admin`
    - `OK`

### Follow the tutorials
- Read the `pep8` warnings when you write your code and correct them accordingly
- You can do a `python manage.py runserver` by clicking the Green Arrow on the top toolbar
    + Click on the drop-down before the green arrow and then on `Edit Configurations` to make changes to the runserver command (e.g. change the port, ip, etc)
        * You can select `Run browser` and `Start Javascript debugger...` if you find this more convinient
    + When the command is run, a console pops up
        * Use the icons on the left of the console to terminate, restart or stop the process
- You can run `manage.py` commands by going to `Tools` > `Run manage.py Task...` and follow the interactive dialog
    + This includes `syncdb`, `sql`, `test` etc...
- You can access the Django shell (`python manage.py shell`) by going to `Tools` > `Run Django Console...`
- Learn the shortcuts to `Open File` and `Open Class` so that you can easily access things without using your mouse
- Enable auto-import for python by going to `Settings` > `Editor` and selecting `Show import popup`
- You may get `typo` warnings from PyCharm, I would suggest that you **don't** disable them and just add unusual words to the dictionary

