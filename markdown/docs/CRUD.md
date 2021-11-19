# CRUD Design Pattern

## Overview

### Goals

Use data models with the Django ORM to update the database records.

Support CREATE, READ, UPDATE, DELETE operations on each data type.

Ensure database integrity.

Work with data using Python.


### Steps
* Define data model as a Python class
* Use operations defined in Django models.Model to do CRUD
* Create named access functions for clarity



## Data

Data Model

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    subtitle = models.CharField(max_length=200, null=True, blank=True)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, editable=False)
```


## CRUD

### CREATE

book/book.py

```python
from book.models import Book

# Create one object
b = Book.objects.create(title='Dune', author=herbert)

# Create if needed
b = Book.objects.get_or_create(title='Dune', author=herbert)[0]

# Create function
def create_book(title, author, subtitle):
    c = Book.objects.get_or_create(title=title, author=author)[0]
    c.subtitle = subtitle
    c.save()
    return c
```


### READ

book/book.py

```python
# Query for one object
book = Book.objects.get(title=title)

book.title

# Query for list of objects
books = Book.objects.filter(author=author)

for b in books:
    print(b.title, b.author)

# Books by author
def books_by_author(author):
    return Book.objects.filter(author=author)
```


### UPDATE

book/book.py

```python
# Edit one object
book = Book.objects.get(title=title)

book.title = 'Children of Dune'
book.subtitle = 'Sequel'

book.save()
```


### DELETE

book/book.py

```python
# Delete one object
book = Book.objects.get(title=title)
book.delete()

# Delete multiple objects
Book.objects.filter(author=author).delete()
```



## Tests

book/tests.py

```python
from django.test import TestCase

class BookDataTest(TestCase):

    def test_book_add(self):
        pass

    def test_book_list(self):
        pass

    def test_book_detail(self):
        pass

    def test_book_edit(self):
        pass

    def test_book_delete(self):
        pass
```

