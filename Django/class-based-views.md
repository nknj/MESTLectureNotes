# Class-Based Views in Django

### What is a Class-Based View?
Using a class instead of a function to handle requests and return responses. This is better becuase you can use methods for different request methods instead of if-statements. You can also use Mixins (Multiple Inheritance).

```python
from django.http import HttpResponse
from django.views.generic.base import View

class MyView(View):
    result = "Hello"

    def get(self, request):
        return HttpResponse(self.result)

    def post(self, request):
        return HttpResponse(self.result + " from POST!")
```

To run call ClassName followed by `as_view()`

```python
MyView.as_view(result="Goodbye!")
GoodByeView.as_view()
BookListView.as_view()
```

You can override things, in the `as_view` function it self or by creating a new Class. So, `MyView.as_view(result="Goodbye!")` and `GoodByeView.as_view()` are the same if:

```python
from some_app.views import MyView

class GoodByeView(MyView):
    result = "Goodbye!"
```

They can be put into the URLconf:

```python
# urls.py
from django.conf.urls import patterns
from some_other_app.views import GoodByeView

urlpatterns = patterns('',
    (r'^bye/', GoodByeView.as_view()),
)
```

They can be decorated as such:

```python
(r'^about/', login_required(ProtectedView.as_view()))
```

or

