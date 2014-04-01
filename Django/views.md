# Views

```python
def my_view(request):
    # do something
```

## What can a view return?
- HTML (directly or using template)
- Error (404, 500, 403, etc...)
- Redirect
- Non-HTML Documents

### HTML
- Directly: `HttpResponse`
- Template: `TemplateResponse`, `render`, `render_to_response` 

### Error
- 404: `HttpResponseNotFound` or `raise Http404`
    + template - `404.html` or `settings.handler404`
    + `get_object_or_404` and `get_list_or_404`
        * 1st arg: Model or Queryset
        * 2nd arg: Filter Parameter
- 500: `HttpResponseServerError` 
    + template - `500.html` or `settings.handler500`
- 403: `HttpResponseForbidden` or `raise PermissionDenied`
    + template - `403.html` or `settings.handler403`
- 400: `HttpResponseBadRequest`
    + template - `400.html` or `settings.handler400`
- Others: `HttpResponse('<html>Error 503</html>', status=503)`

### Redirect
- `redirect()`, possible arguments:
    + Model (will call `model.get_absolute_url()`)
    + URL Name (will `reverse` it with the URLConf)
    + Actual URL (like `/a/b` or `http://google.com`)
- `HttpResponseRedirect`

### Other Document Types
Just set the content_type in the response like this:
```python
HttpResponse(my_data, content_type='content_type_goes_here')
```

- Plain Text: `text/plain`
- PDF: `application/pdf`
- JSON: `application/json` 
- PNG Image:  `image/png`
- Excel File: `application/vnd.ms-excel`

## Decorating Views

- `@login_required`
- `@csrf_exempt`
- `@require_http_methods`

```python
@require_http_methods(["GET", "POST"])
def my_view(request):
    # I can assume now that only GET or POST requests make it this far
```

## Manipulating Requests

- HTTP Method: `request.method`

```python
if request.method == 'GET':
    do_something()
elif request.method == 'POST':
    do_something_else()
```

- GET and POST parameters

```python
try:
    a = request.GET['a']
except KeyError:
    raise NotFoundInGetException

b = request.GET.get('b', None)
if b is None:
   raise NotFoundInGetException 
```
Same for `POST`
To find in both `GET` and `POST` use `REQUEST`

- User
`request.user` returns logged-in user or `AnonymousUser`

- Cookies
`request.COOKIES` returns a `dict` of all cookies

## Manipulating Responses

- Manipulating Headers
```python
response = HttpResponse()
response['Age'] = 120
del response['Age']
```