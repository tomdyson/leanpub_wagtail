# Chapter 1 Create Wagtail Project

Wagtail is one of the top CMS framework in Python world based on Django. In the blog post [Python CMS Framework Review: Wagtail vs django-cms](https://blog.michaelyin.info/2017/06/19/python-cms-framework-review-wagtail-vs-django-cms/) I have discussed the difference between Django-CMS and Wagtail CMS. As I said, Wagtail is very flexible and easy to customize compared with Django CMS, and I will show you how to create a standard blog from scratch using Wagtail CMS from scratch in this book.

## Install Wagtail

It is strongly recommended to install Wagtail CMS in a virtualenv which is an isolated env for Python packages. If you have no idea what is virtualenv, please install it and learn it [here](https://virtualenv.pypa.io/en/stable/), it is very easy, do not worry about that.

```bash
virtualenv wagtail_env
source wagtail_env/bin/activate
```

Now virtualenv has been activated, you can see `wagtail_env` shows up in your shell, which means now you have an isolated Python environments. Let's move on. 

```bash
pip install wagtail

# this command would create project wagtail_tuto similar to `django-admin startproject`
wagtail start wagtail_tuto

cd wagtail_tuto
./manage.py migrate
./manage.py createsuperuser
./manage.py runserver
```

Now you can go to `http://localhost:8000` to check the web page, you will see `Welcome to your new Wagtail site` message on the page.

## Project structure

In the last section, we use a built-in command ship with Wagtail to create a Wagtail project, now let's take a look at the project structure.

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

Here is the project structure of the wagtail_tuto, and this is a classic Django project, `home` is an app to help people get started quickly, `search` is an app which adds search function to this project.

First, you can notice that the `home` app here have django migration `0002_create_homepage.py`, this migration file create a page in wagtail and set it as root page of localhost site. Here is part of the code

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

Wagtail support multi-site structure, so each site should have a root page, you can also manage the site's root page later in admin page of wagtail. Here the localhost is config as default_site and the homepage which has URL `home` created in Django migration `0002_create_homepage.py`, is the home page of the localhost.

Let's continue looking at the URLs.py so we can know how the HTTP request is handled by this django app, here is the urls.py

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

As you can see, **wagtail has its own admin page, so django's classic admin page is now at `django-admin`, if you type `http://127.0.0.1:8000/admin/` in the browser, then you will get the wagtail admin. Now take a look at the last line of urlpatterns, if the url is not caught by previous specific rules, then the request will be handled by Wagtail.**

Let's put things together to figure out the workflow when we visit `http://127.0.0.1:8000`, first, Django would check if there is URL rule matched with this url if not, Django would send this request to wagtail_urls to handle. Wagtail would first check if the site value(`http://127.0.0.1` here) exists in the database, if exist, then wagtail will continue to find its root page, `00010001` here. Then wagtail will render the data. But how?

Here is quote from wagtail official doc, after you read it you can know the workflow.

> Wagtail uses normal Django templates to render each page type. By default, it will look for a template filename formed from the app and model name, separating capital letters with underscores (e.g. HomePage within the ‘home’ app becomes home/home_page.html

From the project structure above, there is `home_page.html` in templates inside `home` and we can found out the the html in it is indeed the html of `http://127.0.0.1:8000`. 

## Start to play with templates

Now we try to change template to show the tilte of `HomePage`, because the root page of localhost is this type. 

Edit the `home/templates/home/home_page.html`

```django
{% block content %}
    <h1>{{ self.title }}</h1>
{% endblock %}
```

Now revisit `http://127.0.0.1:8000/` you can see the content has changed. Wagtail has just help us pull data from db and render the html quickly, which is very simple. 

## Conclusion

In this chapter, I create a wagtail project, and talked about the project structure, the multi-site feature of wagtail, how the http request is handled by this Django project, all these basic points here are very important and I really wish you can get them before start coding. 

To help user focus on the key part, I only paste part of the source code instead of the whole file in this tutorial, If you want source code which can run in your local env directly, just use the commands below.

```bash
git clone https://github.com/michael-yin/wagtail_tuto.git
cd wagtail_tuto
git checkout 826ec1c

# setup the env from the code above
./manage.py runserver
```

Remember to use username `admin` and password `admin` to login in the wagtail CMS admin page.

