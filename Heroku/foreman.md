# Foreman
## `foreman start`
1. Always use `foreman start` to run your server
    - This replicates how heorku's runs your app by using `Procfile` and `.env`
    `Procfile`:
    ```
    web: gunicorn myproject.wsgi
    ```

    `.env` file:
    ```
    DATABASE_URL=postgres://taildbadmin:tailDBadmin@localhost:5432/tail
    DEBUG=True
    TEMPLATE_DEBUG=True
    SECRET_KEY=jiifwfqd1@1o$sxy%3fiho78bsn*%#9#jr7=is1bk$20n0hdzz
    ```

    `settings.py`:
    ```python
    DEBUG = os.environ.get('DEBUG') == 'True'
    TEMPLATE_DEBUG = os.environ.get('TEMPLATE_DEBUG') == 'True'
    SECRET_KEY = os.environ.get('SECRET_KEY')
    DATABASES = {
        'default': dj_database_url.config()
    }
    ```

2. On heroku, `.env` is not used. Environment variables are set and unset using these commands

    ```
    heroku config:set ENV_NAME=test     // To set a (new) var
    heroku config                       // To get all vars
    heroku config:get ENV_NAME          // To get ENV_NAME var
    heroku config:unset ENV_NAME        // To delete ENV_NAME var
    ```

3. To run all other commands use `foreman run`
    + `foreman run ./manage.py shell` for example
    + This will have access to all the env vars in `.env`