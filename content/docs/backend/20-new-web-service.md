+++
title = "Setting up a new web service"
description = ""
date = 2023-12-10T08:00:00+00:00
draft = false
weight = 20
sort_by = "weight"
template = "docs/page.html"
slug = "new-web-service"

[extra]
lead = 'yo.'
toc = true
top = false
+++

For this example we are going to use a web service framework called [Quart](https://github.com/pallets/quart). It is essentially async [Flask](https://flask.palletsprojects.com/), one of the most popular Python based frameworks besides Django.

## Why Python?!

A lot of software developers dislike Python because it is not statically typed and fairly "loose" with standards; it's considered a "normie" language. However, it's fairly simple and great for normies like me to wrap their brains around a lot of these concepts.

## The Setup

I could have you clone something but it's better if you do it from scratch so you get familiarity with build tools, setup, and dependencies.

Create a new directory in your terminal (use BASH - /bin/bash):

```bash
mkdir ~/fs10xer  && cd ~/fs10xer
```

Initialize the directory as a new Git repository and set a .gitignore to not track junk files (I'll explain later):

```bash
git init .
echo .venv > .gitignore
echo .env >> .gitignore
```

Create a new Python virtual environment and activate it (it sets your terminal up to restrict libraries to this directory):

```bash
python3 -m venv .venv
source .venv/bin/activate
```

You've now got a directory which will be home to this `fs10xer` app's libraries. We don't commit that into version control because A, it's not applicable to other people's machines and directory paths, and B, it's too much data; others will need to download the libraries your app needs themselves.

Install your first library, Quart, using the built-in Python library installer tool; `pip`:

```bash
pip install quart==0.18.3
```

You'll see a bunch of output like this:

```
Collecting quart==0.19.4
  Downloading Quart-0.19.4-py3-none-any.whl (100 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.2/100.2 KB 3.7 MB/s eta 0:00:00
Collecting markupsafe
  Using cached MarkupSafe-2.1.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
```

You might notice not just Quart being installed but several other libraries. This is because Quart has it's own list of libraries it requires. Your libraries have libraries and you need all of them locally to develop.

You may also notice that I had you install a very specific version of Quart (`0.19.4`). When developing applications you will want to ensure you are noting the versions of libraries in use to avoid dependency issues when others are developing on their own systems.


## Hello World 

Let's do the first thing you do for any new language; write a "Hello World" app.

Start with a new file in your fs10xer repo directory, `app.py`

```bash
touch app.py
```

Put the following contents into `app.py`:

```python
from quart import Quart

def create_app():
    app = Quart(__name__)

    @app.route('/')
    def hello():
        return 'hello world'

    return app

run = create_app().run()
```

This code is importing the Quart library and setting up a basic web service.

Now run the app with `quart`; ensure you've activate your virtual environment (`source .venv/bin/activate`). Without the virtual environment set the `quart` command would not be found as it's not in your system `$PATH`, it's in your venv `$PATH`:

```bash
quart run
```

You should see something like this:

![](/quart_run.png)

If so, congrats, you've just run a backend web service!

Open a web browser and navigate to http://127.0.0.1:5000 to see your web page and "hello world" response.

![](/hello_world.png)

## Extending

The app so far is basic as hell. It has one route "/" and returns a simple string. No HTML, CSS, Javascript, APIs, etc.

Let's make a minor adjustment so you can get a glimpse at how the server side rendering will interact with the browser. Modify the contents of your `app.py` to this:

```
from quart import Quart, request

def create_app():
    app = Quart(__name__)

    @app.route('/')
    def hello():
        req = request.args.get('name', 'World')
        return 'Hello {}'.format(req)

    return app

run = create_app().run()
```

Spot the differences. We include a new component `request`, which will allow us to interact with the HTTP request data (forms, query strings, headers, etc).

We also reference `request.args` which will retrieve a query string. `request.args.get('name', 'World')` will retrieve the query string `name`, and if not found/defined, will fallback to `World`.

Finally, in our `return` statement we use a formatted string to inject the `name` query string value (variable `req`) into the respose.

Run `quart run` again, except this time navigate in your browser to [http://127.0.0.1:5000/?name=10xer](http://127.0.0.1:5000/?name=10xer)

You'll see "Hello 10xer" on the page. Try changing the name to other things and refresh the page.

---

We will cover server side rendering in greater detail in a future section as we have more data available to insert into pages and start using HTML templates.

At this time you'll have a basic blueprint for the app and we will start morphing from a simple hello world into something more useful.