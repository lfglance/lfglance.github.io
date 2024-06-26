+++
title = "Handling User Inputs"
description = "How to receive, sanitize, and accept data from your users."
date = 2023-12-10T08:00:00+00:00
draft = false
weight = 40
sort_by = "weight"
template = "docs/page.html"
slug = "user-inputs"

[extra]
lead = 'You typically need to be able to safely accept data from users in order for your web service to function properly.'
toc = true
top = false
+++

## Overview

Usernames, passwords, email addresses, birthdays, etc...You eventually will need to be able to take some data from a client and do something with it in the backend. This is typically done with HTML form fields or AJAX requests. These are the most common ways of taking user inputs:

* HTML form GET which sends the form inputs as query strings / URL parameters (i.e. http://site.com/?name=john&age=30)
* HTML form POST which sends the form inputs as a form encoded data payload in the body of the request
* Javascript AJAX requests (i.e. `fetch`) which sends GET or POST requests with data similar to above, sometimes as JSON payloads, to RPC, REST, or other form/data accepting endpoints

The most critical thing to keep in mind with accepting user inputs is to sanitize the data they are providing and **triple-checking** the data they are providing, ensuring compliance and consistency with the type of data you are intending to keep.

This is required to be done on the backend with strong business logic conditions and while it can be guided in the frontend, you cannot guarantee client-side validation as the client-side can be modified. 

**Assume any data you receive is malicious and author your backend as such.**

## Simple Example

Let's setup a small Flask app and render a simple HTML form.

```bash
# go to a temporary directory
cd $(mktemp -d)
python3 -m venv .venv
.venv/bin/pip install flask
```

### app.py

Add an `app.py` file with the following contents:

```python
from flask import Flask, request, render_template, jsonify

def create_app():
    app = Flask(__name__, template_folder='.')

    @app.route('/', methods=['GET', 'POST'])
    def index():
        return render_template(
            'index.html',
            form=request.form,
            args=request.args
        )
    
    @app.route('/api/form', methods=['GET', 'POST'])
    def api():
        data = {'data': {}, 'message': f'Successful {request.method} received'}
        data_source = None
        if request.args:
            data_source = request.args
        elif request.form:
            data_source = request.form
        else:
             return jsonify({'message': 'invalid data payload'}), 400
        for f in data_source:
                data['data'][f] = data_source.get(f)
        return jsonify(data)

    return app

if __name__ == '__main__':
    create_app().run(reload=True)
```

### index.html

Add an `index.html` file with the following contents:

```html
<html>
    <body>

        <h3>Results</h3>
        <p id="results"></p>

        {% if form %}
        <p>Form post found: </p>
        <ul>
            {% for arg in form %}
            <li>{{ arg }} -> {{ form[arg] if form[arg] else "null" }}</li>
            {% endfor %}
        </ul>
        {% endif %}

        {% if args %}
        <p>Request arguments (query strings / url parameters) found: </p>
        <ul>
            {% for arg in args %}
            <li>{{ arg }} -> {{ args[arg] if args[arg] else "null" }}</li>
            {% endfor %}
        </ul>
        {% endif %}

        <hr>

        <div>
            <h2>Form GET</h2>
            <form>
                <label for="name">Enter your name: </label>
                <input name="name" placeholder="hank hill" />
                <label for="age">Enter your age: </label>
                <input name="age" type="number" />
                <input type="submit" />
            </form>
        </div>

        <div>
            <h2>Form POST</h2>
            <form method="post" action="/">
                <label for="name">Enter your name: </label>
                <input name="name" placeholder="hank hill" />
                <label for="age">Enter your age: </label>
                <input name="age" type="number" />
                <input type="submit" />
            </form>
        </div>

        <div>
            <h2>AJAX GET</h2>
            <form id="getForm">
                <label for="name">Enter your name: </label>
                <input name="name" placeholder="hank hill" />
                <label for="age">Enter your age: </label>
                <input name="age" type="number" />
                <input type="submit" />
            </form>
        </div>

        <div>
            <h2>AJAX POST</h2>
            <form id="postForm">
                <label for="name">Enter your name: </label>
                <input name="name" placeholder="hank hill" />
                <label for="age">Enter your age: </label>
                <input name="age" type="number" />
                <input type="submit" />
            </form>
        </div>

        <script>
            // listen for GET form submission for AJAX request
            document.getElementById("getForm").addEventListener("submit", function(e) {
                e.preventDefault();
                const formData = new FormData(e.target);
                const params = new URLSearchParams();

                for (const [key, value] of formData.entries()) {
                    params.append(key, value);
                }

                const url = `/api/form?${params.toString()}`;

                fetch(url, {
                    method: 'GET'
                })
                    .then(response => response.json())
                    .then(data => {
                        console.log('Success:', data);
                        document.getElementById("results").innerHTML = "AJAX GET found: <br /><br />" + JSON.stringify(data);
                    })
                    .catch((error) => {
                        console.error('Error:', error);
                    });
            });

            // listen for POST form submission for AJAX request
            document.getElementById("postForm").addEventListener("submit", function(e) {
                e.preventDefault();
                const formData = new FormData(e.target);
                const url = `/api/form`;

                fetch(url, {
                    method: 'POST',
                    body: formData
                })
                    .then(response => response.json())
                    .then(data => {
                        console.log('Success:', data);
                        document.getElementById("results").innerHTML = "AJAX POST found: <br/ ><br /> " + JSON.stringify(data);

                    })
                    .catch((error) => {
                        console.error('Error:', error);
                    });
            });
        </script>

    </body>
</html>
```

### Running

Once the files are present run the Flask app like so:

```bash
.venv/bin/flask run
```

Open it at [http://127.0.0.1:5000](http://127.0.0.1:5000)

### Testing 

This simple Flask app will two endpoints: 
* `/` - accepts form data as either a GET or POST request and passes the data into the HTML template. The HTML template has some simple forms, one for GETs, one for POSTs.
* `/api/form` - accepts GET or POST data and returns a JSON payload, simulates an RPC endpoint (what some people might call REST API though that's not technically correct)

Try submitting forms and seeing the results that get rendered in the HTML page.

![](/inputs_example.png)

## Security Concerns

There are a ton of ways that threat actors can break your web application; most pertain to poor logic conditions, error handling, and lack of sanitization. You can't assume that everyone will use the application as you intended and even non-malicious people can cause problems.

Here are some of the most common attacks - a more comprehensive list can be found on the internet, and the most common starting point for security defense is the [OWASP Top 10](https://owasp.org/www-project-top-ten/):

* SQL inject (SQLi) - injection attacks are possible when the backend logic does not properly sanitize incoming data and passes it as SQL statements which have logic to escape the query for an attacker to form their own (i.e. `'; DROP TABLE users; --`)
* Server Side Request Forgery (SSRF) - the attacker tricks the server into making HTTP requests to an arbitrary domain of the attacker's choosing and the backend logic lacks conditions to prevent it (i.e. `http://example.com/check?url=http://localhost/phpmyadmin`)
* Remote Code Execution (RCE) - when an attacker is able to execute arbitrary commands on the host operating system via a vulnerable application which makes system shell commands; the backend lacks the logic to sanitize the input (i.e. `http://example.com/ping?host="127.0.0.1; rm -rf /;"`)
* File Upload Vulnerabilities - when an attacker is able to upload a malicious file like a reverse shell or virus to distribute (i.e. `http://example.com/upload` and lack of file checks)
* Directory Traversal - allows an attacker to access directories and files stored outside the web root folder if the backend allows file path arguments (i.e. `http://example.com/read?path=../../../../../etc/passwd` or `http://example.com/read?path=../../.env`)

Again, this is not a comprehensive list, but some examples of pretty damaging things that can happen if you're not properly developing your backend to accept user input.

Our simple example above does not contain anything malicious, but consider some of the logic that could be enforced:
* is the age appropriate? not a negative number? over or under a realistic amount?
* is the age an actual number or a string/text?
* is the name appropriate?
* is the amount of characters expected?

Outside of our example, consider a normal CRUD application with registration, logins, storing data, retrieving data:
* does the user have permission to retrieve this record?
* does the user have an active session or are they authenticated?
* does the object exist already?
* will the user be able to define things that the application is not expecting?
* is the data type valid? string, integer, boolean?
* has the user presented a valid option?

You have to consider every way which data could be presented to your application and ensure your logic will prevent unintended data collection.

## Advanced Example

In a [previous page](../database-schemas) we were modeling a web service to capture pullups and pushups. I've developed a simple web service to do exactly this: [pullup-counter](https://github.com/lfglance/pullup-counter)

The below commands should have it up and running at [http://127.0.0.1:5000](http://127.0.0.1:5000).

```bash
cd $(mktemp -d)
git clone https://github.com/lfglance/pullup-counter.git .
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/flask -A app/main.py run --debug
```

![](https://cdn.fs10xer.dev/updogs.gif)

<style>
    img { 
        max-width: 700px;
    }
</style>

This example incorporates a SQLite database to store data and some RPC endpoints to generate and delete data from it. Read through the code to look at how this is implemented.

There are currently no vulnerabilities because the queries are being crafted by my ORM and not permitting arbitrary data or queries...but there are some things which my validation is lacking. I don't think many people are out there doing 9999 pullups at a time :). I'm also not checking the date format or ranges. I'm not checking that the amounts being provided are integers or floats/decimals. If this were handling more sensitive data and I continued to lazily code my backend then I could get in trouble with a vulnerable application.

## Wrap-Up

This was a pretty simple intro to forms and data handling. I didn't really show examples of a vulnerable applications but I at least wanted to explain the concepts behind them and why it's important.

Assuming my comment service is online, check out the comments box at the bottom of any of the pages on this site. This is another Python web service I wrote to accept comments and store them in a SQLite database. You can see some of the server-side logic I implemented to sanitize and validate the inputs (there are still things I can do, but this is good enough): [comments source code](https://github.com/lfglance/comments/blob/main/comments/app.py#L127)

In future examples we'll be expanding on this service and extending it with authentication and user sessions as well as administrative CLI commands and web interfaces.

The dashboard for comments can be found here: [comments dashboard](https://comments.fs10xer.dev)

Leave a comment and try it out.