```python
class ProtectedView(View):

    @method_decorator(login_required)
    def dispatch(self, *args, **kwargs):
        return super(ProtectedView, self).dispatch(*args, **kwargs)
```
but as you will find out, it is better to create a mixin for this as it will be used often. Or you could get a mixin from [django-braces](https://github.com/brack3t/django-braces)

### Base Views
Views that other Generic Views  
[Reference](https://docs.djangoproject.com/en/dev/ref/class-based-views/flattened-index/)

#### View
**Attribute**: `http_method_names`

- Default: `['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']`
- If you remove something from here the method `http_method_not_allowed(request)` will be called

**Method**: `dispatch(request)`

- The main method that runs the view when you call `as_view`
- Looks at the request method and calls the appropriate method
    + HEAD calls `get()` by default, unless you override `head()`

#### TemplateView

Used to return a Template to the View
Inherits from `ContextMixin`, `TempalteResponseMixin` and `View`

See `TemplateResponseMixin` and `ContextMixin` for attributes in addition to `View` attributes

#### RedirectView
**Attribute**: `url`

- The url to redirect to

**Attribute**: `pattern_name`

- The name of the URLconf pattern to redirect to

**Attribute**: `permanent`

- True = Http Status Code 301
- False = Http Status Code 302

**Attribute**: `query_string`

- If the query_string should be passed on to the next url or not

**Method**: `get_redirect_url(*args, **kwargs)`

- To do stuff before redirecting
- This method looks for `url` and if it doensnt find, it looks for `pattern_name` and finally adds a `query_string` if present

```python
def get_redirect_url(self, *args, **kwargs):
    article = get_object_or_404(Article, pk=pk)
    article.update_counter()
    return super(ArticleCounterRedirectView, self).get_redirect_url(*args, **kwargs)

```


### Mixins
#### What are Mixins?
Mixins are classes that are meant to take out a particular set of methods or attributes or both out of multiple classes in which they are repeatedly used in order to make for a more DRY approach.

For example, if you use a `method_decorator(login_decorator)` on `dispatch` in multiple classes, you can take it out and put it into a mixin class. A mixin class extends the object class or another mixin class - NOT a `View` class. You cannot extend a Class Based view with more then one class that is the child of a `View`. It has to be a `View` class and a bunch on mixins (that extend `object`).

The Mixin is useless on its own. Since it extends object. It must be combined with a View class in some form of multiple inheritance structure.

#### Multiple Inheritance in Python

```python
class DerivedClassName(Base1, Base2, Base3):
    # ...
    # ...
```

Languages with Multiple Inheritance have a Method Resolution Order (MRO). This dictates the order in which variables or method names will be found when attribute or method is called. Python will do a depth first (left to right) search of the element in question. So it will look in Base1, then everything that Base1 inherits from, and then Base2 and so on...

So, the order of your mixin inheritance matters. The first one that element name is found in will be run. This can create a maintainence nightmare if you mistakingly name 2 things the same way and have unexpected errors, so be careful about the order of things.

#### Mixins in Django
Mixins are the base of Generic Views, once you know what the mixins contain, you will know what the Generic Views contain as they are just aggragations of those mixins

##### Simple Mixins

###### ContextMixin
Returns the context using the method `get_context_data`. `context` is the data to be passed to the template. You can override this method to add more data to the context:

```python
def get_context_data(self, **kwargs):
    # Call the base implementation first to get a context
    context = super(PublisherDetail, self).get_context_data(**kwargs)
    # Add in a QuerySet of all the books
    context['book_list'] = Book.objects.all()
    return context
```

###### TemplateResponseMixin
Creates a response that uses a template (`TemplateResponse`).

**Attribute**: `template_name`

- Which template to render

**Attribute**: `response_class`

- Which response class to use, default is `TemplateResponse`

**Attribute**: `content_type`

- mimetype, default is `text/html`

**Method**: `render_to_response(context, **repsonse_kwargs)`

- Creates the `response_class` object with the template from `get_template_name` (next) and returns it

**Method**: `get_template_names()`

- Used to determine the `template_name`
- If you specify it in the attribute (`template_name`) it returns that, otherwise it can be dynamically generated here
    + Mixins like `SingleObjectTemplateResponseMixin` create it dynamically by overriding this method.

##### Single Object Mixins

###### SingleObjectMixin
Used to get the object required by the request. The `context` (what is passed to the View or usually Template) contains the object under the vairbale `object` or `<model name>` (By default. This can be overridden by `context_object_name`)

**Attribute**: `model`

- Which model in which the object should be searched for using the `pk` or `slug`
- `model = Foo` is the same as `queryset = Foo.objects.all()`

**Attribute**: `queryset`

- You can specify a custom queryset within which the object should be searched for using the `pk` or `slug`
- `queryset = Foo.objects.all()` is the same as `model = Foo` 

**Attribute**: `pk_field_kwargs`

- Name of the var in URLconf that you can get the pk from
    + Default `pk`, so if you use `url(r'^authors/(?P<pk>\w+)/$' ...),` you will not need to make any changes to this field
    + But if you use `url(r'^authors/(?P<hahahah>\w+)/$'...` you will need to set `pk_field_kwargs = hahahah`

**Attribute**: `slug_field_kwargs`

- Name of the var in URLconf that you can get the slug from
    + Default `slug`, so if you use `url(r'^authors/(?P<slug>\w+)/$' ...),` you will not need to make any changes to this field
    + But if you use `url(r'^authors/(?P<hahahah>\w+)/$'...` you will need to set `slug_field_kwargs = hahahah`

**Attribute**: `slug_field`

- Name of the field in the model that should be called to get the slug
    + Default is `slug` but if you call it something like `short_hand`, you will have to set `slug_field = short_hand`

**Attribute**: `context_object_name`

- You can use this to override the name of the object in the template
- Default is retrieved from `get_context_object_name(obj)`

**Method**: `get_object()`

- Returns the object by getting the object from the `queryset`/`model` specified in the attribute and the `pk` or if not `slug` provided.
- After the object is retrieved here, you can edit it and save it or change some related object or whatever.

**Method**: `get_queryset()`

- Evaluates the `queryset` or `model`

**Method**: `get_context_object_name(obj)`

- Uses the value in `context_object_name` for the name of the object in the template
- If none is provided it returns the `model` name is `lowercase`
    + `Article` model will return `article`

**Method**: `get_context_data(**kwargs)`

- returns the object with the name determined in `get_context_object_name`

**Method**: `get_slug_field()`

- returns the slug value

###### SingleObjectTemplateResponseMixin
Extends `TemplateReponseMixin`
This extension is used mostly to get the template name from the object found at `self.object`, usually done by `SingleObjectMixin`

**Attribute**: `template_name_field`

- This is used in determining the name of the template using a field of the object. So, you can set the name of the template to be `object.type`

**Attribute**: `template_name_suffix`

- You can add a suffix to the `template_name_field`
    + Default is `_detail`

**Method**: `get_template_names()`

Returns a list of possible names:

- `template_name` if provided (from `TemplateResponseMixin`)
- `template_name_field` + `template_name_suffix`, if provided
- `<app_label>/<objects_model_name><template_name_suffix>.html`

###### Generic Single Object View

**DetailView**  
Inherits from:  
(`SingleObjectTemplateResponseMixin` > `TemplateResponseMixin`),
(`BaseDetailView` > `SingleObjectMixin`, `View`)  
There are no other methods or attributes that `DetailView` has  

The `BaseDetailView` can be used if you aren't interested in making a template response. You might wanna combine it with somthing like a `JSONResponseMixin` to get a JSON in the view.


##### Multiple Object Mixins

###### MultipleObjectMixin
Used to get a list of objects required by the request. The `context` (what is passed to the View or usually Template) contains the objects under the vairbale `object_list` or `<model name>_list` (By default. This can be overridden by `context_object_name`). It also contains variables dealing with pagination (`is_paginated`, `paginator`, `page_obj`)

**Attribute**: `allow_empty`

- If `True`, returns template even when there are no objects in the list
- If `False`, returns 404 when there are no objects in the list
- You can get this value using the `get_allow_empty()` method

**Attribute**: `model`

- To get all objects in the model, same as `queryset = modelName.objects.all()`

**Attribute**: `queryset`

- Specify a custom queryset `queryset = modelName.objects.filter(...)`
- Supercedes the value of `model`
- If you want to construct a more complex set of statements to define the queryset, you need to override `get_queryset()`

**Attribute**: `context_object_name`

- Name of the variable holding the object list in the template
- `get_context_object_name` sets this to `model_name` + `_list` if the attribute is not provided

**Pagination**

**Attribute**: `paginate_by`

- How many objects per page, needs page number via `request.GET['page']` or URLconf. This info can be retrieved by calling `get_paginate_by(queryset)`

```python
(r'^objects/page(?P<page>[0-9]+)/$', PaginatedView.as_view())
# or objects?page=3
```

**Attribute**: `paginate_orphans`

- Maximum number of items on the last page
    + To avoid having very few items on the last page
    + So if `paginate_orphans` is 5 and `paginate_by` is 10, there will be anywhere between 1 and 15 items on the last page
        * If there are 16 items left, 10 will go on the first and 6 on the next
- This value can be retrieved by `get_paginate_orphans()`

**Attribute**: `page_kwarg`

- The name of the variable in `request.GET` or URLconf that holds the page number 

**Attribute**: `paginator_class`

- Custom pagination class you wanna use, default is django.core.paginator.Paginator. This value can be retrieved by `get_paginator(queryset, per_page, orphans=0, allow_empty_first_page=True)`

**Method**: `paginate_queryset(queryset, page_size)`
- Gets current page contents. Current Page determined by `page` variable from request

###### MultipleObjectTemplateResponseMixin
Extends `TemplateReponseMixin`
This extension is used mostly to get the template name from the object found at `self.object_list`, usually done by `MultipleObjectMixin`

**Attribute**: `template_name_suffix`

- You can add a suffix to the `model_name`
    + Default is `_list`

**Method**: `get_template_names()`

Returns a list of possible names:

- `template_name` if provided (from `TemplateResponseMixin`)
- `<app_label>/<objects_model_name><template_name_suffix>.html`

###### Generic Multiple Object View

**ListView**  

Inherits from:
(`MultipleObjectTemplateResponseMixin` > `TemplateResponseMixin`),   
(`BaseListView` > `MultipleObjectMixin`, `View`)  

There are no other methods or attributes that `ListView` has

The `BaseListView` can be used if you arent interested in making a template response. You might wanna combine it with somthing like a `JSONResponseMixin` to get a JSON in the view.

##### Editing Mixins

###### FromMixin
Used to create a form from a `Form` class. The form is passed under the `form` variable in the `context`, usually to the template

**Attribute**: `initial`

- A dictionary containing the inital form data 
- This can be retrieved using `get_initial()`
    + Override to create dynamic `initial`

**Attribute**: `form-class`

- The name of the Forms class in your `forms.py`
- This can be retrieved using `get_form_class()`
    + Override to create dynamic `form-class`

**Attribute**: `success_url`

- The URL to redirect to when the form is successfully processed.
- This can be dynamic with the use of `%` and the name of a model's field
    + Ie, you can set it to be dynamic using the field `/article/%(slug)s`. This will get `article.slug` and place it in the end
- This can be retrieved using `get_success_url()`
    + Can be overridden to generate a dynamic success url

**Attribute**: `prefix`

- Django can prefix your `id`s and `names` with a prefix for a more unique auto-generated form
- This can be retrieved using `get_prefix()`

**Method**: `get_form()`

- Creates the form using `get_form_class()` and `get_form_kwargs()`.
    - `get_form_kwargs()` builds the keyword args required for the from. I adds initial data using `get_inital()` and looks at the request method to see if it should be `request.POST` or `request.FILES` or what

**Method**: `form_valid(form)`
- Called when the form is valid
    + You can override this to do extra stuff before redirecting
        * Example: save object to multiple tables OR
        * Send an mail notifying something was saved/created/updated
- Returns a redirect to the `get_success_url` at the end.

**Method**: `form_invalid(form)`
- Must be implemented by the class inheriting this, renders a response with an invalid form in the `form` variable of context.

**Method**: `get_context_data(**kwargs)`
- Add additional information to context

###### ModelFormMixin
Used to create a ModelForm. Extends the `SingleObjectMixin` and `FormMixin`. It overrides `form_valid` and `form_invalid` against the model. In `form_valid`, it sets `object` in the `context` with the value of the saved object before redirecting to the `success_url`

**Attribute**: `fields`

- List of fields from the model to include in the form
- Good practice to use this.
    + Not using it will be deprecated soon (Django 1.8)

###### ProcessFormView
Extends `View` to deal with `get`, `put` and `post` specifically for forms. Although this is a View, it cannot be used alone so it is a mixin too

**Method**: `get`

- Constructs the form and renders the response with that form in the context

**Method**: `post`

- Recieves the response, checks validity, and handles where to redirect to using `form_valid` or `form_invalid`

**Method**: `put`

- Passes everything to `post`

###### DeletionMixin
Handles the DELETE request method.

**Attribute**: `success_url`

- The URL to redirect to when the form is successfully processed.
- This can be retrieved using `get_success_url()`
    + Can be overridden to generate a dynamic success url

##### Generic Editing Views

###### FromView
Inherits from:  
(`TemplateResponseMixin`),  
(`BaseFormView` > 
    `FormMixin`,  
    (`ProcessFromView` > `View`)
)

Displays a form and handles the response.
You have to specify `template_name` here because there is no mixin overriding the `get_template_name` method name here.

Other necessary attributes to be set are `form_class` and `success_url` from `FormMixin`

###### CreateView
Inherits from:  
(`SingleObjectTemplateResponseMixin` > `TemplateResponseMixin`),    
(`BaseCreateView` > 
    (`ModelFormMixin` > `FormMixin`, `SingleObjectMixin`),  
    (`ProcessFromView` > `View`)
)  

Displays a model form to create a new object and handles the response.
You **must** specify the `model` and the `fields` you want to show.

The `template_name` here is retrieved from the `SingleObjectTemplateResponseMixin` class but:  

**Attribute**: `template_name_suffix`

- Adds the the end of the `template_name`, default is `_form`, so if you are creating a new `Author`, the template name will be `author_form`
- Advisable to change this to `_create_form` so that it is more unique... `UpdateView` and `DeleteView` also use `_form`

**Attribute**: `object`

- You can access the object that was created in this class using the `object` attribute

###### UpdateView
Inherits from:  
(`SingleObjectTemplateResponseMixin` > `TemplateResponseMixin`),    
(`BaseUpdateView` > 
    (`ModelFormMixin` > `FormMixin`, `SingleObjectMixin`),  
    (`ProcessFromView` > `View`)
)  

Displays a model form to update an object and handles the response.
You **must** specify the `model` and the `fields` you want to show.

The `template_name` here is retrieved from the `SingleObjectTemplateResponseMixin` class but:  

**Attribute**: `template_name_suffix`

- Adds the the end of the `template_name`, default is `_form`, so if you are creating a new `Author`, the template name will be `author_form`
- Advisable to change this to `_update_form` so that it is more unique... `CreateView` and `DeleteView` also use `_form`

**Attribute**: `object`

- You can access the object that was created in this class using the `object` attribute

###### DeleteView
Inherits from:  
(`SingleObjectTemplateResponseMixin` > `TemplateResponseMixin`),  
(`BaseDeleteView` >  
    `DeletionMixin`,  
    (`BaseDetailView` > `SingleObjectMixin`, `View`)  
)  

Displays a confirmation page saying that an object has been deleted. **Request Method must be POST**

You **must** specify the `model` and the `success_url` you want to show.

The `template_name` here is retrieved from the `SingleObjectTemplateResponseMixin` class but:  

**Attribute**: `template_name_suffix`

- Adds the the end of the `template_name`, default is `_confirm_delete`, so if you are creating a new `Author`, the template name will be `author_confirm_delete`

##### Date-based Mixins
Can be used to get items from the DB that correspond to a certain year, month, day, week or date


### Exmples of Extending Views

To add more data to a generic view than what is returned by default, override the `get_context_data` function

```python
def get_context_data(self, **kwargs):
    # Call the base implementation first to get a context
    context = super(PublisherDetail, self).get_context_data(**kwargs)
    # Add in a QuerySet of all the books
    context['book_list'] = Book.objects.all()
    return context
```

- To create a filter of data, override the `get_queryset` function.

```python
def get_queryset(self):
    self.publisher = get_object_or_404(Publisher, name=self.args[0])
    return Book.objects.filter(publisher=self.publisher)
```

- To do stuff that is more than just showing it on a template override the `get_object` function

```python
def get_object(self):
    # Call the superclass
    object = super(AuthorDetailView, self).get_object()
    # Record the last accessed date
    object.last_accessed = timezone.now()
    object.save()
    # Return the object
    return object
```

- To do different things with different request methods, override the function with the name of the request method. `post` or `head` or `delete`

```python
def head(self, *args, **kwargs):
    last_book = self.get_queryset().latest('publication_date')
    response = HttpResponse('')
    # RFC 1123 date format
    response['Last-Modified'] = last_book.publication_date.strftime('%a, %d %b %Y %H:%M:%S GMT')
    return response
```


**If you do any of these things multiple times, it makes sense to create a new mixin and add it to the MRO of your View**