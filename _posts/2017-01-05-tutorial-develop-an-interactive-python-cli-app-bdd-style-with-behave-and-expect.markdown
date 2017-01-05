---
layout: post
title: "Tutorial: Develop an interactive Python CLI app BDD-style with Behave and Pexpect"
categories: python behave bdd gherkin programming pexpect
previous_tutorial_name: 2017-01-01-tutorial-develop-a-simple-python-cli-app-bdd-style-with-behave
---

In the previous tutorial, we [developed a very simple command line interface application]({% post_url 2017-01-01-tutorial-develop-a-simple-python-cli-app-bdd-style-with-behave %}).

In this tutorial, we'll expand our script by adding a new feature, interactive mode. We'll implement our step definitions, which we use to test our scenarios, using [Pexpect](https://github.com/pexpect/pexpect). 

(I tried to use `subprocess` to test my interactive CLI app, with no success. It seems like it isn't supposed to. With Pexpect, however, it's a breeze. Big shout-out to [Jeff Quast](https://jeffquast.com/), [Thomas Kluyver](https://takluyver.wordpress.com/) and [all the other people](https://github.com/pexpect/pexpect/graphs/contributors) who have contributed to Pexpect.)

## Starting our workflow

To structure our work, it's a good idea to begin work on a new feature by creating an issue in whatever issue tracker we're using. Because the project is [hosted on GitHub](https://github.com/yhoiseth/behave-hello-world), I'll use their built-in issue tracker. 

Let's call the issue [Feature request: Personalized greeting](https://github.com/yhoiseth/behave-hello-world/issues/1). The issue contains a description of the value of the feature, nothing about the implementation. This is intentional. It's often a good idea to first think about what we want to achieve before considering the implementation details.

## Prerequisites

In addition to Python and `behave`, which we needed in our [previous tutorial]({% post_url 2017-01-01-tutorial-develop-a-simple-python-cli-app-bdd-style-with-behave %}), we need [Pexpect](https://github.com/pexpect/pexpect). I'm using version 4.2.1:

{% highlight shell %}
➜  behave-hello-world git:(master) pip3 freeze pexpect
…
pexpect==4.2.1
{% endhighlight %}

Before we start, run `behave` to make sure that our tests are passing. If not, refer to our [previous tutorial]({% post_url 2017-01-01-tutorial-develop-a-simple-python-cli-app-bdd-style-with-behave %}).

## Feature description

Let's start by describing the feature in a new file, `personalized_greeting.feature`, located in the `features` directory:

{% highlight gherkin linenos %}
# features/personalized_greeting.feature

Feature: Personalized greeting
  In order to feel welcome
  As a user
  I want to receive a personalized greeting

  Scenario: Provide name
    When I run the application in interactive mode
    And I provide my name
    Then I should receive a personalized greeting
{% endhighlight %}

Notice the step on line 9. We'll create an interactive mode in order to let the user provide their name without messing up the default greeting ("Hello World!").

Let's before continuing. I'll create a new branch, `1-personalized-greeting`. The number "1" is to signify the connection to issue number 1. You can find my commit [here](https://github.com/yhoiseth/behave-hello-world/commit/8107a84e3dec729e0d2da7625f1cabecc814f249).


Now, when we run `behave`, it tells us that we're missing three steps, and provides us with suggestions on how to create them. Let's add them to `steps.py`:

## Step implementations

{% highlight python linenos %}
# features/steps/steps.py

# …

@when(u'I run the application in interactive mode')
def step_impl(context):
    raise NotImplementedError(
        u'STEP: When I run the application in interactive mode')


@when(u'I provide my name')
def step_impl(context):
    raise NotImplementedError(u'STEP: When I provide my name')


@then(u'I should receive a personalized greeting')
def step_impl(context):
    raise NotImplementedError(
        u'STEP: Then I should receive a personalized greeting')
{% endhighlight %}

Run `behave`, and it'll fail with a `NotImplementedError`. Brilliant. We're in the red again. Let's go green.

## Using Pexpect

To use Pexpect, we need to import it, and then use the [spawn()](https://pexpect.readthedocs.io/en/stable/api/pexpect.html#spawn-class) class to start our child application, `hello_world.py`:

{% highlight python linenos %}
# features/steps/steps.py

# …

import pexpect

# …

@when(u'I run the application in interactive mode')
def step_impl(context):
    context.child = pexpect.spawn('python3 hello_world.py --interactive',
                                  encoding='utf-8')

# …
{% endhighlight %}

A few things are worth noticing in the above step implementation:

1. We store the child application as an attribute on the `context` object in order to access it in other step definitions.
2. The `--interactive` flag lets the script know that the user expects interaction.
3. UTF-8 encoding eases string handling.

When we run `behave`, the above step passes.

Let's implement the next one:

{% highlight python linenos %}
# features/steps/steps.py

# …

@when(u'I provide my name')
def step_impl(context):
    context.child.expect('Name: ')
    context.child.sendline('Biggus Dickus')
{% endhighlight %}

## Fix one error, introduce another

Now, Pexpect raises an `End Of File` exceptions, meaning that `hello_world.py` never output the expected `Name: `. Let's fix that:
 
{% highlight python linenos %}
# hello_world.py

greeting = 'Hello World!'

print(greeting)

input('Name: ')
{% endhighlight %}
 
Now, the step `And I provide my name` passes. But we have a small problem: The non-interactive mode doesn't work properly any more. Try running `hello_world.py` manually to see what I mean:

{% highlight shell %}
➜  behave-hello-world git:(1-personalized-greeting) ✗ python3 hello_world.py
Hello World!
Name: 
{% endhighlight %}

This is also apparent when running `behave`. The test hangs indefinitely on `When I run the application`. We can fix that by adding a timeout (of 1 second) to the call to `subprocess.run()`:
 
{% highlight python linenos %}
# features/steps/steps.py

# …

@when(u'I run the application')
def step_impl(context):
    context.completed_subprocess = subprocess.run(('python3', 'hello_world.py'),
                                                  stdout=subprocess.PIPE,
                                                  timeout=1)
                                                  
# …
{% endhighlight %}

## Capture the flag
 
Now, the test fails as it should. To make it pass, we need to add some logic that produces different output depending on whether the `--interactive` flag was used:

{% highlight python linenos %}
# hello_world.py

import argparse

parser = argparse.ArgumentParser()
parser.add_argument('--interactive', action='store_true')
arguments = parser.parse_args()

if arguments.interactive:
    input('Name: ')

greeting = 'Hello World!'

print(greeting)
{% endhighlight %}
 
Now, all steps are passing except `Then I should receive a personalized greeting`. Let's write the step definition:

{% highlight python linenos %}
# features/steps/steps.py

@when(u'I provide my name')
def step_impl(context):
    context.child.expect('Name:')
    context.name = 'Biggus Dickus'
    context.child.sendline(context.name)


@then(u'I should receive a personalized greeting')
def step_impl(context):
    context.child.expect('Hello ' + context.name + '!')
{% endhighlight %}

## A final fix
 
This still fails because we haven't implemented the feature in `hello_world.py`. That's the next step:

{% highlight python linenos %}
if arguments.interactive:
    name = input('Name: ')
else:
    name = 'World'

greeting = 'Hello ' + name + '!'

print(greeting)
{% endhighlight %}

Now, our tests pass and both the interactive and non-interactive modes work.

Let's [commit our changes](https://github.com/yhoiseth/behave-hello-world/commit/f0dba85844f705c080be07e4ae7e19373a37e7e5) and create and merge a [pull request](https://github.com/yhoiseth/behave-hello-world/pull/2). Notice how I typed "Fixes #1" in the description. That tells GitHub to close issue number 1 automatically when the pull request is merged.

## Ideas for improvement

That's it. In case you want to practice by making some improvements yourself, here are some suggestions:

1. Replace `subprocess` with `pexpect`.
2. Add a short `-i` flag which does the same as `--interactive`.
3. Add a second input prompt, for example `Location: `.
