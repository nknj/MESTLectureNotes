# Froms

## About
- Display from easily in HTML
- Validate Data
- Automated Validation Errors
- Parsing data to appropriate data types

## How they work?

- Define in `forms.py`

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField()
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

Could be a "normal" form (`forms.From`) or a `forms.ModelForm`

- Use in a view

Function Based View
```python
def contact(request):
    if request.method == 'POST': # If the form has been submitted...
        form = ContactForm(request.POST) # A form bound to the POST data
        if form.is_valid(): # All validation rules pass
            # Process the data in form.cleaned_data
            # ...
            return HttpResponseRedirect('/thanks/') # Redirect after POST
    else:
        form = ContactForm() # An unbound form

    return render(request, 'contact.html', {
        'form': form,
    })
```

- Display in template

```html
<form action="/contact/" method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="Submit" />
</form>
```

### Defining Froms

#### 'Normal' Forms

Contains `Field`s:
```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField()
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

Relevant Arguments:

- `required` - `True` or `False`
- `label` - Display next to field in HTML
- `initial` - Default value in field
- `widget` - What HTML widget to use `<input type="widget_type">` [List](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#built-in-widgets)
- `help_text` - Help to display next to the field
- `error_messages` - Dict of error `key` and `message`
    + `{key: err_message, key2: err_message2}`
    + Name of keys in [documentation](https://docs.djangoproject.com/en/1.6/ref/forms/fields/#built-in-field-classes)

##### Validation
See Model Froms Validation Section. Only difference is `Model.full_clean()` is not called in 'Normal' forms.

#### Model Forms
- Model Form is a 'Normal' Form PLUS Automatically Generated Fields (from `Meta`)
    + Only fields that do not exist in the 'Normal' part of the form will be automatically generated using the `Meta` class
- Mapping to normal forms [here](https://docs.djangoproject.com/en/1.6/topics/forms/modelforms/#field-types)

```python
class Meta:
    model = Book
    fields = ['name', 'authors']
    # fields = '__all__' # not recommended. security bug on Github
    # exclude = ['title'] # not recommended for the same reason

```

###### Customizing Model Form Fields

Using dictionaries in `Meta`:
```python
class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ('name', 'title', 'birth_date')
        labels = {
            'name': _('Writer'),
        }
        help_texts = {
            'name': _('Some useful help text.'),
        }
        error_messages = {
            'name': {
                'max_length': _("This writer's name is too long."),
            },
        }
```

Re-Declaring Fields like a normal form:
```python
class ArticleForm(ModelForm):
    slug = MySlugFormField()

    class Meta:
        model = Article
        fields = ['pub_date', 'headline', 'content', 'reporter', 'slug']
```
Unlike the `Meta` dictionary method, this doesn't "add-on" to the automatically generated parameters from the model, it resets it to your declarative description.

That is, if you have a `max-length` in your models, it will be present if you add a label using the `Meta` dictionary but it will not be present if you redeclare the field without setting the max-length yourself.


#### Validation
- `from.is_valid()` or `form.errors`
    1. `From.full_clean()`
        1. `Field.clean()` - For each field (have access to raw input from browser, validate format recieved and parsing stuff here):
            1. `to_python()` (used for parsing)
            2. `validate()` **<-- Can override this**
                - Default field format validation
            3. `run_validators()`
                - Runs custom validators provided to the field
                - Eg: `slug = CharField(validators=[validate_name, validate_slug])`)
            4. `clean_<fieldname>()` **<-- Can override this**
                - Has access to 'clean' python object in `self.cleaned_data`
                - Validate **value** and not format of the field, eg: uniqueness)
        2. `Form.clean()` **<-- Can override this**
            - Validating fields based on multiple other fields
            - Eg: Are these two fields unique together?
    2. `Model.full_clean()`
        1. `Model.clean_fields()`
        2. `Model.clean()` **<-- Can override this**
        3. `Model.validate_unique()`

- `f.save()` will also run same validation if `is_valid` has not been called (because it calls `form.errors`)
    - It will return a `ValueError` if invalid

- After validation, all the clean data goes to a `form.cleaned_data` dictionary
    + "Un-clean" data can be accessed using the `request.POST` `QueryDict`

- Errors raised in `Form.clean()` and `Model.clean()` are non-field errors accessible using `form.non_field_errors()`
    + The rest are field errors accessible using `self._errors`
        * Dictionary of `<field_name>` to `list` of errors, can be modified
    + In `From.clean()`
        * non-field error: `raise forms.ValidationError('custom error')`
        * field error: `self._errors["field_name"] = self.error_class([u'custom error'])`

##### How to Override

Call the super constructor before doing anything else (except for `clean_<fieldname>`):
```python
cleaned_data = super(UserCreationForm, self).clean(*args, **kwargs)
```

Return appropriate data at the end:
- For `clean()` needs to return a `cleaned_data` dictionary
- For `clean_<fieldname>()` needs to return the cleaned, final value for that field

Raise Validation Errors in this format:
```python
raise ValidationError(
    _('Invalid value: %(value)s'),
    code='invalid', # Another error with same code will override this error
    params={'value': '42'},
)
```

Multiple Errors:
```python
raise ValidationError([
    ValidationError(_('Error %(value)'), code='username', params={'value': 1}),
    ValidationError(_('Error %(value)'), code='email', params={'value': 2}),
])
```

[Some examples](https://docs.djangoproject.com/en/1.6/ref/forms/validation/#using-validation-in-practice)

##### Model Validation or Form Validation?

- Model Validation applies to ALL forms (including admin forms)
    + Example: Event model with start and end date
        * You need to stop a user from creating an event in the past using your user-facing interface but you should allow it using the admin panel
        * So you can't put the past date validation in the model, you must put it in your form
        * [More here](http://stackoverflow.com/a/14657691/566376)

Note: 

#### Overriding Form.save()
- You can override save too


#### Overriding Model.save()
- `Model.save()` doesnt automatically call `Model.full_clean()`
    + Some people override the `save()` function to include a call to `full_clean()`. [More here](http://stackoverflow.com/questions/4441539/why-doesnt-djangos-model-save-call-full-clean)
- Or you can override `Model.save()`


### Dealing with Forms in Views

#### Binding Forms with Data
Can be any data in a dictionary but its is usually the `request.POST` `QueryDict`

```python
form = FormClass(request.POST)
form2 = FormClass(request.GET)
form3 = FormClass({'field': 'value', 'field2': 'value2'})
```

#### Validating
```python
# Runs Validation and returns true if no ValidationError  thrown
if form.is_valid(): 
    # Clean data only avialable after is_valid is called
    field = form.cleaned_data['field_name'] 
else:
    # error filled form (access error using form.errors)
    return render(request, 'form.html', {'form': form})
```

#### Creating
```python
form.save()
```

#### Editing
```python
u = MyUser.objects.get(username=username)
form = FormClass(instance=u)
form.save()
```

#### Other

##### Dynamic Initial Values

```python
form = FormClass(initial={'field_name':'field_value'})
```

### Showing Forms in Templates

- Each `Form` has:
    + `non_field_errors` (`ul class='errorlist'`)
    + `fields`
        * `errors` (`<ul class='errorlist'>`)
        * `label` (`<label>`)
        * the `field` itself (`<input>`)
- All ids, names and classes are partially customizable by defining them in the form definition @ the form class in `forms.py`
    + Not a big fan of this as it mixes front and back end a little too much for my comfort
    + Hinders flexibility for the frontend developer


