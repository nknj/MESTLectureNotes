# User Management in Django
`django.contrib.auth`

## About
- `User`, `Permission` and `Group` models
- Helper Functions
    + Password Hashing
    + Login
    + Logout
- Forms
    + User Creation (Sign-Up)
    + Login

The User model can be customized, but we are going to look at the default implementation first.

## User (Default: Non-Cutomized)
Lets look at the [API](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#user)

### Required Fields:
- `username` - Required
- `password` - Required

### Create User
```python
user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
user.last_name = 'Lennon'
# user.password = "newpassword" WRONG!!!
user.setPassword('newpassword')
user.save()
```

- Passwords are not stored in the "raw" form, they are hashed and stored in this format: `<algorithm>$<iterations>$<salt>$<hash>`
- `algorithm` is determined by the `PASSWORD_HASHERS` setting.

### Create Superuser

Usually uses the command line:
```bash
python manage.py createsuperuser --username=joe --email=joe@example.com
# Interactively enter password
```

Calls the `User.objects.create_superuser` method at the back  
Same as `create_user` but `is_staff` and `is_superuser` is set to `True`

### Functions

#### Authenticate

- Returns `User` object is correct, else returns `None`
- Doesn't log the person in yet!

```python
user = authenticate(username="john", password="secret")
if user is not None:
    # authenticated
    if user.is_active:
        # user has not be deleted/disabled
        # login the user here
        # redirect to success page
    else:
        # username and password valid, but user is deleted/disabled
else:
    # not authenticated
```

#### Login

```python
login(request, user)
```
Creates a session on that request

#### Logout

```python
logout(request)
```
Destroys the session on that request

#### Accessing Logged-In User
In Python Code:
```python
current_user = request.user
current_user.username
```

In template:
```html
{{ user.username }}
```

#### Checking for authentication

##### Uncool way:
```python
def my_view(request):
    if request.user.is_authenticated():
        # do stuff
    else:
        # redirect to login page
```


##### Cool way (using decorators):  
Decorators are functions that take a function as an argument, modify that function and return that modified function as a result
```python
@login_required
def my_view(request):
    # do stuff
```

- If logged in, stuff will be done  
- Else redirect to `settings.LOGIN_URL + ?/next=/url/you/tried/to/access`  
- So make sure your login view handles the `next` GET attribute properly
    + If you write your own one

##### Templates
```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}. Thanks for logging in.</p>
{% else %}
    <p>Welcome, new user. Please log in.</p>
{% endif %}
```

### Signals
- Functions (handlers) to be called after an event takes place
- Django signals are synchronous. The handlers are executed as soon as the signal is fired, and control returns only when all appropriate handlers have finished.
- Useful for inter-app communication

- `user_logged_in()`
- `user_logged_out()`
- `user_login_failed()`

How to use:  
signals.py
```python
@receiver(user_logged_in)
def after_user_logged_in(sender, user, request, **kwargs):
      # do stuff

@receiver(user_logged_out)
def after_user_logged_out(sender, user, request, **kwargs):
      # do stuff

@receiver(user_login_failed)
def after_user_login_failed(sender, credentials, **kwargs):
      # do stuff
```

Then, `import signals` in `__init__.py` of the app 

### Views

- `login` - Login a user
    + `AuthenticationForm`
- `logout` - Lougout a user
- `logout_then_login` - Logout a user, then redirect to Login page
- `password_change` - Change password of user
    + `PasswordChangeForm`
- `password_change_done` - Password change successful
- `password_reset` - Reset Password using email
    + `PasswordResetForm`
- `password_reset_done` - Reset Password email sent
- `password_reset_confirm` - Reset Password form
    + `SetPasswordForm`
- `password_reset_complete` - Reset Password successful changed

Not a view, but there is a `UserCreationForm` that has `username`, `password1` and `password2` by default. It can be extended to have more fields.

#### Example: Login View

##### Step 1: Set LOGIN_URL in settings
```python
LOGIN_URL = '/users/login/'
```

##### Step 2: Set URL conf with built-in view
```python
(r'^users/login/$', 'django.contrib.auth.views.login'),
```

##### Step 3: Build Template
Default Template Name: `registration/login.html`  

If you want to call it some thing else, URL Conf:
```python
(r'^users/login/$', 'django.contrib.auth.views.login', {`template_name`: `users/login.html`}),
```

Write your template:
```html
{% extends "base.html" %}

{% block content %}

{% if form.errors %}
<p>Your username and password didn't match. Please try again.</p>
{% endif %}

<form method="post" action="{% url 'django.contrib.auth.views.login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>

<input type="submit" value="login" />
<input type="hidden" name="next" value="{{ next }}" />
</form>

{% endblock %}
```

Context available:
- `form`: `AuthenticationForm`
- `next`: Next URL to go to if the login is successful

You get login functionality without writing a single line of code in views

### Authentication Backend

- `AUTHENTICATION_BACKENDS` in settings.py
    + Default: `('django.contrib.auth.backends.ModelBackend',)`
    + You can create your own one
    + Basically describes how the `authenticate()` method works
- You can have multiple backends in the tuple and Django will try to authenticate against each one

## User (Customized)

The default model is very useful but:

- What if you want to add more business logic in the form on new methods to the User Model?
- What if you want to track more fields that the default class gives?
- What if you want to login with the email and not the username?

If you answered yes to any of these, you need to customize the User model by either **extending** it (Option 1 and 2) or by **substituting** it (Option 3)

### Options:
1. If you want to add new methods ONLY and are fine with everything else:
    - Create a **Proxy Model** based on `User`
2. If you want to add new fields
    - Create a **Profile Model** by using a one-on-one relationship with `User`
3. If you want to customize more than methods or fields (example: change login field from username to email)
    - Create your own **User Model**


### Option 3: Substituting Your Own User Model
#### In `settings.py`:

```python
AUTH_USER_MODEL = 'appName.UserModelName'
```

#### In `models.py`:

Requirements:
1. Integer pk
2. Single unique identifier field
3. A way to address they user in `short` and `long` form
4. A email field called `email` (only if you want the default `PasswordResetForm` to work)

Recommendations:
1. `USERNAME_FIELD` must be defined (must be unique, used for login)
2. `REQUIRED_FIELDS` must be defined (no need to re-include `USERNAME_FIELD`)
3. `is_active` must be implement
4. `get_full_name()` must be implemented
5. `get_short_name()` must be implemented

Best way is to extend `AbstractBaseUser` and `BaseUserManager` and look at the example of `AbstractUser` and `UserManager`

Make sure you remove the `abstract = True` line becuase this is a real model now and not an abstract model anymore:

```python
class MyUser(AbstractBaseUser, PermissionsMixin):
    #...
    class Meta:
        #...
        # Remove this line
        abstract = True
```

Make edits by changing/adding/removing fields 

#### In `forms.py`:

- You might have to create a new `UserCreationForm`, a new `UserChangeForm` and a `MyUserAdmin` Admin Interface based on the new fields (names don't matter)
- All other forms will work
