# Token Authentication with Django and AngularJS
For the past month, I have been building all my new apps in a fully decoupled fashion. I build my backend using `Django` which acts like a RESTful API server and I build my frontend using `AngularJS` which consumes that RESTful API. The biggest hurdle I faced while learning how to build this was how to tackle authentication.

When you build a RESTful service, it is recommended that you avoid using session cookies. It is easier to deal with API keys or tokens on clients than it is to deal with cookies. Anyone who has build an `Android` or an `iOS` app should agree with this. Moving away from sessions can be a little complicated for someone who has never built anything without sessions before. This post helps with that and talks about how to use tokens in replacement of sessions cookies to authenticate users are preserve state.  

This post uses a bare-bones and very basic setup that doesn't use any third-party apps. I recommend this setup for learning purposes because it doesn't abstract away concepts like that third party apps do. You can learn more by implementing everything yourself. This setup is **not** recommended in a production environment. I would highly recommend using battle-tested third party apps to build your production apps. I will recommend a few of these third party apps as I move along with this post.

This post assumes some basic `Django` and `AngularJS` knowledge

## Startup

First, we to bootstrap our backend and a frontend component. This requires you to have `node`, `yeoman` and `generator-angular` installed.

```
mkdir django-angular-auth && cd $_
mkvirtualenv django-angular-auth
pip install django # or localpip django
django-admin.py startproject backend
mkdir frontend && cd $_
yo angular

```

Now we have our `Angular` and `Django` app up

**FYI:** What's `localpip`? See my previous blog post [here](http://blog.nknj.me/python-guide-to-hacking-on-an-airplane).

## Backend

Lets open the project in our favorite editor, I am using `PyCharm`
```
cd backend
pycharm .
```

[Pycharm specific]: Before we begin getting our hands dirty, we need to set the right python interpretor so open up the `Preferences > Project Interpreter > Python Interpreters >`. Click the `+` sign and select the `django-angular-auth` interpretor you just created.

### Lets get down to business.

For this project lets use `django.contrib.auth`'s default implementation for the `User` class. In addition to what we get from Django, we need to store the users tokens that will be used for authentication. Lets create an `auth` app to house this model and logic.

On CLI:
```
./manage.py startapp auth
```
(If `./manage.py` doesn't work, it doesn't have permissions to execute. Add them by running `chmod +x manage.py`)

Remember to add the new `auth` app to your installed apps. `INSTALLED_APPS += ('auth',)`

In `models.py`:
```python
import binascii
import os
from django.contrib.auth.models import User
from django.db import models


class Token(models.Model):
    user = models.ForeignKey(User)
    token = models.CharField(max_length=40, primary_key=True)
    created = models.DateTimeField(auto_now_add=True)

    def save(self, *args, **kwargs):
        if not self.token:
            self.token = self.generate_token()
        return super(Token, self).save(*args, **kwargs)

    def generate_token(self):
        return binascii.hexlify(os.urandom(20)).decode()

    def __unicode__(self):
        return self.token
```

Lets run `syncdb` before we forget (using the default `sqlite` is good enough for now.)

```
./manage.py syncdb
```

Our backend and frontend are going to communicate with a RESTful JSON API, so lets create a simple function that helps us return JSON responses.

Create `auth/utils.py`:

```python
import json
from django.http import HttpResponse


def json_response(response_dict, status=200):
    response = HttpResponse(json.dumps(response_dict), content_type="application/json", status=status)
    response['Access-Control-Allow-Origin'] = '*'
    response['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
    return response
```

You might be confused by the `Access-Control-Allow-Origin` and `Access-Control-Allow-Headers` headers. We need these so that Django plays nicely with something called `CORS` or Cross-origin resource sharing. Read about it [here](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing). This is a security feature in modern browsers, particularly Chrome and Safari. Firefox seems to be more lenient with it.

