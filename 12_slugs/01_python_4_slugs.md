If you've noticed in most blog platforms and blogs in general, the URL in the detail page of an article doesn't show you a number that's the `id` of the article. It instead shows you, in the URL, usually the title of the article with hyphens in place of spaces and usually also lowercased. That's called a "_slug_". I know, weird name. Having slugs in the URL also optimizes your website for search engines like Google.

---

### Slugs And Preparations

For example, let's assume we have an article called "Hello, world!" with `id` 1. The URL we'd usually have would be "`example.com/articles/1/`". If that URL is slugified (made into a slug), it would be `example.com/articles/hello-world/`. This is easier to read and understand, and helps search engines like Google understand what this page is about.

Let's create that for our website! Next, everywhere in our project where we're using the `id` to uniquely identify an article, we'll change it to use the `slug` to identify it instead.

For reasons discussed previously (see "_Automatically assigning article `author`_" section), we need to delete the database. To do that, delete the file `db.sqlite3` in your `superblog/` folder.

This time around we need to also delete the _migrations files_. This is because we're adding a new field to the model, and existing objects of that model don't have a value for this new field. So when we migrate, Django will complain and ask us to set a default value or make it optional. So to avoid those options, we can simply delete the old migrations files.

In your `articles/migrations/` folder you should see files with names starting with "000...". Probably "0001...", "0002...", and "0003..." if you've followed along with us step by step. Delete all those files inside the `articles/migrations/` folder, but make sure _not_ to delete the file called "\_\_init\_\_.py".

Now that that's out of the way, let's create _slugs_!

---

### Models

In your `models.py`, add the following field to the `Article` model:

```python
class Article(models.Model):
    [...]
    slug = models.SlugField()
```

Django models come with a built-in ready out-of-the-box `SlugField`. Pretty handy, huh?

Let's not forget to migrate the new model. In your Terminal:

```bash
(superblog_env)$ python manage.py makemigrations
(superblog_env)$ python manage.py migrate
```

Since we just deleted the database completely, let's create a superuser again since we might use it at any time:

```bash
(superblog_env)$ python manage.py createsuperuser
```

---

### Slugification (The Act Of Slugging)

Now, let's write the code to convert an article's title to a slug. Create a _new file_ called `utils.py` inside `articles/`, next to `models.py`, `views.py`, and `admin.py`:

```python
from django.template.defaultfilters import slugify
from .models import Article

def create_slug(instance):
    slug = slugify(instance.title)
    return slug
```

