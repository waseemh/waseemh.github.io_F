---
layout: post
title: "An overview of 'Truth', Google's new Java testing framework"
comments: true
permalink: overview-truth-google-new-testing-framework
---
Although still considered a very new project, it was interesting to see what does [Truth](https://google.github.io/truth/), a new testing framework by Google, pioneer in the world of test automation.

'Truth' is a Java testing framework with an aim to make test assertions and failures more readable.
It is not a fully-featured testing framework, but it can be integrated with any type of test and with any testing framework.

Truth comes with two major features: __Fluent Assertions API__ and __Failure Strategies__

## Fluent Assertions API

Truth's assertion API allows you to write test assertions in a fluent style.

The way assertions are composed allow other users to easily understand what the test is trying to assert.
As a demonstration, let's compare good ol' JUnit assertions with Truth assertions:

{% gist 64811c969907e3d00734 %}

Additionally, API provides advanced assertions which are not available in JUnit:

{% gist e817ce1d6e4df0f3289d %}

As you can see, the fluent API allows you to compose complex, yet readable propositions.

For a [full list](http://google.github.io/truth/usage/#built-in-propositions) of built-in propositions.

When compared with JUnit+Hamcrest assertions, it appears that Truth's fluent API is slightly better:

{% gist f7800ac4f1130bd8ee99 %}

But when compared with other advanced assertion Java open source libraries such as [AssertJ](http://joel-costigliola.github.io/assertj/) or [FEST](https://code.google.com/p/fest/), Truth's fluent API doesn't provide any significant enhancements.

## Failure Strategies

Truth has expanded test assertion beyond its traditional definition. An assertion in Truth is a 'test verb' which asserts on a 'subject' (a subject is the object under test, i.e: a collection, an Integer, a User object).
Other test verbs provided by Truth are 'Assumption' and 'Expectation'. 

Each of these verbs defines how a test will behave upon failure, AKA __'Failure Strategy'__:

  - __Assertion failure strategy__: the normal behavior you would expect from an assertion - fail the test and report error.
  - __Assumption failure strategy__: skip test upon failure without reporting an error.
  - __Expectation failure strategy__: continue to run the test upon failure, but report an error when test is completed.

You can also implement your own failure strategy and use it in a test verb (more on that later).

In order to switch to a different failure strategy (ASSERT is default), you can use ASSUME or EXPECT (as JUnit rule) when writing assertions. For example:

{% gist 0906a41caa39f381a48d %}

## Customizable Failure Messages

If an assertion error message isn't clear enough or too general, you can override it using withFailureMessage(message). Method invocation should be appended to the assertion with an appropriate error message.

For example, if you want to assert that a list is empty but still display a specific message in case of error:

{% gist 2aa04d032b152ee3499c %}

## Extensible Assertions

Truth is a very extensible framework as it provides an abstraction of test assertion.
Assertions are very modular in Truth -  you can define your own domain-specific assertion API by implementing custom test verbs, subjects and failure strategies.

Creating a custom test verb by extending TestVerb class:

{% gist 4379f941b6550cb08183 %}

Creating a custom subject by extending Subject class:

{% gist 9764b24fdb02c92c9626 %}

Use custom verb and subject in a test:

{% gist 07c808831abd85807cd6 %}

## Summary

Truth has a solid foundation given its highly extensible design and abstract definitions of an assertion (verb, subject, propositions, failure strategy).
But other open source testing frameworks provide similar fluent assertion APIs, so Truth doesn't bring any added value in this area.
However since it's still an alpha project, major improvements and new features might appear on upcoming releases.
