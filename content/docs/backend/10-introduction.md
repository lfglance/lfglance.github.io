+++
title = "Backend principles introduction"
description = "Introduction to backend software engineering principles."
date = 2023-12-10T08:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"
slug = "introduction"

[extra]
lead = ''
toc = true
top = false
+++

Backend software is the underlying business logic to a service or platform. It's responsible for storing and serving data to clients (frontends and APIs).

In this section we will go through the process of building a new backend service; it will be a web application with a simple database schema, some routes for form handling, an API for serving and storing data, and administrative functionality/tools.

Some concepts that are important to keep in mind:

## KISS (Keep It Simple, Stupid!)

The biggest one and part of all the below items.

Simplicity is always better. Don't over-engineer things. You're only making the codebase harder to work with and causing headaches with new people as they come aboard and have to reverse engineer things to "grok".

Sometimes complexity is unavoidable, especially as features and teams sprawl. Always look for the simplest option with any implementation you do.

## Minimize Blocking I/O

It's common for backends to do complex things; fetch data from another API, query a database, parse and manipulate data, run math operations, etc.

When you put these "moving parts" into your backend's responses you are blocking I/O and the client hitting your backend has to wait to get the data.

A better pattern is to have background tasks to periodically update a cache or database entry. When clients hit the backend they are served the cached data and responses are simple cache returns. You *can* also do lazy loading techniques that do the complex operation and cache it for subsequent calls...but that first user to miss cache gets screwed.

Sometimes unavoidable, but try to keep this in mind.

## Document Your Stuff

Things inevitably turn into a paper mache of tacked on stuff. Do your best to keep things documented to clarify relevant context so someone can pick it up and start working with it faster.

## Don't Repeat Yourself (DRY)

This applies to any software you develop. If you find you are copy/pasting functions or work try to find a way to consolidate in one place. If a function needs to be updated later you will do it once.

In all my projects I typically will have a library of "helpers" that I can call from anywhere in my code base.

## Consider Production Scale

Most backend operations are dealing with a data store; relational database, cache, object/filesystem storage, etc.

Things that might work well locally, such as a SQL statement (`SELECT * from mytable`), may have different execution times when code hits production. If your local database is 10 MB but production is 800 GB this will definitely be the case. Make sure your SQL statements (or ORM calls) take this into account and are as explicit and efficient as possible.

## There Is No "Best"

There are a lot of these debates in tech where proponents of certain technologies argue about a tool being "the best". It's not a thing, there is only a "best tool for the job".

Don't waste your time in these trivial matters. Use whatever tools work for you and best fit the job. Remember, simpler is always better.

## Nothing Lasts Forever

Whatever you build will eventually be destroyed. Period.

Absolutely everything written and deployed will become deprecated, die, go away, etc. The ever increasing versions of software will retire your code. New cloud services will require refreshes to your infrastructure. There is a rate of decay for all things, even in software, and your job is simply to keep the rate as low as you can.

I still have shell scripts running in production after many years and many team switches. They will probably get retired eventually but it goes to show how picking tools with simplicity and longevity is usually the better long-term option.

## Don't Optimize Prematurely

Same as above, you're never done. Software and infrastructure will constantly evolve. Your "solutions" are just implementation decisions that come with their own set of pros and cons and will need new solutions for down the road. It might be 1 year or it might be 5. It's about keeping balance and ensuring you can make incremental improvements.

Don't boil the ocean and try to fix every possible edge case or nice to have. Start small and work your way out.

## Stick To Twelve-Factor

[The Twelve-Factor App](https://12factor.net/) is a methodology for creating apps and services that are cloud native, portable, and streamlined. You should read it and keep it in mind for any software development you do.

## You Should Write Tests

Tests are a pain in the ass but will ensure you're writing code that conforms to certain conditions and that bugs will be detected before being shipped. Test purists will define tests first and then do their implementation but test along the way ensuring conformity; this is called test-driven development.

## Use Virtual Environments

It's very common for developers to have multiple versions of the programming languages installed to their machine so they can test different software and check legacy compatibility.

Different languages also have different methods to select the version you'd like to use. I am not going to setup multiple versions of Python in this course, but we are going to setup a **virtual environment**.

Virtual environments are essentially a location where you can install all of your third party libraries necessary for your application, but in a way that doesn't interfere with the rest of your system. 

For our Python app, we will be using the built-in `venv` library.

## Use Source Control

You should be using [Git](https://git-scm.com/). Make sure it's installed to your system. A source control provider is your choice, but you should be using *something* to effectively version your code changes and house it in another place besides your machine. This is vital for teams (usually).

## You (Probably) Need (Want) Libraries

Libraries are what you install into your virtual environment. They are typically third party and fetched over the internet. They enable different functionality and capabilities for your software. 

Programming languages all have different ways of managing libraries:
* Ruby uses [Gemfile](https://www.rubyguides.com/2018/09/ruby-gems-gemfiles-bundler/)
* Python uses [Pip](https://pypi.org/project/pip/)
* PHP uses [Composer](https://getcomposer.org/doc/00-intro.md)
* Golang uses [go mod](https://go.dev/ref/mod)
* Javascript uses [npm](https://www.npmjs.com/)

etc. There are other ways languages handle libraries, but those are the most common for those languages. In addition to the tool which installs the libraries there is another dependency on a service to actually download the library from. Your library manager will install libraries from various hosting services, typically hosted by the library manager creators.

All software basically relies on externally located libraries maintained mostly by open-source contributors, sometimes companies. This reliance on external software is a security concern as you run the risk of downloading hijacked or malicious libraries; this is called *supply chain poisoning attacks*. That's [what happened here with Ledger](https://techcrunch.com/2023/12/14/supply-chain-attack-targeting-ledger-crypto-wallet-leaves-users-hacked/), a cryptocurrency wallet company; malicious libraries stole millions of dollars from their users. We will cover this later in the SecOps section.

---

Let's move on and get started.