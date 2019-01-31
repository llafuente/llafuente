+++
draft = false
title = "Advanced Angular Stack+Seed"
date = "2017-01-27T19:56:35+01:00"
tags = ["AngularJs", "Grunt", "Gulp", "Karma", "Protractor", "SASS", "LESS"]
categories = ["AngularJs", "FrontEnd"]
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

## First year (AngularJS 1.1 - 1.2)

I used [requirejs](http://requirejs.org/).
It was really painful to write boilerplate that I don't really need.
Also add an extra indentation...
It's just not justified to use requirejs nowadays, but it wasn't bad in the
past.

Also used [Grunt](gruntjs.com) for the final build-minification and watch
partials modification to run grunt-ng-templates.

## Second & Third Year (AngularJS 1.4 - 1.6)

I started using Grunt for all development tasks and Bower for front-end
dependencies.
RequireJS boilerplate was removed, and nothing change `grunt watch` were just
more complex.

Here are the main development tasks, after those goes the concat and
minification tasks for JS/CSS/Images/SVG etc.

* `grunt-injector`. Inject all LESS files into the main LESS file (bootstrap
  3.x)
* `grunt-wiredep`. The same for JS files on index.html
* `grunt-ng-annotate`. Fill Angular's Injector boilerplate.
* `grunt-ng-templates`. To compile HTML in a single JS


For architectural reasons, I couldn't use `Grunt` to serve my angular project,
I had to use `nginx`, so I miss the opportunity of LiveReload...

When Projects grew, they always did, and node modules were needed, then I used
browserify to bundle it, running the task before grunt-wiredep and
ignore that file in git. It was an easy and nice solution but introduce extra
complexity to debug because don't have sorucemaps support.

I'm not going deeper in the stack this is just a rough into to spot major
problems: unable to debug, inefficient watchers...

## A Stack for the Future (still wip)

Here are some reference links I used to build the stack.

* [markmarijnissen/angular-webpack-seed](https://github.com/markmarijnissen/angular-webpack-seed)
* [kitconcept/webpack-starter-angular](https://github.com/kitconcept/webpack-starter-angular)
* [zombiQWERTY/angular-webpack-starter](https://github.com/zombiQWERTY/angular-webpack-starter)
* [zombiQWERTY/angular-component-way-webpack-starter-kit](https://github.com/zombiQWERTY/angular-component-way-webpack-starter-kit)

### Webpack 2

`Webpack 2` is pretty new, it's a bit of a gamble.

`Grunt` is fine, but `Webpack` it's just better to bundle your code.
While `Grunt` is a task runner and rely on external modules (a lot of modules)
Webpack is a module bundler, that it's precisely what we are doing,

A side effect is that `node_modules` size is drastically reduced,
I had to enable swap my AWS nano machines while now it's ok.

[webpack-dev-server](https://github.com/webpack/webpack-dev-server)
`Serves a webpack app. Updates the browser on changes.` That's what the docs
says, but they don't say that proxy URLs to other URLs: so `/api/*` can be
forwarded to any other system, awesome!

In production you can use anything to serve the minified files nginx, node...

### Javascript

Use babel to support ES5/ES6.

In the past I preferred to avoid it, because it wasn't mature enough and
I don't like to debug 'tooling' code, tools just works or I'm wasting my time!

I'm not a big fan of Object Oriented Programming, but it make a lot of sense
for Angular controllers to be classes. Providers, Factories and Services should
remain functions... Injector continue to be a problem with a little
`/*@ngInject*/` comment, solved!

Support source maps is now mandatory,
I suffered a lot on production environments... no more.

* [babel/babel-loader](https://github.com/babel/babel-loader)
* [huston007/ng-annotate-loader](https://github.com/huston007/ng-annotate-loader)

#### Angular Controllers

When making controllers I see many boilerplate because many things need to be
assigned to scope. ES5 classes remove that boilerplate in exchange of OO
complexity abuses. But...

* Introduce new boilerplate for dependencies.

  Class constructor has the dependencies and assigned to this everytime.
  A babel plugin could solve it similar/complementary of ng-annotate

* Callback boilerplate, need to bind `this` everytime!

  Could be avoided using `::` operator or writting a babel plugin,
  but It's just two characters... not a big deal.

This is an example of a nice clean controller in ES5.

{{< highlight js >}}
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
{{< /highlight >}}

There is another solution to use arrow functions but makes unit-testing
difficut.

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

This approach has a instant pro: You can move this module around
and it's working regardless the path.
While before with grunt I had to adjust the HTML path.

#### Modulas Seed project

Every 'component' of the seed project is a module.

For easy to use I include a module to bundle the entire seed module collection.
See [seed.js](https://github.com/llafuente/angular-es6-stack/blob/master/app/seed.js)

##### Modules

* JWTAuth: Authentication with JWT
* stateBusy: display loading box when there is a ui-router router change
* httpBusy: display loading box when http is in progress.
* uiRouterRedirect: listen to `RedirectTo` property in states to redirect
* httpErrorHandling: display a modal with an error from API

### SASS/CSS

Bootstrap v4 default "CSS engine/preprocessor" is SASS so good bye LESS.

Like in Javascript, sourcemaps are mandatory.

Here I lose automation. `Grunt` were wiring all my LESS files into `main.less`.
I cannot find anything like that for `webpack`, right now wiring is manually.

### Eslint

With `--fix` don't force people to write your way, just fix their nonsense!

```
npm run lint
```

### Karma

Karma support is done using `gulp` but I will move it to `npm run` if find
time to test on windows.

I don't like Jasmine, I may include TAP here.

### jsDoc

I don't know yet if it's an error or not...
AngularJS use it's own doc system `dgeni`, but configure and setup `dgeni`
was so time consuming that I just have to switch to something that just works.

Maybe in the future it will change...

### Bundle versioning

This is something I'm doing since my first project, include the version in
the final build. To change a bit `gulp` is used.

## Github project

[llafuente/angular-es6-stack](https://github.com/llafuente/angular-es6-stack)
You can follow the stack progress, It's working an open to suggestions.

The project is meant to be forked and modified. *NOTE* it include examples that
shouldn't be in your final project.

## TODO list

### Hot module Reload

This is a very nice feature of Webpack. With Angular 1.x is not so easy,
there are some tradeoff mostly in naming convention but my pride could survive!

* [honestica/angular-hot-reloader](https://github.com/honestica/angular-hot-reloader)
* [yargalot/Angular-HMR](https://github.com/yargalot/Angular-HMR)

### Templates: Jade(Pug) and HTML.

Jade is one of those languages that helps you to keep your code sane.
The syntax is clever and clean in exchange of perfect indentation.

Building a Botstrap App also contains boilerplate that Jade `block(s)` could
prevent. Jade to HTML is like SASS to CSS: power!

Another option is to use: [angular-schema-form](http://schemaform.io)

* [pugjs/pug-loader](https://github.com/pugjs/pug-loader)

I don't have a strong opinion about it yet.

### TypeScript

In bigger projects, many people can be full-stack. In environments with .NET
this could lead to an extra productivity. Many people like OO abuses

This is just something to think about, I don't plan to include it in my seed
right now, but it's something to think about.


### Protractor

I'm big fan of protractor. It's a PIA that help me much more than karma to
see real problems, in exchange of an insane amount of time.

Writing e2e test is an art of patience.

### Karma coverage

### Travis

Run test :)

## Final notes

I really think this stack will last a few years with maybe small changes
already mentioned.

* `grunt` vs `gulp`. I don't mind, because current gulp file is just a few lines.
* `jsDoc` vs `dgeni`
