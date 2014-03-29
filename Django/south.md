# South

[Documentation](http://south.readthedocs.org/en/latest/index.html)
[Cheat Sheet](http://south.readthedocs.org/en/latest/commands.html)

## About
- Helps with schemamigrations
- If something changes in your models, helps change your database too
- Keeps track of what has changed so you can revert to older schemas

## Installation
- In your virtualenv, use `pip install south`
- Add `south` to `INSTALLED_APPS` in `settings.py`
- Run `manage.py syncdb` to let south create its database tables

## How to use?

### First Use
South runs on a per app basis. There are two methods of running south on first-use and they are based on what you have already done before

#### Method 1: New App (No models in DB)
By new, I mean the app that you are running the migrations for does not have its models' tables in the db. That is, you have never run `syncdb` on that app before.

If this is the case, run:
```bash
# create migration using:
python manage.py myapp --inital
# run migration using:
python manage.py migrate myapp
```

The first command creates a package in your `myapp` called `migrations` with the file `0001_inital.py`. This is the migration file that is run to change your database.

The second command runs that file to make the actual changes.

Now, this file is going to be committed to your VCS and all the people in your team need to:
1. Run `manage.py syncdb` to get `south`'s tables into the system
2. Run the migration file using the command `python manage.py migrate myapp`. 

They do **not** need to run `python manage.py myapp --inital` becuase the `0001_inital.py` has already been created.

#### Method 2: Existing App (Models already in DB)

If you have already run `syncdb` on an app before and it has its models in the database, you will have to run a command called:

```bash
python manage.py convert_to_south myapp
```

This command is a shortcut command for the following two commands:
```bash
# create migration using:
python manage.py myapp --inital
# "fake" run migration using:
python manage.py migrate myapp 0001 --fake
```

You already know what the first command does.

The second command "fake" runs the migration. This means that `south` records in its own database tables that the migration has been run without actually running it. The migration does not have to run becuase `syncdb` has already done the job. `south` just need to get on to the same page as your app's database table is right now, so that's why this "fake" run is necessary.

Now, when you commit this to your VCS, your teammates need to run the fake migration too using the command `python manage.py migrate myapp 0001 --fake`.

### Future Uses
Once this (slightly) complicated first use is over, `south` is very easy to use.

Each time you change your models and want to push the changes to the database, run the commands: 

```bash
# to create a new migration file:
python manage.py schemamigration myapp --auto
# to run the migration files:
python manage.py migrate myapp
```

Each of your teammates will have to run `python manage.py migrate myapp` on their machines to be able to get access to the new database tables

Before you run `migrate myapp`, you can `update` your migration if you end up making more changes to your models by runnung:

```bash
python manage.py schemamigration myapp --auto --update`
```

### Specifing Defaults
Sometimes, you will need to specify a default if you add a new column to the database. Example:

In `models.py` you add a field:
```python
is_member = models.BooleanField()
```

`south` is not going to know if you want it to be `True` or `False` for all the existing rows in the db. So you will see:

```bash
python manage.py schemamigration myapp --auto
 ? The field 'modelName.is_memeber' does not have a default specified, yet is NOT NULL.
 ? Since you are adding or removing this field, you MUST specify a default
 ? value to use for existing rows. Would you like to:
 ?  1. Quit now, and add a default to the field in models.py
 ?  2. Specify a one-off value to use for existing columns now
 ? Please select a choice:
```

You can enter 1 and specify a default by:

In `models.py`:
```python
is_member = models.BooleanField(default=True)
```

Or if you want to allow it to be blank:
```python
is_member = models.BooleanField(null=True)
```

Else, you can enter 2 and specify a default by:

Typing `2` (hit enter) `True` (hit enter) in the interactive prompt

#### Data Migrations
If you want to make a complex calculation to specify the value in the new column, you will have to run a data_migration using the forwards and the backwards functions. This is a little complex so read the documentation [here](http://south.readthedocs.org/en/latest/tutorial/part3.html#data-migrations) carefully.

In a nutshell, you have to write your python code that you want to run  to populate the new column by creating a template migration using the command:

```bash
python manage.py datamigration myapp name_of_migration
```

And then you have to add code to make and reverse the changes in the `forwards()` and `backwards()` methods

### Advanced Stuff
If two people on different machines create migrations, on the same parts of the model, you can have problems that might be solved with

```bash
python manage.py migrate --merge
```

If not, you will have to edit the migration files seperately.

Or you might even have to write your own migration using the template created by the command:

```bash
python manage.py schemamigration --empty myapp name_of_migration
```

Read [this](http://south.readthedocs.org/en/latest/tutorial/part5.html#team-workflow) for more information

