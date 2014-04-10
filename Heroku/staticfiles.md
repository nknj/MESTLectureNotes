# Static Files on Heroku
## Static Files Overview
### Settings
- `STATIC_ROOT`: Where to collect to
- `STATICFILE_DIRS`: Where to collect from
- `STATIC_URL`: What URL to serve under

### Fundamental Idea
1. Move files from `STATICFILE_DIRS` to `STATIC_ROOT`
2. Configure server to serve static files at `STATIC_URL`

Step 1 is achieved my running `manage.py collecstatic`  
Step 2 depends on the server

### `STATIC_ROOT` on a different machine 

If `STATIC_ROOT` is not on current server and on a different "static file server" machine or a CDN, you can write your custom file storage backend to enable `manage.py collectstatic` to transfer files to that external server.

Use the `STATICFILES_STORAGE` setting to set a new custom file storage backend
```python
STATICFILES_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
```

### What about `DEBUG = False`?
- `manage.py runserver` serves the static files when `DEBUG = True` but not when `DEBUG = False`
- `guinicorn` never serves static files (not when `DEBUG = True` or `DEBUG = False`)
    + We use `gunicorn` on heroku, this is why your static files "disappear sometimes"

#### Why?
Serving static files is a very simple job for a server. Dynamic file servers like `gunicorn` are too 'heavy' and 'slow' since they need to server dynamic files. So serving static files using dynamic servers is not recommended and disabled by default.  

## Heroku

### Bad Options:
1. Using `manage.py runserver` with anything. 
    - With `DEBUG = True` or
    - With `DEBUG = False` and 'weird' `URLPatterns` in `urls.py` or
    - With `dj-static`

2. Using `gunicorn` and 'weird' `URLPatterns` in `urls.py`

### Good Option:
Using `gunicorn` and `dj-static`

wsgi.py
```python
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")

from django.core.wsgi import get_wsgi_application
from dj_static import Cling

application = Cling(get_wsgi_application())
```

#### How does `dj-static` work?
It wraps around your entire django application and looks at the URL coming in.

If the url matches `STATIC_URL`, it serves the file using a python app called `static` that makes serving static files on dynamic server acceptable. If not, it just passes control to the django app.

Static files are still served from the `STATIC_ROOT` folder.

Works in:
- `DEBUG = True` from `STATICFILE_DIRS` only and when
- `DEBUG = False` from `STATIC_ROOT` only

### Best Option:
Using AWS and `django-storages`

### Summary:
`settings.py`:
```python
DEBUG = False
STATIC_URL = '/static/'
STATICFILE_DIRS = (os.path.join(BASE_DIR, 'static'), )
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
```

`wsgi.py`:
```python
import os
from django.core.wsgi import get_wsgi_application
from dj_static import Cling

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
application = Cling(get_wsgi_application())
```

Push to heroku using git, it will automatically run `manage.py collectstatic`
