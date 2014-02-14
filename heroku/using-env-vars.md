# Using Environment Variables on PaaSs
## Setting Variables
### Heroku
[Heroku Guide on Environment Variables](https://devcenter.heroku.com/articles/config-vars)

Set `config-vars` to:

- Determine environment type (Development, Testing, Production)
- Store secrets (S3 Key, Secret Key used in hashing, Beta Code, etc)

```
heroku config:set ENV_TYPE=test     // To set a (new) var
heroku config                       // To get all vars
heroku config:get ENV_TYPE          // To get ENV_TYPE var
heroku config:unset ENV_TYPE        // To delete ENV_TYPE var
```

If you want to set vars on your laptops:  
Create a file called `.env` in your project directory 
```
ENV_TYPE=dev
S3_KEY=mykey
S3_SECRET=mysecret
```
Next, run the program using `foreman start`

### Pushing and Pulling Config-Vars
You can also pull all the `config-vars` from heroku to your local directory's `.env` file using the command `heroku config:pull --overwrite --interactive`

Similarly, you can push from `.env` to the server using `heroku config:push`

- Next section explains how to access these vars in code.

### Pagodabox
[Guide](http://help.pagodabox.com/customer/portal/articles/175470)  
Add variables to the Boxfile or using the Dashboard

### CloudBees
[Guide](http://developer.cloudbees.com/bin/view/RUN/Configuration+Parameters)

```
bees config:set env_type=test
bees config:list
bees config:update env_type=prod
bees config:delete env_type
```

## Accessing Variables

`ENV_VARIABLE` is the name of the enviroment variable you want to access

### Javascript
```js
process.env['ENV_VARIABLE']
```

### Python
```python
import os
os.getenv('ENV_VARIABLE', default_value)
```

### Ruby
```ruby
ENV['ENV_VARIABLE']
```

### PHP
```php
getenv('ENV_VARIABLE')
```

### Java
```java
System.getenv('ENV_VARIABLE');
// OR
System.getProperty('ENV_VARIABLE');
```

### Groovy
```groovy
env['ENV_VARIABLE'];
```

## Using Variables in Code
Pick properties in your settings based on the environment dynamically.

Pseudo-code:
```
environment_type = env('ENV_TYPE')
if (environment_type == "DEV")
    DEBUG = false;
    LOGGING = 'VERBOSE';
else if (environment_type == "TEST")
    DEBUG = env('DEBUG', false);
    LOGGING = 'VERBOSE'
else if (environment_type == "PROD")
    DEBUG = false;
    LOGGING = 'WARN'
else
    throw SOME_ERROR
```

## Dealing with Dependencies

Have different files that install dependencies on your dev vs. test + prod servers. You might include some debugging tools in your dev file, but those should not be installed on your testing or prod environments.  

Example:

- Django: `django-debug-toolbar` in `dev_requirements.txt` but not in `requirements.txt`
- Node.js: `nodemon` in `devDependencies` and not in `dependencies` inside the `package.json` file
