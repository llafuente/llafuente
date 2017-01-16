+++
draft = false
title = "Advanced Angular Seed"
date = "2017-01-12T12:31:35+01:00"
tags = ["aws", "aws-cli", "lambda", "dynamodb", "ec2"]
categories = ["AWS"]
series = []
images = []
description = ""
summary = """
Developing and deploying AngularJS apps involves many tasks that are all
connected to produce a final single HTML, JS and CSS files.
After 3 years working on with AngularJS I explain the evolution of my seed
project to current webpack 2, es6 and NPM.
"""

+++

I've been working in Angular for 3 years. After some time tooling start to
improve, as always, there were difficult beginnings (manually beginnings).

# First year

I worked with [requirejs](http://requirejs.org/).
It was really painful to write boilerplate that I don't really need.
Also add an extra indentation...
It's just not justified to use requirejs nowadays, but it wasn't bad in the
past.

Also used [Grunt](gruntjs.com) for the final build-minification.

# Second & Third Year

I started using Grunt for all development tasks and Bower for front-end
dependencies.
All requirejs boilerplate was gone in exchange of `grunt watch`.

Here are the main development tasks, after those goes the concat and
minification tasks.

* grunt-injector. Inject all LESS files into the main LESS file (bootstrap 3.x)
* grunt-wiredep. The same for JS files on index.html
* grunt-ng-annotate. Fill Angular's Injector boilerplate.
* grunt-ng-templates. To compile HTML in a single JS


For some external reasons, I couldn't use grunt to serve my angular project,
I have to use nginx, so I miss the opportunity of LiveReload...

When I need a node module in Angular I used browserify to bundle it,
running the task before grunt-wiredep and ignore that file in git. It was an
easy solution but introduce extra complexity to debug because don't
have sorucemaps support after the build.

I'm not going deeper in the stack this is just a rough into.

# Current stack

After working on Angular for many years and a few stacks changes
I know a lot of problems with each stack and almost all are solvable.

Here are some reference links I used to build the stack.

* [markmarijnissen/angular-webpack-seed](https://github.com/markmarijnissen/angular-webpack-seed)
* [kitconcept/webpack-starter-angular](https://github.com/kitconcept/webpack-starter-angular)
* [zombiQWERTY/angular-webpack-starter](https://github.com/zombiQWERTY/angular-webpack-starter)
* [zombiQWERTY/angular-component-way-webpack-starter-kit](https://github.com/zombiQWERTY/angular-component-way-webpack-starter-kit)

### Webpack 2

`Webpack 2` is pretty new, it's a bit of a gamble.

`Grunt` is fine, but `Webpack` it's just better to bundle your code.
While `grunt` is a task runner and realy on external modules (a lot of modules)
Webpack is a module bundler.

I have problems with the size of node_modules, webpack need less modules.

[webpack-dev-server](https://github.com/webpack/webpack-dev-server)
`Serves a webpack app. Updates the browser on changes.`
In production you can use anything to serve the minified files nginx, node...

### Javascript

Supporting ES5/ES6 in my stack is now mandatory.

In the past I prefer to avoid it, because it wasn't mature enough and I don't
like to debug 'tooling' code, tools just works or I'm wasting my time!

Source map is now mandatory, never has one and I suffer a lot on production
environments to figure where is the problem.

* [babel/babel-loader](https://github.com/babel/babel-loader)

#### Angular Controllers

When making controllers I see many boilerplate because many things need to be
assigned to scope. ES5 classes remove that boilerplate in exchange of OO
complexity abuses.

But!

* Introduce new boilerplate for dependencies.

  This is something I need to address some time in the future but
  I'm clueless atm. The main problem is to know what is a dependency.
  I really like to do `import $http 'angular/$http'` and avoid $injector,
  but it's not possible.

  I plan todo something with babel/ng-annotate to avoid to assign dependencies
  to `this`.

* Callback boilerplate, need to bind this everytime!

  This could be avoided using `::` operator or writting a babel transformation,
  It's just two characters... not a big deal.

This is an example of a nice clean controller in ES5

```js
export default /*@ngInject*/ class UserListController {
  constructor($http) {
    this.$http = $http;
  }

  get() {
    this.$http({
      method: 'GET',
      url: '/api/users'
    }).then(::this.setList);
  }

  setList(response) {
    this.list = response.data;
  }
}
```

#### Angular Config

Controllers are classes, but config are functions I leave an example.

```js
import loginHTML from './login.tpl.html';
import loginController from './login.controller.js';

export default /*@ngInject*/ function($stateProvider) {
  $stateProvider.state('login', {
    url: '/login',
    templateUrl: loginHTML,
    controller: loginController,
    controllerAs: 'loginCtrl'
  });
}
```

This aproach has a instant pro: You can move this module aound and it's working
regardless the path. While before with grunt I had to adjust the html path.

### SASS

Bootstrap v4 default "CSS engine/preprocessor"

Like in Javascript, sourcemaps are mandatory.

Here we lose automation, when using `grunt` all my less files where wired into
my main LESS file using `grunt-injector`. I cannot find anything like that
for webpack, right now wiring is manually.


### Seed project is modular.

Every 'component' of the seed project is modular.

For easy to use I include a module to bundle the entire seed see
[seed.js](https://github.com/llafuente/angular-es6-stack/blob/master/app/seed.js)

Components

* JWTAuth: Authentication with JWT
* stateBusy: display loading box when there is a ui-router router change
* httpBusy: display loading box when http is in progress.
* uiRouterRedirect: listen to `RedirectTo` property in states to redirect
* httpErrorHandling: display a modal with an error from API


### Eslint

With `--fix` don't force people to write your way, just fix their nonsense!

```
npm run lint
```

## Github project

[llafuente/angular-es6-stack](https://github.com/llafuente/angular-es6-stack)
You can follow the progress of the stack is mainly finished to my needs,
but I can include something if it proves to be useful or increase productivity.

The project is meant to be forked and modified, it include examples that
shouldn't be in your project.

### TODO list

#### Hot module Reload

This is a very nice feature of Webpack. With Angular 1.x is not so easy,
there are some tradeoff mostly in naming convention but my pride could survive!

* [honestica/angular-hot-reloader](https://github.com/honestica/angular-hot-reloader)
* [yargalot/Angular-HMR](https://github.com/yargalot/Angular-HMR)

### Templates: Jade(Pug) and HTML.

Jade is one of those languages that helps you to keep you code sane.
The syntax is clever and clean in exchange of perfect indentation.

Building a Botstrap App also contains boilerplate I should be able to avoid
using Jade `block(s)`. Jade to HTML is like SASS to CSS: power!

Another option is to use: [angular-schema-form](http://schemaform.io)

* [pugjs/pug-loader](https://github.com/pugjs/pug-loader)

I don't have a strong opinion about it yet.

### TypeScript

In bigger projects, many people can be full-stack. In environments with .NET
this could lead to an extra productivity. Many people like OO abuses

This is just something to think about, I don't plan to include it in my seed
right now, but it's something to think about.
