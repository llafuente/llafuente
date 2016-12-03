+++
draft = true
title = "Advanced Angular Seed"
date = "2016-11-21T16:31:35+01:00"
tags = ["aws", "aws-cli", "lambda", "dynamodb", "ec2"]
categories = ["AWS"]
series = []
images = []
description = ""
summary = """
Lambda functions are very useful for cronjob, but eventually you will need to
run something inside your architecture or maybe check if your service is
running... use SSH power!
"""

+++

I'm working in Angular for 3 years. After some time tooling start to improve,
but it was also more difficult to setup and maintain.

# First year

I work with [requirejs](http://requirejs.org/).
It was really painfull to write boilerplate that I don't really need.
Also add an extra indentation...
It's just not justified to use requirejs nowadays but it wasn't in the past.

Also use [Grunt](gruntjs.com) for the final build-minification.

# Second/Third Year

I started using Grunt for all development tasks and Bower for front-end
dependencies.
All requirejs boilerplate was gone in exchange of `grunt watch`.

Here are the main development tasks, after those a normal build is fired.

* grunt-injector. Inject all LESS files into the main LESS file (bootstrap 3.x)
* grunt-wiredep. The same for JS files on index.html
* grunt-ng-annotate. Fill Angular's Injector boilerplate.
* grunt-ng-templates. To compile HTML in a single JS


For some external reasons, I couldn't use grunt to serve my angular project,
I have to use nginx, so I miss the opportunity of LiveReload...

Projects grow and I need some node modules. I used browserify to bundle it,
running the task before grunt-wiredep and ignore that file in git was an
easy and nice solution


# Future stack

After working on Angular for many years and changing stack I know some of
the problems of each stack and all are almost solvable (maybe^^).

Here is my propossal (in fact my todo list) for and Advanced Angular
Seed/Stack.

Here are some reference links to build the stack

* [markmarijnissen/angular-webpack-seed](https://github.com/markmarijnissen/angular-webpack-seed)
* [kitconcept/webpack-starter-angular](https://github.com/kitconcept/webpack-starter-angular)
* [zombiQWERTY/angular-webpack-starter](https://github.com/zombiQWERTY/angular-webpack-starter)
* [zombiQWERTY/angular-component-way-webpack-starter-kit](https://github.com/zombiQWERTY/angular-component-way-webpack-starter-kit)

### Webpack

Grunt is fine, but Webpack it's just better to bundle your code.
While grunt is a task runner Webpack is a module bundler.

Webpack is also capable of running a modest sever to start working and 
test the stack felling.

### Hot module Reload

This is a very nice feature of Webpack. With Angular 1.x is not so easy,
there are some tradeoff mostly in naming convention but my pride could survive!

* [honestica/angular-hot-reloader](https://github.com/honestica/angular-hot-reloader)
* [yargalot/Angular-HMR](https://github.com/yargalot/Angular-HMR)

### Templates: Jade and HTML.

Building a Botstrap App also contains boilerplace you can avoid.
I was thinking about using Angular.component to simplify my HTML but components
created an *Isolated scope* that could be very problematic,
specially when building complex forms. It's just not ideal, I could fallback to
normal directives, but I will hit browser performance...

Using Jade is like using LESS, extra power.
That's the main reason to use Jade in special `block`(s).
Each block contain a bootstrap "components" that can be written easily.

Also support plain HTML because knowing Jade and the bootstrap blocks could
be troublesome to many people. I try to minimize knowlege in my stack, so Jade
it's just optional and transparent.

Another option is to use: [angular-schema-form](http://schemaform.io)

* [pugjs/pug-loader](https://github.com/pugjs/pug-loader)

### Javascript

Supporting ES5 in my stack is now mandatory.

In the past I prefer to avoid it, because it wasn't mature enough and I don't
like to debug 'tooling' code, tools just works or I'm wasting time!

### TypeScript

In bigger projects, many people can be full-stack. In environments with .NET
this could lead to an extra productivity.

### Angular Controllers

When making controllers I see many boilerplate because many things need to be
assigned to scope. ES5 classes remove that boilerplate.

But!

* It also introduce extra boilerplate for dependencies.

  This is something I need to address some time in the future but
  I'm clueless atm. The main problem is to know what is a dependency.
  I really like to do `import $http 'angular/$http'` and avoid $injector,
  but it's not possible, because injector can be configure per App and
  Controllers can be reused... I can't find a bulletproof method.

* More boilerplate for callback, need to bind this everytime!

  This could be avoided using `::` operator or writting a babel transformation,
  It's just two characters... not a big deal.

* Open the door to OO abuses and complexity.

This is an example of a nice clean controller in ES5
  
```
export default class UserListController {
  constructor($http) {
    this.$http = $http;
  }

  get() {
    $http({
      method: 'GET',
      url: '/api/users'
    }).then(::this.setList);
  }

  setList(response) {
    this.list = response.data;
  }
}
// these maybe are required
UserListController.$isNgClass = true;
UserListController.$inject = ['$http'];
```

But above all, the best is that I can reuse the 'get-scope-method' in other
files. And that will allow to reuse code without extending that it's my main
concern.

### SASS

Not a big fan of SASS, I think LESS has better and fit all my needs. But
it's bootstrap v4 default so *move along angela*

Also something i have problems in the past are *source maps*, I never have a 
source map for production environments, I'm need to fix it! because it something
painful to debug.


### Services configured.

angular.config is per App. Many modules require configuration


### Karma using TAP

Karma is nice, jasmine is not. I'm avoiding complex staff. TAP is simple.

Objectives

* Auto-wire dependencies
* Stacktraces
* Coverage
* No mocks, just stubs. Mocks are evil!

### Protractor

### Eslint

With `--fix` don't force people to write your way, just fix their nonsense!




### If I have backend control.

I will force backend to display all models in a readable/usable format for
font-end. One things always happens is that backend change string length in
database and font-end don't event have max-length.
Avoid this it's important.


### Final wiring


Webpack do not wire your App unlike grunt-wiredep, but it a nice `plus` feature.

```
import angular from 'angular';

import ExampleController from './example-controller/example-controller.js';
import ExampleDirective from './example-directive/example-directive.js';
import ExampleService from './example-service/example-service.js';

export default angular.module('example', [])
	.controller('exampleController', ExampleController)
	.service('exampleService', ExampleService)
	.directive('exampleDirective', () => new ExampleDirective);
```
