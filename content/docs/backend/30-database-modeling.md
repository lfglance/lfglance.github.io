+++
title = "Database modeling and schemas"
description = ""
date = 2023-12-10T08:00:00+00:00
draft = false
weight = 30
sort_by = "weight"
template = "docs/page.html"
slug = "database-schemas"

[extra]
lead = 'Defining the data you will store in a relational database.'
toc = true
top = false
+++

In this section we will introduce a relational database and start incorporating real data. We will also start converting our simple hello world app to something more "real".

## Object-Relational Mapping (ORM)

In the old days of writing software, developers would have to manage databases manually by running SQL statements. They would then include SQL statements in their code which query the database and return data in responses. However, this proved to be clunky and unsafe. There were many security issues such as [SQL injection attacks](https://owasp.org/www-community/attacks/SQL_Injection) where someone could trick an application into running a different SQL statement as is designed. It could be used to leak data that wasn't intended to be shared, delete, add, or modify data (i.e. updating a student's test scores, delete everything, etc).

ORM's don't fix this problem entirely, caution is still required when developing business logic, but ORMs reduce a lot of the attack surface with basic guardrails AND give developers a simpler means to query the database in a way that is native to their programming language.

For this example we will be using a very lightweight Python ORM, [peewee](https://docs.peewee-orm.com/en/latest/index.html).

## Installing Our ORM

Ensure you're in your fs10x repo and have your virtual environment activated (`source .venv/bin/activate`). We will install `peewee` the same way we did `quart` (web framework), by using `pip` (library manager):

```bash
pip install peewee==3.17.0
```

## Installing Our Database

There are a lot of different relational databases available. Open-source ones tend to be the best and most battle-tested:
* PostgreSQL
* MySQL / MariaDB
* SQLite


There's also tons of proprietary and licensed ones (gross):
* Microsoft SQL Server
* Oracle
* DB2

For this example we will be using [SQLite](https://www.sqlite.org/index.html) because it's high speed, low drag (easy to install and get going).

## Hooking It Up

So far in our `fs10xer` repo we have a single file, `app.py`. Now create a new file, `models.py`, with the following contents:

```python
from peewee import *

db = SqliteDatabase("data.sqlite")
```

Now our app is ready to read and write into our SQLite relational database using the `peewee` Python ORM. Now we need to feed it data....we need some tables.

## Database Schema

A relational database is pretty simple; you have one or more databases consisting of one or more tables, which consist of one or more columns, which can hold one or more rows. Maybe a bad way to explain it.

Here's a simple diagram to depict a database consisting of multiple tables containing data regarding movies. 

![](/schema_example.png)

Each blue square represents a table and each item in the table represents a column. The key icon represents a *Foreign Key* (referencing an item from another table). When data is added to the tables they will be added as rows.

We're going to design the schema for our own app. We should do something better than "hello world" though...

## Modeling Schema For A New App

Let's scrap the "hello world" and do something better that we can actually put some data into. I like to do push-ups and pull-ups, so let's make an app that can store that. We should also make it so that people can login and set a profile with an avatar. We can compare stats to see who's doing the most. Maybe they can comment on each other's stuff and leave emojis?

Let's call it something stupid...how about....updogs? Get it? Like pull-**ups**. Eh, you'll get it later. :)

I'm going to type out a decent starting schema first and later I'll show you how to incorporate it into the Python app.

## Pseudocode Schema

* We want people to be able to signup/login...so we'll want a `Users` table.
  * We want to get their username, password, email address, date they joined, and date they last logged in.
    * We *could* put pull-ups/push-ups count in here, but that'll get messy as we'd lose context of *when* those sessions happened. You'll see.
* We want people to store the push-ups/pull-ups they do...so we'll want a `Workouts` table.
  * We want the date of the workout, number of push-ups, number of pull-ups, and maybe some extra detail for the user to input

That'll work for now. We can always add to this.

## Making it Happen

Now let's implement the designed schema into our app. To do this update your `models.py` file to look like this:

```python
from datetime import datetime

import peewee as pw


db = pw.SqliteDatabase('data.sqlite')

class User(pw.Model):
    username = pw.CharField()
    password = pw.CharField()
    email_address = pw.CharField()
    join_date = pw.DateTimeField(default=datetime.utcnow)
    last_login_date = pw.DateTimeField(default=datetime.utcnow)

    class Meta:
        database = db


class Workout(pw.Model):
    date = pw.DateTimeField(default=datetime.utcnow)
    pullups = pw.IntegerField(null=True)
    pushups = pw.IntegerField(null=True)
    details = pw.TextField(null=True)

    class Meta:
        database = db

db.create_tables([User, Workout])
```

The `class` lines you see represent a table and the detail under them the columns. The last line `db.create_tables` is actually going to populate a new SQLite database, `data.sqlite`, with the above schema.

Finally, set your `app.py` file to this:

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

Run `quart run` now to start the app and you will see `data.sqlite` as a new file after that.

With this `peewee` framework we can interact with the database in our Python code. Here is an example with the `quart shell`:

```bash
quart shell
```

This is an interactive Python prompt with the application context and models available.

```python
from models import Workout

# save a record
workout = Workout(
    pullups=10,
    pushups=20,
    details="Quick set in-between phone calls"
)
workout.save()

# query records
workouts = Workout.select()
for workout in workouts:
    print(f"Workout {workout.id} - {workout.pullups} pullups, {workout.pushups}, {workout.details}")
```

---

Now we have an empty database with schema populated, models defined, and our ORM dialed in. We can start capturing data to populate the database. We will start making updates to our business logic and accepting data from users in the next chapter.