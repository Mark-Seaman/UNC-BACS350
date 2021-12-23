# Django Tests Design Pattern


## Overview

### Goals

Build a standard set of tests that can be automatically applied to each
new data type.

Build test without thinking (up and running in 5 minutes).

Generate full test coverage.


### Steps
* Define data model
* Define Data Test class
* Define View Test class
* Use template to create automatic code


### Design Pattern for Django Tests
* Blank database
* Setup
* Fixtures
* Data (CRUD)
* Views (list, detail, add, edit, delete)
* Standard test design


### Blank database
* Before every test case a blank database is created
* All database operations are performed on this test database


### Setup
* Initialization steps can be added to the setUp function
which is run before every test case.
* Objects needed by multiple tests are added to setUp.

```python
def setUp(self):
    self.user, self.user_args = create_test_user()
    self.author = Teacher.objects.create(user=self.user, name='Mark')
    self.course = Course.objects.create(title='BACS 350', author=self.author)
```


### Fixtures
* JSON test fixture can set the blank database to any known state
* Test can be run in that database context


### Data Test (CRUD)
* Test for each operation (CREATE, READ, UPDATE, DELETE)


```python
class LessonDataTest(TestCase):

    def setUp(self):
        self.user, self.user_args = create_test_user()
        self.author = Teacher.objects.create(user=self.user, 
                                            name='Mark Seaman')
        self.course = Course.objects.create(name='bacs200', title='BACS 200', 
                                            author=self.author, 
                                            doc_path='Documents/Course/bacs200')
        self.lesson1 = dict(course=self.course, title='Github', order='1')
        self.lesson2 = dict(course=self.course, title='Servers', order='2')

    def test_add_lesson(self):
        self.assertEqual(len(Lesson.objects.all()), 0)
        Lesson.objects.create(**self.lesson1)
        Lesson.objects.create(**self.lesson2)
        self.assertEqual(len(Lesson.objects.all()), 2)

    def test_lesson_list(self):
        Lesson.objects.create(**self.lesson1)
        Lesson.objects.create(**self.lesson2)
        b = Lesson.objects.filter(course=self.course).order_by('order')
        self.assertEqual(b[0].title, 'Github')
        self.assertEqual(b[1].title, 'Servers')
        self.assertEqual(b[1].document, 'Documents/course/bacs200/lesson/02.md')

    def test_lesson_edit(self):
        Lesson.objects.create(**self.lesson1)
        b = Lesson.objects.get(pk=1)
        b.title = 'Servers'
        b.order = 2
        b.save()
        self.assertEqual(b.title, 'Servers')
        self.assertEqual(b.order, 2)
        self.assertEqual(b.document, 'Documents/course/bacs200/lesson/02.md')

    def test_lesson_delete(self):
        Lesson.objects.create(**self.lesson1)
        b = Lesson.objects.get(pk=1)
        b.delete()
        self.assertEqual(len(Lesson.objects.all()), 0)
```


### Views Test
* Test all five views: list, detail, add, edit, delete

```python
class LessonViewsTest(TestCase):

    def login(self):
        response = self.client.login(username=self.user.username,  password=self.user_args['password'])
        self.assertEqual(response, True)

    def setUp(self):
        self.user, self.user_args = create_test_user()
        self.author = Teacher.objects.create(user=self.user, name='Charles Dickens')
        self.course = Course.objects.create(title='Tale of Two Cities', author=self.author,
                                            description='description', doc_path='Documents')
        self.lesson1 = dict(course=self.course, title='Best of Times', order='1', date='2001-09-11', week='1')
        self.lesson2 = dict(course=self.course, title='Worst of Times', order='2', date='2001-09-18', week='2')

    def test_lesson_list_view(self):
        Lesson.objects.create(**self.lesson1)
        self.assertEqual(reverse('lesson_list'), '/lesson/')
        response = self.client.get('/lesson/')
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'lesson_list.html')
        self.assertTemplateUsed(response, 'theme.html')
        self.assertContains(response, '<tr>', count=2)

    def test_lesson_detail_view(self):
        self.assertEqual(reverse('lesson_detail', args='1'), '/lesson/1')
        self.assertEqual(reverse('lesson_detail', args='2'), '/lesson/2')
        Lesson.objects.create(**self.lesson1)
        response = self.client.get('/lesson/1')
        self.assertEqual(response.status_code, 200)

    def test_lesson_add_view(self):

        # Add without Login
        response = self.client.post(reverse('lesson_add'), self.lesson1)
        self.assertEqual(response.url, '/accounts/login/?next=/lesson/add')

        # Login to add
        self.login()
        response = self.client.post(reverse('lesson_add'), self.lesson1)
        response = self.client.post(reverse('lesson_add'), self.lesson2)
        self.assertEqual(response.status_code, 302)
        self.assertEqual(response.url, '/lesson/')
        self.assertEqual(len(Lesson.objects.all()), 2)

        # List the lessons
        response = self.client.get('/lesson/')
        self.assertContains(response, '<tr>', count=3)

    def test_lesson_edit_view(self):

        # Edit without Login
        Lesson.objects.create(**self.lesson1)
        self.assertEqual(reverse('lesson_edit', args='1'), '/lesson/1/')
        response = self.client.get('/lesson/1/')
        self.assertEqual(response.status_code, 302)
        self.assertEqual(response.url, '/accounts/login/?next=/lesson/1/')

        # Login to edit
        self.login()
        response = self.client.post('/lesson/1/', self.lesson2)
        self.assertEqual(response.status_code, 302)
        self.assertEqual(response.url, '/lesson/')

        # Check the course object
        c = Lesson.objects.get(pk=1)
        self.assertEqual(c.title, self.lesson2['title'])
        self.assertNotEqual(c.title, self.lesson1['title'])
        self.assertEqual(c.document, 'Documents/course/XXX/lesson/02.md')

    def test_lesson_delete_view(self):
        self.login()
        Lesson.objects.create(**self.lesson1)
        self.assertEqual(reverse('lesson_delete', args='1'), '/lesson/1/delete')
        response = self.client.get('/lesson/1/delete')
        self.assertEqual(response.status_code, 200)
        response = self.client.post('/lesson/1/delete')
        self.assertEqual(len(Lesson.objects.all()), 0)
```


### Standard test design
* Use a standard template for generating new tests
* Customize class name, object name, field names
* Comment out all new test code
* Debug one test case at a time

