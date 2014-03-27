# ORM (Object-Relational Mapping)
[Source](https://docs.djangoproject.com/en/1.6/topics/db/queries/)

Mapping objects in a programming language to a database.

- Allows saving objects to database
- Taking data from the database and converting into an object
    + Good ORMs allow complex queries on the database

## Python ORMs

- **SQLAlchemy**
- **Django ORM**
- Storm
- Peewee
- many more...

## Django ORM

All `models.Model` classes have the `Model` itself, and a `ModelManager`. 

```python
class Poll(models.Model):
    question = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def __unicode__(self):
        return self.question

class Choice(models.Model):
    poll = models.ForeignKey(Poll)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __unicode(self):
        return self.choice_text
```

- The `Model` describes the data that the object needs to take care of and the business logic related to that object. In `Poll`:
    + Data:
        * `question = models.CharField(max_length=200)`
        * `pub_date = models.DateTimeField('date published')`
    + Business Logic:
        * `def was_published_recently(self):`
        * `def __unicode__(self):`
        * `def clean(self):`
- The `ModelManager` describes the behavior of multiple Model objects (saving objects, querying them, etc.)
    + `Poll.objects` access the the ModelManager for `Poll`
        * `.all()`
        * `.filter()`
        * `.get()`
        * `.count()`
        * `.create()`
    + Some methods are called on the `Model` but invoke the Manager in the background 
        * `.save()`
        * `.delete()`
    + You can access managers of related objects too!
        * `p.choice_set`
            - Since `p` is a `Poll` object, you can access all the related `Choice` objects because `Choice` has a FK `Poll`
            - This is pretty powerful stuff!
        * `p.choices`
            - If you change: 
                + `poll = models.ForeignKey(Poll)` **to:** `poll = models.ForeignKey(Poll, related_name=choices)`
                + `p.choices` instead of `p.choice_set`
                + More intuitive

### Queries

#### Methods

##### Creating stuff
- Create object: `save` on **object** or `create` on **manager**

```python
p = Poll(question="What's new?", pub_date=timezone.now())
p.save()
```

or

```python
p = Poll.objects.create(question="What's new?", pub_date=timezone.now())
```

##### Reading stuff
- Return one object: `get` on manager
- Return list (QuerySet) of multiple objects: `filter` on manager
- Return list (QuerySet) of multiple objects: `exclude` on manager
- Return all objects: `all` on manager
- Return count of objects: `count` on manager

Examples in next section...

##### Updating  stuff
- Update object: `save` on object

```python
p.question = "New Question?"
p.save()
```

- or `update` on multiple objects

```python
Poll.objects.filter(question="test").update(question="test?")
# Changes all the objects with question: test to question: test?
```

##### Deleting stuff
- Delete object: `delete` on object

```python
p.delete()
# Deletes single object p
Poll.objects.filter(question="test").delete()
# Deletes all objects with question: test
```

#### More on Reading Stuff

##### Field Lookups
- You can look up using one field:

```python
Poll.objects.get(id=1)
# returns poll with id: 1
Poll.objects.get(pk=1)
# returns poll with id: 1, since pk is id in our case
Poll.objects.filter(question="test")
# returns a list of all the polls with the question: test
```

- or multiple fields:

```python
Poll.objects.get(id=2, question="test")
# returns poll with id 2 and question: test
```

- You can also concatenate or chain queries:

```python
Poll.objects.filter(question="test").get(id=2)
# returns the object with id: 2 from the set of objects with the question: test
```

##### Other notes:
- Using `get` can raise the `Poll.DoesNotExist` exception
- Querysets are lazily evaluated (not run until they are needed)
- Querysets are very similar to lists and can be sliced!
    + `Poll.objects.all()[:5]`
    + `Poll.objects.all()[5:10]`
    + However, negative indexing is **not supported**: `Poll.objects.all()[-1]`
- You can also order queries using `Poll.objects.order_by('question')`

#### Complex Field Lookups

- `__exact` like `Poll.objects.filter(question__exact="test?")`
- `__iexact` like `Poll.objects.filter(question__iexact="TeSt?")`
- `__contains` like `Poll.objects.filter(question__contains="es")`
- `__startswith` or `__endswith`
    + like `Poll.objects.filter(question__startswith="te")`
    + like `Poll.objects.filter(question__endswith="st?")`

- `__in`
- `__gt`, `__gte`, `__lt`, `__lte`
- many more...

##### Related Objects

- Forwards:
```python
Choice.objects.filter(poll__question="test?")
# returns all the choices with a poll that has a question: test?
```

- Backwards (use lowercase model name):
```python
Poll.objects.filter(choice__votes=3)
# returns all the Polls that have a choice with 3 votes. This may have
# duplicates if more that one choice has 3 votes. To avoid duplicates call
# set([p for p in Poll.objects.filter(choice__votes=3)])
```

This is similar but different from:
```python
p = Poll.objects.get(id=1)
p.choice_set.filter(votes=3)
```

This returns the choice objects linked specifically to `Poll.objects.get(id=1)`. The previous snippet returns all Polls objects with a choice that has 3 votes.

### Complex Queries

- Read up on `F` and `Q` class
- Using `select_related` to cache objects