# Chapter 1 Create Wagtail Project

Wagtail is one of the top CMS frameworks in the Python world, based on Django. In my blog post [Python CMS Framework Review: Wagtail vs django-cms](https://blog.michaelyin.info/2017/06/19/python-cms-framework-review-wagtail-vs-django-cms/) I have discussed the difference between Django-CMS and Wagtail CMS. As I said there, Wagtail is very flexible and easy to customize compared with Django CMS, and in this tutorial I will show you how to create a standard blog from scratch using Wagtail CMS.

### Install Wagtail

I strongly recommend that you install Wagtail CMS in a virtualenv which is an isolated env for Python packages. If you have no idea about virtualenv, you can learn about it [here](https://virtualenv.pypa.io/en/stable/). It's easy!

```bash
virtualenv wagtail_env
source wagtail_env/bin/activate
```

Now virtualenv has been activated, you can see `wagtail_env` shows up in your shell, which means you now have an isolated Python environment. Let's move on. 

```bash
pip install wagtail

# this command will create a project wagtail_tuto similar to `Django-admin startproject`
wagtail start wagtail_tuto

cd wagtail_tuto
./manage.py migrate
./manage.py createsuperuser
./manage.py runserver
```

Now you can go to `http://localhost:8000` to check the web page, where you will see `Welcome to your new Wagtail site`.

### Project structure

In the last section, we use a built-in command shipped with Wagtail to create a Wagtail project. Now let's take a look at the project structure.

```bash
.
├── db.sqlite3
├── home
│   ├── __init__.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── 0002_create_homepage.py
│   │   └── __init__.py
│   ├── models.py
│   └── templates
│       └── home
│           └── home_page.html
├── manage.py
├── requirements.txt
├── search
│   ├── __init__.py
│   ├── templates
│   │   └── search
│   │       └── search.html
│   └── views.py
└── wagtail_tuto
    ├── __init__.py
    ├── settings
    │   ├── __init__.py
    │   ├── base.py
    │   ├── dev.py
    │   └── production.py
    ├── static
    │   ├── css
    │   │   └── wagtail_tuto.css
    │   └── js
    │       └── wagtail_tuto.js
    ├── templates
    │   ├── 404.html
    │   ├── 500.html
    │   └── base.html
    ├── urls.py
    └── wsgi.py
```

Here is the project structure of the wagtail_tuto, which is a classic Django project. `home` is an app to help people get started quickly, `search` is an app which adds a search function to the project.

First, you will notice that the `home` app here has Django migration `0002_create_homepage.py`. This migration file creates a page in Wagtail and sets it as the root page of the site. Here is part of the code:

```python
def create_homepage(apps, schema_editor):
    # Get models
    ContentType = apps.get_model('contenttypes.ContentType')
    Page = apps.get_model('wagtailcore.Page')
    Site = apps.get_model('wagtailcore.Site')
    HomePage = apps.get_model('home.HomePage')

    # Delete the default homepage
    # If migration is run multiple times, it may have already been deleted
    Page.objects.filter(id=2).delete()

    # Create content type for homepage model
    homepage_content_type, __ = ContentType.objects.get_or_create(
        model='homepage', app_label='home')

    # Create a new homepage
    homepage = HomePage.objects.create(
        title="Home",
        slug='home',
        content_type=homepage_content_type,
        path='00010001',
        depth=2,
        numchild=0,
        url_path='/home/',
    )

    # Create a site with the new homepage set as the root
    Site.objects.create(
        hostname='localhost', root_page=homepage, is_default_site=True)
```

Wagtail supports a multi-site structure, in which each site has a root page. You can manage the site's root page later in Wagtail's admin interface. Here the localhost is configured as default_site and the homepage - which has URL `home` created in Django migration `0002_create_homepage.py` - is the home page of the localhost.

Let's continue looking at `urls.py`, so we can see how the HTTP request is handled by this Django app:

```python
urlpatterns = [
    url(r'^django-admin/', include(admin.site.urls)),

    url(r'^admin/', include(wagtailadmin_urls)),
    url(r'^documents/', include(wagtaildocs_urls)),

    url(r'^search/$', search_views.search, name='search'),

    # For anything not caught by a more specific rule above, hand over to
    # Wagtail's page serving mechanism. This should be the last pattern in
    # the list:
    url(r'', include(wagtail_urls)),
]
```

As you can see, **wagtail has its own admin page, so Django's classic admin page is now at `django-admin`. If you type `http://127.0.0.1:8000/admin/` in the browser, then you will see the Wagtail admin. Now take a look at the last line of urlpatterns; if the url is not caught by previous specific rules, then the request will be handled by Wagtail.**

Let's put things together to figure out the workflow when we visit `http://127.0.0.1:8000`. First, Django will check if there is URL rule matched with this request. If not, Django will send this request to wagtail_urls to handle. Wagtail will first check if the site (`http://127.0.0.1` here) exists in the database; if it exists, Wagtail will continue to find its root page, `00010001` here. Then Wagtail will render the data. But how?

Here is a quote from Wagtail's official documentation:

> Wagtail uses normal Django templates to render each page type. By default, it will look for a template filename formed from the app and model name, separating capital letters with underscores (e.g. HomePage within the ‘home’ app becomes home/home_page.html

From the project structure above, there is `home_page.html` in templates inside `home` and we can see the HTML it contains is indeed the HTML we see at `http://127.0.0.1:8000`. 

### Start to play with templates

Now we'll try to change the template to show the title of `HomePage`, because the root page of localhost is this type. 

Edit the `home/templates/home/home_page.html`:

```django
{% block content %}
    <h1>{{ self.title }}</h1>
{% endblock %}
```

Now revisit `http://127.0.0.1:8000/` and you'll see the content has changed. Wagtail has just helped us pull data from the database and render the HTML quickly, which is very simple. 

### Conclusion

In this chapter, I created a Wagtail project, explained the project structure, the multi-site feature of wagtail and how the HTTP request is handled by the Django project. All these basic points are very important and I really hope you can get them before you start coding. 

To help user focus on the key part, I only pasted part of the source code instead of the whole file in this tutorial. If you want source code which can run in your local env directly, just use the commands below:

```bash
git clone https://github.com/michael-yin/wagtail_tuto.git
cd wagtail_tuto
git checkout 826ec1c

# setup virtualenv
pip install -r requirements.txt

./manage.py runserver
```

Remember to use the username `admin` and the password `admin` to log into the Wagtail admin page.
