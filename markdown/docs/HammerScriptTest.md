# Hammer Script Test Design Pattern


### Demo Code
* Clone my repo and study the code in 
[Hammer Test Source Code](https://github.com/Mark-Seaman/BACS350/tree/main/week14/HammerTest)
* Build the code and experiment with it
* Set up tests for your favorite web sites
* Do it simple; do it now!


### Data

probe/models.py

```python
from django.db import models
from django.urls.base import reverse_lazy


class Test(models.Model):

    name = models.CharField(max_length=100)
    expected = models.TextField(default='Initial Output', null=True)
    source = models.TextField(default='none')

    def __str__(self):
        return f'{self.name}'

    def get_absolute_url(self):
        return reverse_lazy('test_list')

    @staticmethod
    def create(**kwargs):
        x = Test.objects.get_or_create(name=kwargs.get('name'))[0]
        expected = kwargs.get('expected')
        if expected:
            x.expected = expected
        x.source = kwargs.get('source')
        x.save()


class Result(models.Model):

    probe = models.ForeignKey(Test, on_delete=models.CASCADE, editable=False)
    date = models.DateTimeField(auto_now=True)
    output = models.TextField(default='Not yet run')
    passed = models.BooleanField(default=False)

    def __str__(self):
        return f'{self.probe.name} - {self.date} - {self.passed}'

    def get_absolute_url(self):
        return reverse_lazy('test_list')

    @staticmethod
    def create(**kwargs):
        c = Result.objects.create(probe=kwargs.get('probe'))
        c.output = kwargs.get('output')
        c.passed = kwargs.get('passed')
        c.save()
        return c

```


### Find the Page Probe Tests
* Scan the test directory
* Scan files for test functions

probe/probe.py

```python
all_probes = {}

def find_tests():

    def module_list(directory):
        return [f.replace('.py', '') for f in listdir(directory) if f.startswith('test_')]

    def test_functions(module_name):
        module = get_module(module_name)
        functions = getmembers(module, isfunction)
        return [f for f in functions if f[0].startswith('test_')]

    global all_probes

    for module_name in module_list('test'):
        for f in test_functions(module_name):
            source = '.'.join(['test', module_name, f[0]])
            all_probes[f[0]] = f[1]

```

test/test_system.py

```python
def test_system_source():
    files = len(recursive_files('.'))
    return f'System files: {files}'


def test_python_source():
    files = len(recursive_files('.', filetype='.py'))
    return f'Python files: {files}'

```


### Execute Page Probe Tests
* Fetch and examine web pages from local and remote servers
* Use "requests" module in Python to fetch pages
* Use an app to manage test cases

probe/probe.py

```python
all_probes = {}

def execute_probe(probe):

    def get_test_function(probe):
        global all_probes
        return all_probes[probe.source]

    try:
        output = get_test_function(probe)()
        passed = output == probe.expected
    except:
        output = f'Test Failed to execute:  {probe.name}'
        passed = False

    return Result.create(probe=probe, output=output, passed=passed)

```


### Views

probe/probe_views.py

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.shortcuts import render
from django.urls import reverse_lazy, reverse
from django.views.generic import CreateView, DeleteView, DetailView, ListView, RedirectView, UpdateView

from .models import Result, Test
from .probe import approve_result, clear_probe_history, execute_probe, reset_tests, result_list


class TestView(RedirectView):
    url = reverse_lazy('test_list')


class TestApproveView(RedirectView):

    def get_redirect_url(self, **kwargs):
        pk = self.kwargs.get('pk')
        result = Result.objects.get(pk=pk)
        result = approve_result(result)
        return reverse('test_detail', args=[result.probe.pk])


class TestListView(ListView):
    template_name = 'test_list.html'
    model = Test


class TestDetailView(DetailView):
    template_name = 'test_detail.html'
    model = Test

    def get_context_data(self, **kwargs):
        kwargs = super().get_context_data(**kwargs)
        probe = kwargs['object']
        # execute_probe(probe)
        kwargs['results'] = result_list(probe)
        return kwargs


class TestCreateView(LoginRequiredMixin, CreateView):
    template_name = "test_add.html"
    model = Test
    fields = '__all__'

    def form_valid(self, form):
        form.instance.author_id = 1
        return super().form_valid(form)


class TestResetView(RedirectView):

    def get_redirect_url(self, **kwargs):
        pk = self.kwargs.get('pk')
        if pk == 0:
            reset_tests()
            return reverse('test_list')
        else:
            clear_probe_history(pk)
            return reverse('test_detail', args=[pk])


class TestRunView(RedirectView):

    def get_redirect_url(self, **kwargs):
        pk = self.kwargs.get('pk')
        if pk == 0:
            for probe in Test.objects.all():
                execute_probe(probe)
            return reverse('test_list')
        else:
            probe = Test.objects.get(pk=pk)
            execute_probe(probe)
            return reverse('test_detail', args=[pk])


class TestUpdateView(LoginRequiredMixin, UpdateView):
    template_name = "test_edit.html"
    model = Test
    fields = '__all__'


class TestDeleteView(LoginRequiredMixin, DeleteView):
    model = Test
    template_name = 'test_delete.html'
    success_url = reverse_lazy('test_list')

```


### Templates


templates/probe_detail.html

```html  
{% extends 'theme.html' %}

{% block content %}

<div class="container m-5">

    <h1 class="text-primary">Test Details - {{ object.name }}</h1>

    <h2>Expected Output</h2>
    <pre>{{ object.expected }}</pre>

    <h2>Python Source Code for Test</h2>
    <pre>{{ object.source }}</pre>

    <a href="/test/" class="btn btn-success m-5">Test List</a>
    <a href="/test/{{ object.pk }}/" class="btn btn-success m-5">Edit Test</a>
    <a href="/test/{{ object.pk }}/delete" class="btn btn-success m-5">Delete Test</a>
    <a href="/test/{{ object.pk }}/run" class="btn btn-success m-5">Run Test</a>
    <a href="/test/{{ object.pk }}/reset" class="btn btn-success m-5">Clear History</a>

    <h2>Test History</h2>
    <table class="table">

        <tr>
            <th>Date</th>
            <th>Pass/Fail</th>
            <th>Text Output</th>
        </tr>
        {% for r in results %}
        <tr>
            <td>{{ r.date }}</td>
            <td>{% if r.passed %}PASS{% else %}FAIL{% endif %}</td>
            <td>{{ r.output }}</td>
            <td><a href="/test/{{ r.pk }}/approve" class="btn btn-success">Approve</a></td>
        </tr>
        {% endfor %}
    </table>

</div>

{% endblock content %}
```


### URL Routes

config/urls.py

```python
from hammer.views_probe import TestView

urlpatterns = [
    path('', TestView.as_view()),
    path('test/', include('hammer.urls_probe')),
]

```


probe/probe_urls.py

```python

from django.urls import path
from .views_probe import (TestApproveView, TestCreateView, TestDeleteView, TestDetailView,
                          TestListView, TestResetView, TestRunView, TestUpdateView)
urlpatterns = [

    # Test
    path('',                       TestListView.as_view(),      name='test_list'),
    path('<int:pk>',               TestDetailView.as_view(),    name='test_detail'),
    path('add',                    TestCreateView.as_view(),    name='test_add'),
    path('<int:pk>/',              TestUpdateView.as_view(),    name='test_edit'),
    path('<int:pk>/delete',        TestDeleteView.as_view(),    name='test_delete'),

    # Results
    path('<int:pk>/run',           TestRunView.as_view(),       name='test_run'),
    path('<int:pk>/approve',       TestApproveView.as_view(),   name='test_approve'),
    path('<int:pk>/reset',         TestResetView.as_view(),     name='test_reset'),

]

```

