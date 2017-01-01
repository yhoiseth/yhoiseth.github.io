---
layout: post
title:  "Tutorial: Develop a simple Python CLI app BDD-style with Behave"
categories: python behave bdd gherkin programming
---

I've been playing with _behave_ lately and thought I'd share some of what I've learned in a simple tutorial.

In this tutorial, we'll make a very simple command line interface (CLI) application using Python. Instead of just making it as quickly as possible, we'll use a process called [behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development). [_behave_](https://github.com/behave/behave) will make this process easy for us.

## Prerequisites

I'll assume that you already have Python installed. I'm using version 3.6.0:

{% highlight shell %}
➜  python3 --version
Python 3.6.0
{% endhighlight %}

Next, install _behave_ if you haven't already, for example using _pip_. I'm using version 1.2.5:

{% highlight shell %}
➜  behave --version
behave 1.2.5
{% endhighlight %}

## Project setup

### Directory

Let's make a directory for our application inside a `tutorials` directory in our home directory:

{% highlight shell %}
➜  mkdir tutorials
➜  cd tutorials 
➜  mkdir behave-hello-world
{% endhighlight %}

### Git repository

Next, we'll initiate a Git repository in order to keep track of our changes: 

{% highlight shell %}
➜  cd behave-hello-world 
➜  git init
{% endhighlight %}

I've shared my repository [here](https://github.com/yhoiseth/behave-hello-world).

If you don't want to commit while working through this tutorial, that's completely fine.

## Setting up _behave_

Let's run _behave_ and see what happens:

{% highlight shell %}
➜  behave
ConfigError: No steps directory in "/Users/yhoiseth/tutorials/behave-hello-world/features"
{% endhighlight %}

_behave_ provides us with a useful error message: We're lacking a `steps` directory (and also a `features` directory). Let's make them:

{% highlight shell %}
➜  mkdir features
➜  mkdir features/steps
{% endhighlight %}

Using the `tree` command (`brew install tree` if you're on a Mac), we can view our directory structure so far:

{% highlight shell %}
➜  tree
.
└── features
    └── steps

2 directories, 0 files
{% endhighlight %}

## Creating our feature specification

Let's run _behave_ again:

{% highlight shell %}
➜  behave              
ConfigError: No feature files in "/Users/yhoiseth/tutorials/behave-hello-world/features"
{% endhighlight %}

Again, _behave_ is showing us a useful error message: We don't have any feature files. Let's create one.

Using your favorite text editor (such as [Atom](https://atom.io/)) or integrated development environment (such as [PyCharm](https://www.jetbrains.com/pycharm/)), create `say_hello.feature` inside `features`:

{% highlight gherkin linenos %}
# features/say_hello.feature

Feature: Say hello
  In order to know that my command line application is working
  As a command line user
  I need to be able to run the application and get an output

  Scenario: No flags or arguments
    When I run the application
    Then I should see a greeting to the world
{% endhighlight %}

The above behavior specification is written using [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin). 

Now, when we run _behave_ again, things start happening:

{% highlight shell %}
➜  behave
Feature: Say hello # features/say_hello.feature:1
  In order to know that my command line application is working
  As a command line user
  I need to be able to run the application and get an output
  Scenario: No flags or arguments             # features/say_hello.feature:6
    When I run the application                # None
    Then I should see a greeting to the world # None


Failing scenarios:
  features/say_hello.feature:6  No flags or arguments

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 1 failed, 0 skipped
0 steps passed, 0 failed, 0 skipped, 2 undefined
Took 0m0.000s

You can implement step definitions for undefined steps with these snippets:

@when(u'I run the application')
def step_impl(context):
    raise NotImplementedError(u'STEP: When I run the application')

@then(u'I should see a greeting to the world')
def step_impl(context):
    raise NotImplementedError(u'STEP: Then I should see a greeting to the world')
{% endhighlight %}

## Writing step definitions

Once again, _behave_ is very helpful. This time, it suggests how we can make _step definitions_ in order to test our scenario. We'll do as _behave_ suggests by creating a file called `steps.py` in our `steps` directory:

{% highlight python linenos %}
# features/steps/steps.py

from behave import given, when, then, step


@when(u'I run the application')
def step_impl(context):
    raise NotImplementedError(u'STEP: When I run the application')


@then(u'I should see a greeting to the world')
def step_impl(context):
    raise NotImplementedError(u'STEP: Then I should see a greeting to the world')

{% endhighlight %}

Run _behave_ again, and you'll see a `NotImplementedError` for `STEP: When I run the application`.

This is a small milestone, because we have made our first "failing" test. We're not actually testing anything, but at least we're getting an error. That means that we can get started on our _Red, Green, Refactor_ cycle.

This is a good time to make [our initial commit](https://github.com/yhoiseth/behave-hello-world/commit/c22613bbba2c9b941e1001826f491578eae58abe):

{% highlight shell %}
➜  git add .
➜  git commit -m 'Initial commit'
{% endhighlight %}

Let's implement our step definitions. In `STEP: When I run the application`, we simply need to run the application. We'll do that using _subprocess_, which is part of the standard library:

{% highlight python linenos %}
# features/steps/steps.py

from behave import given, when, then, step
import subprocess


@when(u'I run the application')
def step_impl(context):
    subprocess.run(('python3', 'hello_world.py'))


@then(u'I should see a greeting to the world')
def step_impl(context):
    raise NotImplementedError(u'STEP: Then I should see a greeting to the world')
        
{% endhighlight %}

Now, when we run _behave_, the first step passes. This may seem kind of strange, considering that `hello_world.py` doesn't exist. But `subprocess.run()` doesn't raise an error when the file doesn't exist. It simply runs the command.

_behave_ then goes on to the second step (`Then I should see a greeting to the world`), which fails.

## Our first assertion

Now, let's [commit](https://github.com/yhoiseth/behave-hello-world/commit/f8bd329494d56018bef7b86d6ec47bc5ba578496) before making our first properly failing step definition (with an assertion):

{% highlight python linenos %}
# features/steps/steps.py

from behave import given, when, then, step
import subprocess


@when(u'I run the application')
def step_impl(context):
    context.completed_subprocess = subprocess.run(('python3', 'hello_world.py'),
                                                  stdout=subprocess.PIPE)


@then(u'I should see a greeting to the world')
def step_impl(context):
    assert 'Hello World!' in context.completed_subprocess.stdout.decode('utf-8')

{% endhighlight %}

Notice how we stored `completed_subprocess` as an attribute on the `context` object on line 9 so that we can retrieve the output on line 15.

We now get an `AssertionError` because the script we're running (`hello_world.py`) doesn't output "Hello World!".

## Going green

Let's [commit](https://github.com/yhoiseth/behave-hello-world/commit/faa56d200c7b812e16258a7e7cafa1b4089d6af5) and move on to creating our actual script, `hello_world.py`:

{% highlight python linenos %}
# hello_world.py

print('Hello World!')

{% endhighlight %}

Running _behave_ after creating our script yields the following summary:

{% highlight shell %}
➜  behave                                                           
Feature: Say hello # features/say_hello.feature:1
  In order to know that my command line application is working
  As a command line user
  I need to be able to run the application and get an output
  Scenario: No flags or arguments             # features/say_hello.feature:6
    When I run the application                # features/steps/steps.py:5 0.038s
    Then I should see a greeting to the world # features/steps/steps.py:11 0.000s

1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
2 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m0.038s
{% endhighlight %}

## Progress so far

After [committing](https://github.com/yhoiseth/behave-hello-world/commit/4c17a419a8592039faf87cde2c22eee1f99c955c), let's summarize what we've achieved. Our project now looks like this:

{% highlight shell %}
➜  tree
.
├── features
│   ├── say_hello.feature
│   └── steps
│       └── steps.py
└── hello_world.py

2 directories, 3 files
{% endhighlight %}

We have made a feature description (which you don't have to be a Python programmer to read) and step definitions as well as the actual script.

## Refactoring

Now that we have moved from _red_ (failing tests) to _green_ (passing tests), I'd like to demonstrate some simple refactoring. Let's change `hello_world.py` just a little bit:

{% highlight python linenos %}
# hello_world.py

greeting = 'Hello World!'

print(greeting)

{% endhighlight %}

When we run `behave` again, the assertion still passes. Let's [commit](https://github.com/yhoiseth/behave-hello-world/commit/711e0c66bf5c16a64821d9206c5ee45be8b6e4f9) and wrap up. 

## Closing thoughts

In this case, the script is trivial, but imagine that you were maintaining a large Python application. Having the behavior covered by assertions with _behave_ would mean that you could refactor with confidence.

In addition, the scenarios provide a living documentation which both programmers and non-programmers can read, given that the scenarios are written in a not-to-technical manner. This has the potential to ease communication between people involved in the project.