Now, this function will receive an article object, `slugify()` its title, and return that. ([read more about Django's `slugify()` function here](https://docs.djangoproject.com/en/2.2/ref/utils/#django.utils.text.slugify).) An example of a slugified article title is: Let's say there's an article with the title "Hello World". The slugified version of that article title is "hello-world". This version of the title is URL- and human-friendly.

However, there's an issue here. Article titles aren't unique. So if we have two articles with the exact same titles, their slugs would be identical also. The problem is when we're using slugs to go to the detail page of an article. Which article should we display if multiple articles have the same exact slug?

Let's fix that problem. Change the function to the following:

```python
def create_slug(instance, new_slug=None):
    slug = slugify(instance.title)
    if new_slug is not None:
        slug = new_slug
    qs = Article.objects.filter(slug=slug)
    if qs.exists():
        try:
            int(slug[-1])
            if "-" in slug:
                slug_list = slug.split("-")
                new_slug = "%s%s" % (slug[:-len(slug_list[-1])], int(slug_list[-1]) + 1)
            else:
                new_slug = "%s-1" % (slug)
        except:
            new_slug = "%s-1" % (slug)
        return create_slug(instance, new_slug=new_slug)
    return slug
```

The logic behind this function is the following:

Let's say we have two articles both with the title "Hello World". Their slugs will both be "hello-world". What this function does if it's making a slug that already exists it will add "-1" to the end of it. If there's already a slug "hello-world-1", it will take the number at the end and add 1 to it. So the slug will be "hello-world-2". And so on. This way each article's slug is always unique and readable.

---

### Slugs

In your `urls.py`, replace the following:

```python
urlpatterns = [
    [...]
    path('articles/<int:article_id>/', views.article_detail, name="article-detail"),
    [...]
    path('drafts/<int:article_id>/', views.draft_edit, name="draft-edit"),
    path('drafts/<int:article_id>/delete/', views.draft_delete, name="draft-delete"),
]
```

with:

```python
urlpatterns = [
    [...]
    path('articles/<slug:article_slug>/', views.article_detail, name="article-detail"),
    [...]
    path('drafts/<slug:article_slug>/', views.draft_edit, name="draft-edit"),
    path('drafts/<slug:article_slug>/delete/', views.draft_delete, name="draft-delete"),
]
```

In your `views.py`, replace the following:

```python
def article_detail(request, article_id):
    article_object = Article.objects.get(id=article_id)
    [...]
```

with:

```python
def article_detail(request, article_slug):
    article_object = Article.objects.get(slug=article_slug)
    [...]
```

Also in your `views.py`, change your `draft_edit(...)` view to:

```python
from .utils import create_slug

def draft_edit(request, article_slug):
    article = Article.objects.get(id=article_slug)
    form = ArticleForm(instance=article)
    if request.method == "POST":
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            article = form.save(commit=False)
            if form.data.get("title") != article.title:
                article.slug = create_slug(article)
            article.save()
            if 'publish' in form.data:
                article.draft = False
                article.published = now()
                article.save()
                return redirect("article-detail", article_slug=article.slug)
            return redirect('draft-list')
    context = {
        'form': form,
        'draft': article
    }
    return render(request, "draft_edit.html", context)
```

##### `article = form.save(commit=False)`

The `commit=False` bit tells Django to _not_ save the `Article` object in the database yet. The `form.save(...)` will return the `Article` object and we're storing it in a variable called `article`.

##### `if form.data.get("title") != article.title`

Here we're checking if the `title` was changed.

##### `article.slug = create_slug(article)`

This line will only run if the title was changed. It will generate a slug from the article and store it in the `article`'s `slug` field. It's important that this line is inside this if-statement. Because if it's not, assuming the article's title is "Hello World", the article's slug will be "hello-world-1", even if there isn't another article with the title "Hello World" and the title wasn't changed. Then the next time you edit the draft without changing the title, the slug will be changed to "hello-world-2". This will continue to happen. So, to avoid this, we put the if-statement so that the slug is only updated if the title is changed.

##### `article.save()`

This will save the edited article.

Next, change your `article_create(...)` view to:

```python
def article_create(request):
    form = ArticleForm()
    if request.method == "POST":
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)
            article.author = request.user
            article.slug = create_slug(article)
            article.save()
            return redirect("article-list")
    context = {
        "form": form
    }
    return render(request, 'create.html', context)
```

As explained previously, this new line sets the `slug` of the `article` to a slug.

Now we're gonna change `id`s to `slug`s everywhere we're referencing the `id` of an `article` or a `draft`. In your `views.py`, replace the following:

```python
def draft_delete(request, article_id):
    article = Article.objects.get(id=article_id)
    [...]
```

with:

```python
def draft_delete(request, article_slug):
    article = Article.objects.get(slug=article_slug)
    [...]
```

In your `draft_list.html`, change the following line:

```django
<a href="{% url 'draft-edit' draft.id %}">
```

to:

```django
<a href="{% url 'draft-edit' draft.slug %}">
```

In your `draft_edit.html`, change the following two lines:

```django
[...]
<form action="{% url 'draft-edit' draft.id %}" method="POST">
[...]
<a href="{% url 'draft-delete' draft.id %}">
[...]
```

to:

```django
[...]
<form action="{% url 'draft-edit' draft.slug %}" method="POST">
[...]
<a href="{% url 'draft-delete' draft.slug %}">
[...]
```

Lastly, in your `list.html`, change the following:

```django
<h2 class="card-title"><a href="{% url 'article-detail' article.id %}">{{ article.title }}</a></h2>
```

to:

```django
<h2 class="card-title"><a href="{% url 'article-detail' article.slug %}">{{ article.title }}</a></h2>
```

---

Commit changes to git.
