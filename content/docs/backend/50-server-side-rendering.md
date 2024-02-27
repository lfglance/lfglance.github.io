+++
title = "Server-Side Rendering"
description = "Getting content to a web browser."
date = 2024-02-11T11:40:00+00:00
draft = false
weight = 50
sort_by = "weight"
template = "docs/page.html"
slug = "server-side-rendering"

[extra]
lead = "Server-side rendering is a technique used in web development where the web server generates the HTML for a web page and sends it to your browser."
toc = true
top = false
+++

Most of today's web sites tend to be all client-side, Javascript based, with frameworks such as [React](https://react.dev/). Your browser requests a web page and is returned a small boilerplate HTML page which has references to Javascript files which the browser then downloads. Once downloaded, the browser will execute the Javascript and render the page you see on your screen by modifying the HTML with dynamic content. This is called **client-side rendering** (CSR) because the HTML rendering happens with the Javascript downloaded on your machine.

This is a pretty modern paradigm which we will plan to get into in the frontend part of the course, but I think it's important to learn the way our founding fathers did...**server-side rendering** (SSR). With SSR, the web server will generate all of the HTML and serve it to the user where the browser renders it exactly as-is; the HTML rendering happens on the server. The old-school tools to do this were things such as Perl, CGI, and later PHP. As time went on more languages began to support these features and today all modern programming languages today offer frameworks to build web services; Javascript/NodeJS/Typescript, Golang, Rust, Python, C++, Ruby, Erlang, etc.

**client-side**: _User requests page -> backend serves boilerplate HTML and compiled Javascript -> Javascript runs in user's browser to render pages (will typically query backend APIs to render data)_

**server-side**: _User requests page -> backend fetches data, generates/renders HTML templates -> serves content to user_


## HTML Templating

Pretty much every programming language you work with will have different formats for HTML templating:
* Ruby on Rails -> eRuby (erb)
* Python Flask -> Jinja
* Rust -> Tera
* etc

For the most part they are fairly similar and offer basic functionality such as looping, if statements, etc. To continue our example of our [new web service](zo) we're going to update our app to serve some HTML templates and render data into them.

Recall our example's `app.py`:

```python
from quart import Quart, request

from models import db

def create_app():
    app = Quart(__name__)

    @app.route('/')
    def hello():
        req = request.args.get('name', 'World')
        return 'Hello {}'.format(req)

    return app

run = create_app().run()
```

There's one route on this app right now and it simply returns text: `Hello World`. Let's update this example to render and return an HTML template instead:

Create a new directory called `templates` and in it a new file called `hello.html` with the following HTML content:

```html
<html>
<head>
<title>{{ title }}</title>
<body>
<h1>The current date and time is: {{ datetime }}</h1>
</body>
</html>
```

Now update your `app.py` to the following:

```python
from quart import Quart, request, render_template
from datetime import datetime

def create_app():
    app = Quart(__name__)

    @app.route('/')
    async def hello():
        return await render_template(
            'hello.html', 
            title='My Sick App', 
            datetime=datetime.utcnow()
        )

    return app

run = create_app().run()
```

A couple changes here. We're importing `render_template` from Quart; this is what will read a template HTML file and inject any data required to render.

We're also import `datetime` so we can get the current time; this will help understand server-side rendering concepts.

We change the `hello` route to make it `async`, required for the asynchronous `render_template` function.

