Let's put a button at the bottom of the article detail page for sharing this article. A basic sharing functionality, in this case, means to copy the URL of the article so that the user can just paste it elsewhere to share it. So we're gonna have a share button that copies the URL of this article to clipboard, as if the user selected the URL from the URL bar and copied it. This button will copy it for them.

[Click here](https://superblog.codeunicorn.io/articles/hoomans/) for an example on how this button will work. In this article (written by the brilliant author _noug_), you can see the share button, once you click it it'll copy the URL of that webpage to your clipboard. Then you can just paste it anywhere.

This is the trello card "_As a user, I can share a post by sharing a link to it._" Move the card from Backlog to Done.

In your `detail.html`, at the bottom of the body block:

```django
{% block body %}
    [...]
    <button onclick="copyToClipboard('{{ request.build_absolute_uri }}')">
        Share Link
    </button>

    <script>
        const copyToClipboard = str => {
            const el = document.createElement('textarea');
            el.value = str;
            el.setAttribute('readonly', '');
            el.style.position = 'absolute';
            el.style.left = '-9999px';
            document.body.appendChild(el);
            el.select();
            document.execCommand('copy');
            document.body.removeChild(el);
            $("#poppy").popover("toggle");
        };
    </script>
{% endblock %}
```

The `copyToClipboard()` function is what does the actual copying into clipboard. It does this by adding a hidden element to the page and make its value be the string it receives (which is the URL of the page), selecting that text and copying it, then removes it from the page. We have a button that says "Share Link" that if clicked will call this function to copy to clipboard. You can read the JavaScript code to try and understand how it works.

##### `{{ request.build_absolute_uri }}`

This will return the absolute URL of this current page. So if the article's title is "Hello World", this will return "http://127.0.0.1:8000/articles/hello-world/". Of course once deployed to a web server with a domain name, this URL will automatically be changed to "http://<YOUR_DOMAIN_NAME>.com/articles/hello-world/". For example: "http://example.com/articles/hello-world/".

[Read more about `request.build_absolute_uri` here](https://docs.djangoproject.com/en/2.2/ref/request-response/#django.http.HttpRequest.build_absolute_uri)

---

Move the card to Done, and commit changes to git.