Basically what we are saying is that we allow any website to access our API because of the `Access-Control-Allow-Origin: *` and that we allow that website to have a `Content-Type` and/or a `Authorization` header in its request to us. This is obviously not very secure but works for our example here. For a more complete implementation for future projects, look at the `django-cors-headers` project [here](https://github.com/ottoyiu/django-cors-headers).

In `views.py` we need three main functions. The logic is pretty simple and straightforward. The only surprising thing you will see here is the following snippet:

```python
elif request.method == 'OPTIONS':
    return json_response({})
```

Once again, we need this so that Django plays nicely with `CORS`.

Here are the functions:

- `register`

```python
@csrf_exempt
def register(request):
    if request.method == 'POST':
        username = request.POST.get('username', None)
        password = request.POST.get('password', None)

        if username is not None and password is not None:
            try:
                user = User.objects.create_user(username, None, password)
            except IntegrityError:
                return json_response({
                    'error': 'User already exists'
                }, status=400)
            token = Token.objects.create(user=user)
            return json_response({
                'token': token.token,
                'username': user.username
            })
        else:
            return json_response({
                'error': 'Invalid Data'
            }, status=400)
    elif request.method == 'OPTIONS':
        return json_response({})
    else:
        return json_response({
            'error': 'Invalid Method'
        }, status=405)
```

- `login`

```python
@csrf_exempt
def login(request):
    if request.method == 'POST':
        username = request.POST.get('username', None)
        password = request.POST.get('password', None)

        if username is not None and password is not None:
            user = authenticate(username=username, password=password)
            if user is not None:
                if user.is_active:
                    token, created = Token.objects.get_or_create(user=user)
                    return json_response({
                        'token': token.token,
                        'username': user.username
                    })
                else:
                    return json_response({
                        'error': 'Invalid User'
                    }, status=400)
            else:
                return json_response({
                    'error': 'Invalid Username/Password'
                }, status=400)
        else:
            return json_response({
                'error': 'Invalid Data'
            }, status=400)
    elif request.method == 'OPTIONS':
        return json_response({})
    else:
        return json_response({
            'error': 'Invalid Method'
        }, status=405)
```

**Authenticated Calls**
When the frontend talks to the backend, it embeds its token in the HTTP header called `Authorization` in the format `Token` space `Token Value`. We need to extract this information from the request and check is there is a valid token.

To do this, lets create a simple decorator called `token_required` in our `utils.py` module.

```python
def token_required(func):
    def inner(request, *args, **kwargs):
        if request.method == 'OPTIONS':
            return func(request, *args, **kwargs)
        auth_header = request.META.get('HTTP_AUTHORIZATION', None)
        if auth_header is not None:
            tokens = auth_header.split(' ')
            if len(tokens) == 2 and tokens[0] == 'Token':
                token = tokens[1]
                try:
                    request.token = Token.objects.get(token=token)
                    return func(request, *args, **kwargs)
                except Token.DoesNotExist:
                    return json_response({
                        'error': 'Token not found'
                    }, status=401)
        return json_response({
            'error': 'Invalid Header'
        }, status=401)

    return inner

```

- `logout`

```python
@csrf_exempt
@token_required
def logout(request):
    if request.method == 'POST':
        request.token.delete()
        return json_response({
            'status': 'success'
        })
    elif request.method == 'OPTIONS':
        return json_response({})
    else:
        return json_response({
            'error': 'Invalid Method'
        }, status=405)
```

Now set the `urls.py` and you are done:
```python
urlpatterns = patterns('',
    #...
    url(r'^api/register/$', 'auth.views.register'),
    url(r'^api/login/$', 'auth.views.login'),
    url(r'^api/logout/$', 'auth.views.logout'),
    url(r'^api/username/$', 'auth.views.get_username'),
    #...
)
```

That's it! To make this more scalable for a larger project, you would want to put the logic `allowed_methods` in a `decorator` or use the token-based authentication from the awesome 3rd party app, `django-rest-framework`, along with other things.

## Frontend
Lets open the project in our favorite editor
```
cd frontend
subl .
```

Before we begin, lets create our `app` global so that we don't have to write `angular.module('frontendApp')` each time.

In `app.js`:
```js
use strict;
/* global app: true */

var app = angular.module('frontendApp', [
  'ngCookies',
  'ngResource',
  'ngSanitize',
  'ngRoute'
]);

app.config(function ($routeProvider) {
//...
```

Add `app` to the  `globals` object in `.jshintrc` too:
```js
"globals": {
  "angular": false,
  "app": false
}
```

### Lets Begin

We are going to create 6 components here:

1. `auth` service: responsible for communicating with backend and all business logic
2. `auth` interceptor: middlewear to check if the user is logged in
3. `auth` controller: responsible for displaying stuff on `auth.html`
4. `auth.html` view: a page that allows a user to sign-up and/or login
5. `dashboard` controller: responsible for displaying stuff on `dashboard.html`
6. `dashboard.html` view: a login-required view

The first thing we need to do before tackling these problems is to create a constant that refers to our backend. To do this, we add this line to the end of our `app.js`:

```js
app.constant('API_SERVER', 'http://127.0.0.1:5000');
```

Now, lets build the first part of the `auth` service which is the registration functionality

#### Registration

The main template of our function is as follows in `app/scripts/services/auth.js`:
```js
'use strict';

app.factory('AuthService', function ($http, API_SERVER) {

  var register = function (username, password) {
    // Registration logic goes here
  };

  return {
    register: function (username, password) {
      return register(username, password);
    }
  };

});
```

The first thing we need to do with our registration logic is to send the post request and prepare to handle the response (using a promise):

```js
var url = API_SERVER + 'register/';
$http.post(url, {
  username: username,
  password: password
}, {
  headers: {
    'Content-Type': 'application/json'
  }
}).then(
  function (response) {
    // success callback
  },
  function (response) {
    // error callback
  }
);
```

A response status code between 200 and 299 is considered a success status. When we have a successful response, we will get the token and username and need to save it into `localstorage` or `sessionstorage`.

```js
var token = response.data.token;
var username = response.data.username;

if (token && username) {
  $window.localStorage.token = token;
  $window.localStorage.username = username;
  //success 
} else {
  // error
}
```

If we get a error and move into the error callback, we can do nothing and just pass the data to the controller.

Now, what does this register method return? Since the returned value depends on what happens inside the promise in the callback, it is best that we return a promise ourselves. This way we will be able to handle the situation using a promise idiom in the controller too.

Here is how a promise is created using Angular's `$q`:

```js
// Create the promise
// It's called defer because we are essentially deferring the $http.post
// promise to later. We are deferring it.
var deferred = $q.defer();

// Whenever there is a success
deferred.resolve(success_value);

// Whenever there is an error
deferred.reject(error_value);

// When you are done:
return deferred.promise;
```

In application to the `AuthService`, this is how the registration logic looks:

```js
var deferred = $q.defer();
var url = API_SERVER + 'register/';

$http.post(url, 'username=' + username + '&password=' + password, {
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
}).then(
  function (response) {
    var token = response.data.token;
    var username = response.data.username;

    if (token && username) {
      $window.localStorage.token = token;
      $window.localStorage.username = username;
      deferred.resolve(true);
    } else {
      deferred.reject('Invalid data received from server');
    }
  },
  function (response) {
    deferred.reject(response.data.error);
  }
);
return deferred.promise;
```

Next we build the interface to utilize this service. To do this, we need our `auth.js` controller, a couple of views (`auth.html` and `dashboard.html`) and some routing configuration in `app.js` to tie it all together.

In `app.js`:

```js
app.config(function ($routeProvider) {
  $routeProvider
    .when('/', {
      templateUrl: 'views/auth.html',
      controller: 'AuthCtrl'
    })
    .when('/dashboard', {
      templateUrl: 'views/dashboard.html',
      controller: 'DashboardCtrl'
    })
    .otherwise({
      redirectTo: '/'
    });
});
```

Here is the basic template of what goes on in the controller at `app/scripts/controllers/auth.js`:
```js
'use strict';

app.controller('AuthCtrl', function ($scope, AuthService) {
  $scope.register = function () {
    var username = $scope.register.username;
    var password = $scope.register.password;

    if (username && password) {
      AuthService.register(username, password).then(
        function () {
          $location.path('/dashboard');
        },
        function (error) {
          $scope.error = error;
        }
      );
    } else {
      $scope.error = 'Username and password required';
    }
  };
});

```

When its a success, we just need to forward the user to the `dashboard`. This can be done using Angular's `$location` service.

```js
$location.path('/dashboard');
```

If there is an error, we just update the `$scope` to show the error:
```js
$scope.error = error;
```

That's it! Now, lets move on to the views and create `auth.html` which contains a register form that calls the `register` function we created in our controller.

`/app/views/auth.html`
```html
<h1>Register here!</h1>
<p class="text-danger" ng-show="error">{{ error }}</p>
<form ng-submit="register()">
  <input type="text" class="form-control" ng-model="register.username" placeholder="Username">
  <input type="password" class="form-control" ng-model="register.password" placeholder="Password">
  <input type="submit" class="btn btn-default" value="Submit">
</form>
```

Before we move on to test our current work, lets quickly create a very simple dashboard controller and view that show the token and username. To login protect the dashboard page, all we have to do is check for the token in `localStorage`. If it doesn't exist, we can redirect the user to the login page.

`/app/scripts/controllers/dashboard.js`
```js
'use strict';

app.controller('DashboardCtrl', function ($scope, $window, $location) {
  if (!$window.localStorage.token) {
    $location.path('/');
    return;
  }
  $scope.token = $window.localStorage.token;
  $scope.username = $window.localStorage.username;
});
```

`/app/views/dashboard.html`
```html
<h1>Username: {{ username }}</h1>
<h1>Token: {{ token }}</h1>
```

Finally, lets link all the files we created by adding them as scripts in `index.html`

```html
<!-- build:js({.tmp,app}) scripts/scripts.js -->
<script src="scripts/app.js"></script>
<script src="scripts/controllers/auth.js"></script>
<script src="scripts/controllers/dashboard.js"></script>
<script src="scripts/services/auth.js"></script>
<!-- endbuild -->
```

Woohoo! Lets run both servers and see if everything works correctly.

1. Frontend: `grunt serve`
2. Backend: `./manage.py runserver 0.0.0.0:5000`

Everything should work perfectly :)

