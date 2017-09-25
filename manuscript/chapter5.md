# Chapter 5 Customize Blog Post URL

In this chapter, I will talk about how to customize the permanent link of our post page. As you know, `RoutablePageMixin` can be used to add route function to blog page, so here we continue to dive into `RoutablePageMixin` to get this job done. For example, we can add publish date of the post to URL so the permanent link would seem like this `/blog/2017/06/27/post-page-1/`. Which means user can visit the same page from `/blog/2017/06/27/post-page-1/` or `/blog/post-page-1/`

### Add Date Info Into Blog Post Url

In the last post, we talked about how to use `RoutablePageMixin` and `route` decorator to make our blog page routable, now we keep moving to make the blog have more control of post permanent link.

```python
class BlogPage(RoutablePageMixin, Page):
    ......
    def get_posts(self):
        return PostPage.objects.descendant_of(self).live()

    @route(r'^(\d{4})/$')
    @route(r'^(\d{4})/(\d{2})/$')
    @route(r'^(\d{4})/(\d{2})/(\d{2})/$')
    def post_by_date(self, request, year, month=None, day=None, *args, **kwargs):
        self.posts = self.get_posts().filter(date__year=year)
        if month:
            self.posts = self.posts.filter(date__month=month)
        if day:
            self.posts = self.posts.filter(date__day=day)
        return Page.serve(self, request, *args, **kwargs)

    @route(r'^(\d{4})/(\d{2})/(\d{2})/(.+)/$')
    def post_by_date_slug(self, request, year, month, day, slug, *args, **kwargs):
        post_page = self.get_posts().filter(slug=slug).first()
        if not post_page:
            raise Http404
        return Page.serve(post_page, request, *args, **kwargs)
```

There are some points you should notice from the code above. First, many people have no idea if Wagtail support multiple routes, the answer is yes, you can add more than one decorator to the view function to make it handle different url patterns. Here we add multiple routes to `post_by_date`, make it can handle different patterns of urls. The view function can filter the posts by year, month and day of publishing date. If you want to blog application support archive, then you must be happy with this feature.

Second, you should understand what `Page.serve` do here.

> All page classes have a `serve()` method that internally calls the get_context and get_template methods and renders the template. This method is similar to a Django view function, taking a Django Request object and returning a Django Response object

Every page has `serve` method, so if we want to render some specific blog post, we can just call `post.serve` and return the response object back. **This method sounds very simple here but almost all repos on GitHub about Wagtail blog use a very complex solution by hacking the `urls.py`. Do not use urls.py if you can do it with `RoutablePageMixin`**

As you can see, in `post_by_date_slug` we first get the slug from the URL, then find the post which have the slug, if not found, we return 404 HTTP error, **if found, we call Page.serve to render the post, the first parameters passed in is the post object instead of blog page object itself, `post_page.serve(request, *args, **kwargs)` should also work here too**

### Reversing Post Urls

Now our blog application can handle url which contains the date info of blog post, `http://127.0.0.1:8000/blog/2017/06/27/post-page-1/` and `http://127.0.0.1:8000/blog/post-page-1/` would return the same HTML. What should I do if we want the links in blog page also have date info?

Since there is no direct way to generate the link, we need to create our own Django template tags now. It is not a big problem if you have no idea what is Django template tags, just follow this article step by step. 

Create directory `blog/templatetags`, create `blog/templatetags/__init__.py` in directory to make it treated as packages in python, and create `blog/templatetags/blogapp_tags.py` to edit.

```python
# -*- coding: utf-8 -*-
from django.template import Library, loader
from django.core.urlresolvers import resolve

register = Library()

@register.simple_tag()
def post_date_url(post, blog_page):
    post_date = post.date
    url = blog_page.url + blog_page.reverse_subpage(
        'post_by_date_slug',
        args=(
            post_date.year,
            '{0:02}'.format(post_date.month),
            '{0:02}'.format(post_date.day),
            post.slug,
        )
    )
    return url
```

The logic here is very simple, but you should know the basic points here. `register.simple_tag` is used to create custom template tags in Django, the variable passed to the function should be post object, and blog page object. Because we did not set name in the route decorator, so the name passed in `blog_page.reverse_subpage` is the method name.

Now edit `blog/templates/blog/blog_page.html` to call the template tags in our template.

```django
{% load wagtailcore_tags blogapp_tags %}

{% block content %}
    <h1>{{ blog_page.title }}</h1>

    <div class="intro">{{ blog_page.description }}</div>

    {% for post in posts %}
        <h2><a href="{% post_date_url post blog_page %}">{{ post.title }}</a></h2>
    {% endfor %}

{% endblock %}
```

In template we first load `blogapp_tags` to make them available in this template, then we use `post_date_url` custom tag to help us generate post urls. After work, we can see the post link in the blog page now have publish date info and we can click the post link to ask wagtai to handle the post url for us, which is awesome!

### Conclusion

In this chapter, we successfully customized the permanent link of post page, and we also learned how to create Django template tags to help us keep the template code clean and easy to manage.

If you want source code which can run in your local env directly, just use the commands below. You can also get the code of this tutorial from the [wagtail_tuto from michael-yin](https://github.com/michael-yin/wagtail_tuto) github repo. I would appreciate if you could star this project.

```bash
git clone https://github.com/michael-yin/wagtail_tuto.git
cd wagtail_tuto
git checkout 3f28b44

# setup the env from the code above

./manage.py runserver
# http://127.0.0.1:8000/blog/
```

Remember to use username `admin` and password `admin` to login in the wagtail CMS admin page.