We change the `return` statement to instead `await render_template` and pass the name of the template and the data we are passing (`await` because it's an async function).

Run `quart run` and hit the page [http://127.0.0.1:5000](http://127.0.0.1:5000).

The title of your page will be `My Sick App` and the page will now return a sentence with the date and time.

![simple server-side rendered web page](/images/50-ssr-1.png)

You'll notice that the date and time is not updating. It's just a flat text that was generated on the server as it formed it's response to your web request, and the UTC datestamp was "injected" dynamically into the HTML template in `{{ datetime }}`. The title was also injected at `{{ title }}` in the template. This is HTML rendering in an SSR app in effect.


## Adding Backend Logic

You'll often hear this called "business logic". It's the underlying code that determines what to actually return to web requests based on a number of factors such as:

* does the user have a valid session token or cookie?
* what IP address is their request from?
* what URL parameters did they provide?
* what is their browser's user-agent header?
* what headers have they specified?
* have they setup a profile and other required fields yet?
* etc

We're going to extend our app to have some logic in it's response. The simplest way to do this is to check for certain URL parameters, in this case, we'll use a param called `token`. Update `app.py` to:

```python
from quart import Quart, request, render_template
from datetime import datetime

secret_token = 'zerglingrush'

def create_app():
    app = Quart(__name__)

    @app.route('/')
    async def hello():
        req = request.args.get('token')
        if req == secret_token:
            return await render_template(
                'hello.html', 
                title='My Sick App', 
                datetime=datetime.utcnow()
            )
        else:
            return 'invalid token'

    return app

run = create_app().run()
```

We've now defined a `secret_token` at the top of our code with a secret string...`zerglingrush` (Zerg ftw). In the `hello` route, we have an `if` statement that is checking if the URL parameter matches our `secret_token`. If it matches, the app will render and return the `hello.html` HTML template. If it does not match, the app will return a simple string, `invalid token`. This is what happens when hitting the URL with no parameters (or a different one):

![](/images/50-ssr-2.png)

Using the right token now, [http://127.0.0.1:5000/?token=zerglingrush](http://127.0.0.1:5000/?token=zerglingrush) will show the previous template with the UTC datestamp. The business logic routed us to the template since we met the right criteria.

* [invalid token](http://127.0.0.1:5000/)
* [valid token](http://127.0.0.1:5000/?token=zerglingrush)


## Adding Template Logic

In addition to handling logic in the backend you can also define logic in your templates. There may be use-cases where it's easier to do this so you can decide on your own.

To do this, let's adjust our `app.py`:

```python
from quart import Quart, request, render_template
from datetime import datetime

secret_token = 'zerglingrush'

def create_app():
    app = Quart(__name__)

    @app.route('/')
    async def hello():
        req = request.args.get('token')
        return await render_template(
            'hello.html', 
            title='My Sick App', 
            datetime=datetime.utcnow(),
            secret_token=secret_token
        )

    return app

run = create_app().run()
```

Now let's adjust our `hello.html` template:

```html
<html>
<head>
<title>{{ title }}</title>
<body>
{% if request.args.token == secret_token %}
<h1>The current date and time is: {{ datetime }}</h1>
{% else %}
<h1>invalid token</h1>
{% endif %}
</body>
</html>
```

You will notice that our `app.py` no longer does any checking...it just simply renders the HTML and supplies `secret_token` to it. However, now our `hello.html` has an if statement; `{% if request.args.token == secret_token %}`. We can accomplish the same result as before but by putting the business logic into the HTML template instead.

* [invalid token](http://127.0.0.1:5000/)
* [valid token](http://127.0.0.1:5000/?token=zerglingrush)


## Extending Logic

Right now the logic we have is checking simple things and deciding which response to return. However, there's much more to be done here, such as retrieving data from various sources:

* relational data from our database
* data from upstream APIs
* objects from object storage
* file from the filesystem
* objects from cache
* etc

Let's extend our logic a little bit and fetch some company info on IBM from a free API provided by [AlphaVantage](https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=IBM&apikey=demo). We'll need another Python library to issue an HTTP call so run this command first.

```bash
.venv/bin/pip install requests
```

Then let's add a new file in `templates`, create `stock.html` and add the following:

```html
<h1>fetched: {{ fetched }}</h1>
<p>data: {{ data }}</p>
```

Then let's update `app.py` to this:

```python
import json
from pathlib import Path

import requests
from quart import Quart, render_template


endpoint = 'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=IBM&apikey=demo'
data_file = 'data.json'

def create_app():
    app = Quart(__name__)

    @app.route('/')
    async def hello():
        data = Path(data_file)
        fetched = False
        if not data.exists():
            with open(data, 'w') as f:
                fetched = True
                res = requests.get(endpoint)
                res.raise_for_status()
                text_data = json.dumps(res.json())
                f.write(text_data)

        with open(data, 'r') as f:
            text_data = json.loads(f.read())
            return await render_template(
                'stock.html',
                data=text_data,
                fetched=fetched
            )

    return app

run = create_app().run()
```

This is quite a bit different from past versions and has a lot going on so let's break it down.

* We specify an `endpoint` with the API we're going to hit (a free, public one with IBM stock data)
* We check the existence of a `data.json` file
  * If `data.json` does not exist we fetch data using the newly added `requests` library to issue an HTTP GET request to retrieve the JSON data from the API and then we save it to `data.json` on our system in a text formatted JSON payload that Python can consume
* We note in `fetched` if on this page load we have fetched the data for the first time or not
* We read the contents of `data.json` and load the text format into a Python dictionary object
* We render the `stock.html` template with the data read from `data.json` and the boolean value of `fetched`

What this means is that on the very first time we load the page, the app will fetch and write the JSON data from this API to a file on the disk. This will act as a cache of sorts to prevent hitting the API on subsequent refreshes. This is a generally a good practice but you'd want to use a cache and expiration times instead; this works for this small example. Then the app will simply read the file and render it's data in the template. 

Now check [http://127.0.0.1:5000](http://127.0.0.1:5000) to see the results. It will look like this:

<img src="/images/50-ssr-3.png" width="100%">
<img src="/images/50-ssr-4.png" width="100%">

## Wrapping Up

We've only just scratched the surface on what is possible with SSR. This is an extremely lightweight demo and just an introduction. Full fledged applications have been made around this construct and were the basis of all internet web applications for decades.

While this is generally a simpler way to build a web application, there are some downsides to it's use. The largest one is typically performance hits. In an SSR app, every single click is an HTTP request, a logic processing and template rendering in the backend, and a full HTML page reload in the user's browser. Click, load. Click, load. It's very inefficient because the entire HTML page (the DOM) is replaced on every single click. CSR apps have introduced a more efficient way to handle this by replacing only small pieces of the page and tend to "feel" much more performant.

Additionally, SSR apps out of the box lack truly dynamic page content (i.e. page updates without reloading). Once the SSR app returns it's HTML, that's it. However, this is easily worked around by incorporating Javascript into the HTML templates and/or static assets the HTML pages load. It's doable, but typically a bit more work. There are also some Javascript frameworks that bridge the gap between SSR and CSR, but those are a subject for later pages.

If you see the comments box below, that is a SSR app I wrote and have hosted on a Digital Ocean virtual machine. You can see it in action by leaving a comment below (or on any page on this site) and viewing a status page on [Comments](https://comments.fs10xer.dev). It extends the demo today by incorporating a SQLite database to store comment data, a websocket endpoint to route messages and store submitted comments, and an HTML dashboard with Javascript chart renders. It's also a pretty simple example; you can check out the code on [Github](https://github.com/lfglance/comments).