> To test the functionality of the dashboard redirecting to the login page if the person is not logged in, just use an incognito window or clear your localStorage if you have already registered and create a token.

#### Login and Logout

Similarly, lets create the login and logout functionalities:

In our simplistic case, the login and register calls are exactly the same from the service perspective. The only thing that is different is the URL to POST to. So, I am going to rename my register function to authenticate and take the URL endpoint as an argument to the function.

In `/app/scripts/services/auth.js`:
```js
var authenticate = function (username, password, endpoint) {
    var url = API_SERVER + endpoint;
    // all the same
```

And in the bottom, where I create the service request, I am going to pass in the appropriate endpoint:

```js
return {
    register: function (username, password) {
      return authenticate(username, password, 'register/');
    },
    login: function (username, password) {
      return authenticate(username, password, 'login/');
    }
  };
```

Thats it for login. Now, lets do the logout functionality.

```js
var logout = function () {
  var deferred = $q.defer();
  var url = API_SERVER + 'logout/';

  $http.post(url).then(
    function () {
      $window.localStorage.removeItem('token');
      $window.localStorage.removeItem('username');
      deferred.resolve();
    },
    function (error) {
      deferred.reject(error.data.error);
    }
  );
  return deferred.promise;
};
```

```js
return {
    // ...
    logout: function () {
      return logout();
    }
  };
```

