This page is the trello card "_As a user, I can login to my account._" So let's move this card from Backlog to Doing.

---

### Login Form

The login page consists essentially of a form with username and password fields. In your `forms.py`:

```python
class LoginForm(forms.Form):
    username = forms.CharField(required=True)
    password = forms.CharField(required=True, widget=forms.PasswordInput())
```

In Django, we can define our forms as Python classes that subclass Django's built-in `Form` class. Let's dissect this code!

##### `class LoginForm(forms.Form):`

Here we're defining a form that we will later use on our website, and we're calling this form "`LoginForm`". What makes this a form, not just a Python class, is that it subclasses Django's `Form` class.

([read more about Django's Form](https://docs.djangoproject.com/en/2.2/topics/class-based-views/generic-editing/))

##### `username = forms.CharField(required=True)`

Here we're defining, as part of this form, a field called "`username`", and the type of data this form field will have is a string, hence the familiar `CharField` bit. We're also saying that the username field in this form is _required_, hence the `required=True` bit.

##### `password = forms.CharField(required=True, widget=forms.PasswordInput())`

This is the same as the `username`, but we're calling it `password` and it has the `PasswordInput()` widget. For a normal `Form` class (not a `ModelForm`), this is how you specify a widget for a field.

---

### URLs

In your `urls.py`, let's define the URL path for the login page:

```python
urlpatterns = [
    [...]
    path('login/', views.login_view, name="login"),
]
```

---

### Login Page

Let's start with the view. In your `views.py`:

```python
from .forms import LoginForm

def login_view(request):
    form = LoginForm()
    context = {
        'form': form
    }
    return render(request, "login.html", context)
```

Here we're importing the `LoginForm` and creating an object out of the form class, then sending the object to the `login.html` template.

In your `login.html`:

```django
{% extends "base.html" %}

{% block title %}
Login Page
{% endblock %}

{% block body %}
<form action="{% url 'login' %}" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit">
</form>
{% endblock %}
```

Here we're rendering the login form along with a submit button. Once submitted, the URL it submits to is the login URL.

Now, if you go to `127.0.0.1:8000/login`, you'll see the login form. If you fill it and submit it won't do anything. We need to receive the form submission in the views.

In your `views.py`, change your `login_view()` to:

```python
from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate

def login_view(request):
    form = LoginForm()
    if request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            username = form.cleaned_data['username']
            password = form.cleaned_data['password']

            auth_user = authenticate(username=username, password=password)
            if auth_user is not None:
                login(request, auth_user)
                return redirect('article-list')

    context = {
        'form': form
    }
    return render(request, "login.html", context)
```

Here we added the code that responds to the `POST` request, which handles it by validating user's credentials, and logging them in if they're valid and correct. Most of this code is familiar to you, from the registration view. But there's a bit of code there that's new to you. Let's dissect this code!

##### `username = form.cleaned_data['username']`

If you recall, this `LoginForm` we're using is _not_ a `ModelForm`. It's a basic form. We can get the data the user entered in the form by using the dictionary `cleaned_data` that comes built-in to the form. This dictionary has keys that match the fields in the form that we defined. It has key `username`, its value is whatever the user entered as the username in the form. Same for password.

Here we're taking the username the user entered into the form and storing it in a variable called `username`.

##### `password = form.cleaned_data['password']`

Here we're taking the password the user entered into the form and storing it in a variable called `password`.

##### `auth_user = authenticate(username=username, password=password)`

This line checks the validity of the username/password combination the user entered. It does that by calling Django's built-in `authenticate()` function, and giving it the username and password the user typed. It returns `None` if the username/password combination is incorrect, and returns the `User` object if the combination is correct.

##### `if auth_user is not None:`

Here we're checking if the user enter a valid username/password combination. Because, again, the call to `authenticate()` returns _`None`_ if the user entered an invalid username/password combination.

##### `login(request, auth_user)`

This line is familiar to you. It'll log the user in with the credentials they entered to the form.

##### `return redirect('article-list')`

This line will redirect the user to the article list page if they've successfully logged in.

---

### Base Template

Let's add a button in the base template to take us to the login page. In your `base.html`, under the "Register" button:

```django
<body>
    <a href="{% url 'register' %}">
        <button>
            Register
        </button>
    </a>
    <a href="{% url 'login' %}">
        <button>
            Login
        </button>
    </a>
    {% block body %}{% endblock %}
</body>
```

Now you can click on the login button and be taken to the login page.

Next we're gonna build the registration page!

---

### Git & Trello

Move the trello card from Doing to Done. Commit changes to git.
