# Heroku Django Instructions

## Prerequisutes
Make sure you have python, pip, virtualenv and virtualenvwrapper installed

**Some Links**

- [python](http://www.python.org/getit/)
- [setuptools/pip](https://pypi.python.org/pypi/setuptools)
- [virtualenv](https://pypi.python.org/pypi/virtualenv)
- [virtualenvwrapper](http://virtualenvwrapper.readthedocs.org/en/latest/install.html)

## Creating a django app in a virtual enviroment

If you are using `virtualenv` and not `virtualenvwrapper`:
```bash
mkdir appName
cd appName
virtualenv venv --distribute
source venv/bin/activate
```

If you are using `virtualenvwrapper` and not `virtualenv`
(make sure you have followed the [guide](http://virtualenvwrapper.readthedocs.org/en/latest/install.html) to configure virtualenvwrapper correctly):
```bash
mkproject appName
```

Then, once you are in the virtualenv:
```bash
pip install django-toolbelt #this installs django, psycopg2, gunicorn, dj-database-url, dj-static, static
django-admin.py startproject appName .
```

## Creating files for heroku
```bash
echo "web: gunicorn appName.wsgi" > Procfile
pip freeze > requirements.txt
```

## Test
```bash
foreman start
```

## Database

### Creating the Database

```bash
psql template1
CREATE USER username WITH PASSWORD 'password';
CREATE DATABASE appName;
GRANT ALL PRIVILEGES ON DATABASE appName to username;
\q
```

### Configuring Django to access the Database
Add this to the **end** of `settings.py` without editting the `DATABASES` variable at the start of the file
```python
# Parse database configuration from $DATABASE_URL
import dj_database_url
DATABASES['default'] =  dj_database_url.config()
```

Since you are here, configure static files too. Add this to the end of the file:

```python
# Honor the 'X-Forwarded-Proto' header for request.is_secure()
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# Allow all host headers
ALLOWED_HOSTS = ['*']

# Static asset configuration
import os
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
STATIC_ROOT = 'staticfiles'
STATIC_URL = '/static/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```

And change the `get_wsgi_application()` line in `wsgi.py` to these lines:
```python
from django.core.wsgi import get_wsgi_application
from dj_static import Cling
application = Cling(get_wsgi_application())
```

### Create env variable
Temporary solution:
```bash
export DATABASE_URL=postgresql://username:password@localhost:5432/appName
```

Permanent Solution (virtualenvwrapper only)

- Make sure virtualenv is activated (using `workon appName`) 
- Add the same line the file `$VIRTUAL_ENV/bin/postactivate`


### Test

```bash
python manage.py syncdb
```
should work correctly

### Add to git, create app and push
- Create an appropriate `.gitignore`
> If you are using `vitrualenv` and not `virtualenvwrapper` make sure you add `venv` or whatever you named your `virtualenv` folder to `.gitignore`

`.gitignore`
```
*.pyc
venv
staticfiles
```

Commands:
```bash
git init
git add .
git commit -m "Initial Commit"
heroku create
git push heroku master
heroku run python manage.py syncdb
heroku open
```

Everything should work correctly.  
Enable the admin app to test further.
