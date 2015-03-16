---
layout: documentation
title: Integrating Yeoman
category: authoring
sidebar: sidebars/authoring.html
excerpt: How can you integrate Yeoman system in your own tools
---

Everytime you run a generator, you're actually using the `yeoman-environment`. This environment is a base system who's decoupled from any UI component and can be abstracted away by any tool. When you run `yo`, you're basically just running a terminal UI façade on top of the core yeoman environment.

## The basics

What you need to know first is that the environment system is contained in the `yeoman-environment` package. You can install it:

```sh
npm install --save yeoman-environment
```

This module provide methods to: retrieve installed generators, register generators, run generators and provide the common interfaces components. We provide a [full API documentation](http://yeoman.github.io/environment/) (which is the terse list of methods available.)

## Using `yeoman-environment`

### A simple usage example

Let's start with a simple usage example of `yeoman-environment` before we move to deeper topics.

In this example, let's assume `npm` wants to provide a `npm init` command to scaffold a `package.json`. Reading the other pages of the documentation, you already know how to create a generator - so let's assume we already have a `generator-npm`. We'll see how to invoke it.

First step is to instantiate a new environment instance.

```js
var yeoman = require('yeoman-environment');
var env = yeoman.createEnv();
```

Then, we'll want to register our `generator-npm` so it can be used later. You have two options here:

```js
// Here we register a generator based on it's path. Providing the namespace
// is optional.
env.register(require.resolve('generator-npm'), 'npm:app');

// Or you can simple provide a generator constructor. Doing so, you need
// to provide a namespace manually
var GeneratorNPM = generators.Base.extend(/* put your methods in here */);
env.registerStub(GeneratorNPM, 'npm:app');
```

Note that you can register as many generators as you want. Registered generators are just made available throughout the environment (to allow composability for example).

At this point, your environment is ready to run `npm:app`.

```js
// In it's simplest form
env.run('npm:app', done);

// Or passing arguments and options
env.run('npm:app some-name', { 'skip-install': true }, done);
```

There you go. You just need to put this code in a `bin` runnable file and you can run a yeoman generator without using `yo`.

### Find installed generators

But what if you wish to provide access to every yeoman generators installed on a user machine? Then you need to execute a lookup of the user disk.

```js
env.lookup(function () {
  env.run('angular');
});
```

`Environment#lookup()` takes a callback that'll be called once yeoman is done searching for installed generators. Every found generator is going to be registered on the environment.

In case of namespace conflicts, local generators will override global ones.

### Get data about registered generator

Calling `Environment#getGeneratorsMeta()` will return an object describing the meta data the lookup task registered.

## Providing a custom User Interface (UI)
