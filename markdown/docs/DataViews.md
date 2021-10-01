# Common Data Views

### Development Process
* Create a project
* Edit settings.py
* Add an app
* Define Templates
* Define Views
* Define Routes


### Django View Classes
* TemplateView
* ListView
* DetailView
* CreateView
* UpdateView
* DeleteView


### Create a new Django project
* The default view should contain a list of link to superheroes
* An add button will let users add a new record


### ListView
* Create a table or divs that show a list of records
* Each hero should have a link that goes to the details page


### DetailView
* Display all info from the Database records
* Show the image as a thumbnail with a link to the large image
* Add a button to Edit the record


### CreateView
* Create new records with a view
* You can cheat by loading the image file into a directory
* Add the image as a URL pointing to this file


### UpdateView
* Make sure that you can edit the records
* Make sure that the page is redirected after save


### DeleteView
* Delete the records after confirmation
* Go to the list after a delete


### View Inheritance
* Create a base page template
* Style all pages without duplicating any code