We have made good progress but this is still incomplete. On the server side, the logout function is an authenticated call with a `token_required` decorator on it. This means that it requires a header. We could add the header over here in the logout function, just like we added the `Content-Type` header in the login and register functionality, but this is not optimal.

We want to add this header whenever the user is authenticated, for all calls. To make this as DRY as possible we need to use something called an `interceptor`. This is very similar to the concept of a `middlewear` module in Django. Every request and response passes through this `interceptor` and we can add the header to every request if the token and username exist in `localStorage`. If a `response` ever returns a `401`, we can redirect the user to the login page. Angular documentation [here](https://docs.angularjs.org/api/ng/service/$http)

Let us go ahead and write this interceptor:

```js
'use strict';

app.factory('AuthInterceptor', function ($rootScope, $q, $window, $location) {
  return {
    request: function (config) {
      config.headers = config.headers || {};
      if ($window.localStorage.token) {
        config.headers.Authorization = 'Token ' + $window.localStorage.token;
      }
      return config;
    },

    responseError: function (response) {
      if (response.status === 401) {
        $window.localStorage.removeItem('token');
        $window.localStorage.removeItem('username');
        $location.path('/');
        return;
      }
      return $q.reject(response);
    }
  };
});
```

Next we need to add this file to our scripts in `index.html` and to our configuration in `app.js`

`index.html`
```html
<!-- build:js({.tmp,app}) scripts/scripts.js -->
<!-- ... -->
<script src="scripts/interceptors/auth.js"></script>
<!-- ... -->
<!-- endbuild -->
```

`app.js`
```js
app.config(function ($routeProvider, $httpProvider) {
  $httpProvider.interceptors.push('AuthInterceptor');
  // route configs...
```

Now the header will be send with every request we make!

Let quickly create the interface for logging in and out and test what we have done.

`/app/scripts/controller/auth.js`
```js
$scope.login = function () {
  var username = $scope.login.username;
  var password = $scope.login.password;

  if (username && password) {
    AuthService.login(username, password).then(
      function () {
        $location.path('/dashboard');
      },
      function (error) {
        $scope.loginError = error;
      }
    );
  } else {
    $scope.error = 'Username and password required';
  }
};
```

`/app/views/auth.html`
```html
<h1>Login here!</h1>
<p class="text-danger" ng-show="loginError">{{ loginError }}</p>
<form ng-submit="login()">
  <input type="text" class="form-control" ng-model="login.username" placeholder="Username">
  <input type="password" class="form-control" ng-model="login.password" placeholder="Password">
  <input type="submit" class="btn btn-default" value="Submit">
</form>
```

`/app/scripts/controller/dashboard.js`
```js
$scope.logout = function () {
  AuthService.logout().then(
    function () {
      $location.path('/');
    },
    function (error) {
      $scope.error = error;
    }
  );
};
```

`/app/views/dashboard.html`
```html
<a ng-click="logout()" class="btn btn-primary">Logout</a>
```

Lets test this out!

**Once you are done with your own testing, try this test-case:**  
First login and once you are on the dashboard, open the `localStorage` options in your browser and delete the `token` and `username`. Now try to click on the logout button, you should be redirected to the login page. This time it is not because you were actually logged out but because logout is an authenticated call and the `token_required` return an `{'error': 'Invalid Header'}` response with a `401` code. This was caught by the `AuthInterceptor` in Angular which pushed us to the login page.

Any call with the `token_required` decorator is going to be automatically protected on the frontend when a user tries to perform a protected action.

## Conclusion
That is all! A disclaimer: this is a very bare-bones implementation, specially on the Django side. I highly recommend using apps like `django-cors-headers` and `django-rest-framework` for future apps. Also, make it a point to think through the functions in the `views.py` properly. These were written by me just for the purpose of this example app and in no way production ready.

Feel free to ask me questions about this via [email](mailto:nikunj.sg@gmail.com) or [twitter](https://twitter.com/_nknj)
