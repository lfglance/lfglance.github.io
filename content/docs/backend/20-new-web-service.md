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
lead = "Let's make a simple web app."
toc = true
top = false
+++

For this example we are going to use a web service framework called [Quart](https://github.com/pallets/quart). It is essentially async [Flask](https://flask.palletsprojects.com/), one of the most popular Python based frameworks besides Django.

## Why Python?!

A lot of software developers dislike Python because it is not statically typed and fairly "loose" with standards; it's considered a "normie" language. However, it's fairly simple and great for normies like me to wrap their brains around a lot of these concepts.

## Installing

Whatever programming language you are working with you will need the core software installed. Most languages will need the binaries (compiled software) to be installed to your machine and available in your terminal's `$PATH`. For example:
* [Golang](https://go.dev/doc/install)
* [Ruby](https://www.ruby-lang.org/en/documentation/installation/)
* [Rust](https://www.rust-lang.org/tools/install)
* [Java](https://www.java.com/en/download/help/linux_x64_install.html)
* [Python](https://www.python.org/downloads/)
* [PHP](https://www.php.net/manual/en/install.php)

Since we're using Python for this course so you need to install Python from the above link. I'm using `3.10.12` (Ubuntu 22 default, pre-installed). MacOS also comes with `python3` out of the box but you have to run `xcode-select --install` first. After you've installed `python3` to your system, double-check it's available in your `$PATH` like so:

```bash
python3 -V
> Python 3.10.12
```

Anything `Python 3.*` should be fine. Let me know if it's not.

## The Setup

I could have you clone something but it's better if you do it from scratch so you get familiarity with build tools, setup, and dependencies.

Create a new directory in your terminal (use BASH - /bin/bash):

```bash
mkdir ~/fs10xer && cd ~/fs10xer
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

## Saving dependencies

You may notice that I had you install a very specific version of Quart (`0.19.4`). When developing applications you will want to ensure you are noting the versions of libraries in use to avoid dependency issues when others are developing on their own systems.

For Python, the way we do that is using `pip` again:

```bash
pip freeze | tee requirements.txt
```

The `freeze` argument will print all of the installed libraries in your current virtual environment. `| tee requirements.txt` will store that output into a file and print it so you can see all the libraries.

## Committing Changes

So far we will have created the following:

* `.venv` - folder for our virtual environment settings and libraries
* `requirements.txt` - file containing our installed libraries and versions
* `.gitignore` - file containing things we do not want stored in Git

Git is fairly complex, but don't worry, you can keep it simple. Run the following:

```bash
git add -A  # stages all files (-A) for committing
git commit -m "adding initial repo items"  # creates a commit with given info message
```

These git commands basically make point in time snapshot of the state of this project repo. Later on we'll actually push it to a Git provider like Github to store it. Right now it's fine to just keep locally.

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

## Commit That

Run the Git commands again to "save" this point in time snapshot.

```bash
git add app.py && git commit -m "adding hello world app"
```

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

## Commit That

```bash
git add app.py && git commit -m "extend app query strings"
```

## See Your Commit History

If you've run the Git commands along the way you'll be able to see your past commits and thus will be able to roll back into any point in time. Run:

```bash
git log
```

If you wanted to rollback, there are several ways to go about it, but a common one would be to run `git checkout $COMMIT_HASH` where $COMMIT_HASH is the long string you see next to your commits. Here's the commit log for this project's repo:

![](/git_log_example.png)

---

We will cover server side rendering in greater detail in a future section as we have more data available to insert into pages and start using HTML templates.

At this time you'll have a basic blueprint for the app and we will start morphing from a simple hello world into something more useful.