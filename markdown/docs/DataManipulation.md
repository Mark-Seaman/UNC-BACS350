# Data Manipulation Design Pattern

## Overview

### Goals

Work with data from database in Python

Typical data processing

* Queries to work with database records
* Processing with Python Lists, Dictionaries
* Read and write files
* Pass dictionary to templates within the view code


### Steps
* Common Queries
* File read and write
* Records to dictionary
* Use a generic template to display the info
* Allow for display of multiple documents



## Data Queries

See [CRUD Design Pattern](CRUD) for details

```python
# CREATE
book = Book.objects.get_or_create(title=title, author=author)[0]

# READ
book = Book.objects.get(title=title)
books = Book.objects.filter(author=author)

# UPDATE
book.title = 'Children of Dune'
book.save()

# DELETE
book.delete()
```


## File read and write

Read and write files

```python
# Read a file
text = open(filepath).read()

# Write a file
open(filepath, 'w').write(text)
```


Read and write CSV files

```python
from csv import reader, writer

def read_csv_file(path):
    with open(path) as f:
        return [row for row in reader(f)]

def write_csv_file(path, table):
    with open(path, 'w', newline='') as f:
        writer(f).writerows(table)

```



## Values of Data Records

Query for the records and show values

```python
# Query for books
books = Book.objects.all()

# Convert to values list
for book in books.values_list():
    print(book)

>>> (1, 'Leverage Principle', None, 1, 'None', 'Leverage')
>>> (2, 'From the Edge of Reality', None, 1, 'Insights and irony', 'Poems')
```


Convert to dictionaries

```python
# Query for books
books = Book.objects.all()

# Convert to dictionary
for book in books.values():
    print(book)

>>> {'id': 1, 'title': 'Leverage Principle', 'subtitle': None, 'author_id': 1, 
'description': 'None', 'doc_path': 'Leverage'}
>>> {'id': 2, 'title': 'Edge of Reality', 'subtitle': None, 'author_id': 1, 
'description': 'Insights and irony', 'doc_path': 'Poems'}


```


Print dictionary with indents

```python
# Print with indents
from pprint import pformat

print(pformat(book, indent=4))


{   'author_id': 1,
    'description': 'Insights and irony',
    'doc_path': 'Poems',
    'id': 2,
    'subtitle': None,
    'title': 'Edge of Reality'}

```