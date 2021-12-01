# Page Probe Test Design Pattern


### Demo Code
* Clone my repo and study the code in 
[Page Probe Source Code](https://github.com/Mark-Seaman/BACS350/tree/main/week14/PageProbe)
* Build the code and experiment with it
* Set up tests for your favorite web sites
* Do it simple; do it now!


### Data

probe/models.py

```python
from django.db import models
from django.urls.base import reverse_lazy


class Probe(models.Model):

    name = models.CharField(max_length=100)
    page = models.URLField()
    text = models.CharField(max_length=100)

    def __str__(self):
        return f'{self.name} - {self.page}'

    def get_absolute_url(self):
        return reverse_lazy('probe_list')

    @staticmethod
    def create(**kwargs):
        c = Probe.objects.get_or_create(name=kwargs.get('name'))[0]
        c.page = kwargs.get('page')
        c.text = kwargs.get('text')
        c.save()
        return c


class Result(models.Model):

    probe = models.ForeignKey(Probe, on_delete=models.CASCADE, editable=False)
    date = models.DateTimeField(auto_now=True)
    output = models.TextField(default='Not yet run')
    passed = models.BooleanField(default=False)

    def __str__(self):
        return f'{self.probe.name} - {self.date} - {self.passed}'

    def get_absolute_url(self):
        return reverse_lazy('probe_list')

    @staticmethod
    def create(**kwargs):
        c = Result.objects.create(probe=kwargs.get('probe'))
        c.output = kwargs.get('output')
        c.passed = kwargs.get('passed')
        c.save()
        return c

```


### Execute Page Probe Tests
* Fetch and examine web pages from local and remote servers
* Use "requests" module in Python to fetch pages
* Use an app to manage test cases

probe/probe.py

```python
def launch_all_probes():
    for probe in Probe.objects.all():
        execute_probe(probe)

def execute_probe(probe):
    try:
        response = get(probe.page)
    except:
        response = None
    if not response:
        status = f'Status Code: Domain not found,  {probe.page}'
        passed = False
    elif response.status_code != 200:
        status = f'Status Code: {response.status_code}'
        passed = False
    elif probe.text not in response.text:
        status = f'Text not found: {probe.text}'
        passed = False
    else:
        status = f'Matched: {probe.text}'
        passed = True
    Result.create(probe=probe, output=status, passed=passed)
```


### Views

probe/probe_views.py

```python
from django.shortcuts import render
from django.urls import reverse_lazy, reverse
from django.views.generic import (CreateView, DeleteView, DetailView, ListView,
                                  RedirectView, UpdateView)
from requests import get

from .models import Probe
from .probe import clear_probe_history, execute_probe, launch_all_probes, result_list


class ProbeView(RedirectView):
    url = reverse_lazy('probe_list')


class ProbeListView(ListView):
    template_name = 'probe_list.html'
    model = Probe


class ProbeDetailView(DetailView):
    template_name = 'probe_detail.html'
    model = Probe

    def get_context_data(self, **kwargs):
        kwargs = super().get_context_data(**kwargs)
        probe = kwargs['object']
        execute_probe(probe)
        kwargs['results'] = result_list(probe)
        return kwargs


class ProbeLaunchView(RedirectView):

    def get_redirect_url(self, *args, **kwargs):
        launch_all_probes()
        return reverse('probe_list')


class ProbeClearView(RedirectView):

    def get_redirect_url(self, *args, **kwargs):
        probe_pk = self.kwargs.get('pk')
        clear_probe_history(probe_pk)
        return reverse('probe_detail', args=[probe_pk])


class ProbeCreateView(CreateView):
    template_name = "probe_add.html"
    model = Probe
    fields = '__all__'

    def form_valid(self, form):
        form.instance.author_id = 1
        return super().form_valid(form)


class ProbeUpdateView(UpdateView):
    template_name = "probe_edit.html"
    model = Probe
    fields = '__all__'


class ProbeDeleteView(DeleteView):
    model = Probe
    template_name = 'probe_delete.html'
    success_url = reverse_lazy('probe_list')

```


### Templates


templates/probe_detail.html

```html  
{% extends 'theme.html' %}

{% block content %}

<div class="container m-5">

    <h1 class="text-primary">Probe Details</h1>

    <div class="row">

        <div class="col-lg">

            <table class="table">
                <tr>
                    <td>Name</td>
                    <td><b><a href="/probe/{{ object.pk }}/">{{ object.name }}</a></b></td>
                </tr>
                <tr>
                    <td>Page URL</td>
                    <td><b><a href="{{ object.page }}" target="page">{{ object.page }}</a></b></td>
                </tr>
                <tr>
                    <td>Text to Match</td>
                    <td><b>{{ object.text }}</b></td>
                </tr>
            </table>

            <h2>Test Results</h2>
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
                </tr>
                {% endfor %}
            </table>

        </div>

    </div>


    <a href="/probe/" class="btn btn-success m-5">Probe List</a>
    <a href="/probe/{{ object.pk }}/clear" class="btn btn-success m-5">Clear History</a>
    <a href="/probe/{{ object.pk }}/" class="btn btn-success m-5">Edit Probe</a>
    <a href="/probe/{{ object.pk }}/delete" class="btn btn-success m-5">Delete Probe</a>
</div>

{% endblock content %}
```


### URL Routes

config/urls.py

```python
from django.urls import path
from django.urls.conf import include

from probe.views_probe import ProbeView

urlpatterns = [

    path('', ProbeView.as_view()),
    path('probe/', include('probe.urls_probe')),

]
```


probe/probe_urls.py

```python
from django.urls import path
from .views_probe import (ProbeClearView, ProbeCreateView, ProbeDeleteView,
                          ProbeDetailView, ProbeLaunchView, ProbeListView,
                          ProbeUpdateView)
urlpatterns = [

    path('',                       ProbeListView.as_view(),    name='probe_list'),
    path('<int:pk>',               ProbeDetailView.as_view(),  name='probe_detail'),
    path('add',                    ProbeCreateView.as_view(),  name='probe_add'),
    path('<int:pk>/',              ProbeUpdateView.as_view(),  name='probe_edit'),
    path('<int:pk>/delete',        ProbeDeleteView.as_view(),  name='probe_delete'),
    path('<int:pk>/clear',         ProbeClearView.as_view(),   name='probe_clear'),
    path('test',                   ProbeLaunchView.as_view(),  name='probe_launch'),

]
```

