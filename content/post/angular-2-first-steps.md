+++
draft = false
title = "Angular 2. First steps."
date = "2017-02-15T19:56:35+01:00"
tags = ["Angular2", "AngularJs]
categories = ["AngularJs", "FrontEnd"]
series = []
images = []
description = ""
summary = """
Hards days for the allmighty, super-heroic framework.
Angular 2 lack of power is...
"""

+++

After spending a month with `Angular 2`, I can reliably say `AngularJs`
is far more powerful and more productive, but also less maintainable.

There are things that can't be done by design that are just necessary.
`Angular 2` is more verbose than `AngularJS`, mostly stuff that's not
necessary and don't add value, just noise, I'm talking mostly about importing.

`Angular 2` is really no in the good path for programmers that values
time and productivity, but not everything is bad. Just most of it.


## Scope

`Angular 2` has scopes, don't even think or believe that they dissapear,
because they don't. But there is a **BIG** differece, there is no inheritance.

This could be good and bad. I find that using `#xxx` (variable declaration
in templates) it's a perfect substitute of inheritance.

`AngularJS` scopes makes easier to write any application. It could also make 
very difficult to read and maintain. `Angular 2` address the problem 
introducing more work on the programmer side. It's ok for me, more work, 
more control.


## $http vs Http

There is just two differences, did you see them?
A *dollar sign* and an uppercased *H*.
In this matter `Angular 2` remove a character and also remove a bunch of functionality.

`Angular 2` is moving towards `React` in this matter. Leaving programmers
the opportunity to use another Http libraries like `fetch`.

`AngularJS` has many services that depend on Http requests like template
fetching.

But just learning how to unit-test `fetch` or make it Observable friendly
it's again more, and more, study.


### `Angular 2` has no interceptors.

One of the best things of `AngularJS` just disappear.

I used for offline support, catch error globally and display loading screens.

If you miss them like me use: [angular2-interceptors](https://github.com/voliva/angular2-interceptors)

### over-engineering-design

When worked with `Simfony` I suffered a lot the complexity of everything being
a class, makes your code code far more unreadable and need to cast/transform
many types that really contain the same information.

* Response is a class that recieve ResponseOptions: both contains the same info.
* Request is a class that recieve RequestOptions: both contains the same info.
* Header is a class that recieve an Object with key-values of the headers.
* Cannot sent Header as plain-object to ResponseOptions/RequestOptions (wtf!).

etc.

```ts
import {Headers} from '@angular/http';

let h:Headers = new Headers();
h.set('Content-Type', 'application/json')

this.http.post('<url>', data, {
 headers: headers
})
```

This was all handled by `AngularJS` implicitly, less code and more readable.


### Declare backend

In `Angular 2` you need to declare a backend for Http, usuarlly `XHRBackend`.

As always the lack of defaults in `Angular 2` is not a problem, but it's a
burden to learn because configure an application is more difficult than ever.

### MockBackend

Http can be easily be overriden in `Angular 2`. To mock backend in `AngularJS`
I used interceptors but now it's more elegant and use my own class that
extend Http functionality.

Take a look: `ng2-vs-backend`, to implement a full API :)


## Templates

### Syntax

I have to admit that I really hate `Angular 2` template syntax,
specially the banana syntax `[(ngModel)]`, but after a while
you just don't notice it much, until you start making errors because
error reporting is confusing... it didn't improve at all.

`AngularJS` has problems with it too and I really think `Angular 2`
make the right move this time. Many times found myself thinking about what
should I send to an attribute: an expression, an interpolation, a function.

This is mostly solved in `Angular 2` moving that decision from directives
to templates, I also see the verbose side as annoying but this time it
clarify the usage, so it's ok with me.

### Can't Override templates
 
Something as easy as override templates it's... impossible.
 
Extending a component, may be just a partial solution for a very specific
usecase. In some cases end up being a complete re-bundle of the module.
It's a reverse ingeneering effort of big magnitude.
 
reference:
* http://stackoverflow.com/questions/35018592/override-extend-third-party-components-template
* http://stackoverflow.com/questions/37568173/extend-override-style-of-reusable-angular2-component
* https://github.com/angular/angular/issues/11144
 
### Directive function template
 
In `AngularJS` templates can be functions, `Angular 2` no.
 

## Stack traces / Errors

There is a new library that `Angular 2` use (and develop) `zone.js`
to do long-track-traces.

I'm just wondering why is so confusing because having more lines is not
important. Doesn't make any difference. It's just as confusing as always,
didn't improve.

In `Angular 2` error messages are even more confusing that `AngularJS`, mostly
because configuring your app is more difficult as I said already.

When you get an error, don't try to debug, search for it in google and seek
the answer elsewhere.


## TypeScript
 
I have to say it. It's not bad. Very picky with types IMHO. I have to cast
to any many times because it's impossible to work with user/library
types at the same time.

It's just Javascript with types. No more, no less.

I would never use classes without a reason, but `Angular` (both) gives you a
good one, that [I explained before](/2017/01/advanced-angular-stack-seed/).
Your code will be cleaner and `$scope` var will dissapear in exchange of
using `this` across your code.

I really like the change but also need to learn something that TypeScript
do that are totally unexpected, like how class defaults works.


### The import madness!

Using RxJs is a PIA. Really! I need to import all operators?

After using RxJs for a while I'm just bored to import... so import the entire library.

```ts
import 'rxjs/Rx';
import {Observable} from 'rxjs/Observable';
```

### Lodash types

I'm having problems typing lodash, I'm pretty sure it's my fault.
I have to cast to any so Typescript let me code.

```ts
import 'lodash';
declare var _: any;
```

### Module bundle

When I first try to make a library bundle it as plain TypeScript just don't work
as expected.

I use a generator to make my life easier
[generator-angular-library](https://github.com/mattlewis92/generator-angular-library).

This bundle my module as plain JS and TypeScript types.

 
## Debuggability
 
Forms can't be debugged, for 'unkown reason' `{{ngForm | json}}` throw a circular
recursion, not in `AngularJs` so you can easily debug your forms.

This show the lazyness in `Angular 2`. Why not use:
`Object.defineProperty -> enumerable` to hide those recursion properties.
But they forgot the most important thing, they are using Javascript.

Not only happens with forms also with Http, this example can't be json'ed.

```ts
@Injectable()
export class SessionService {
  username: string = null;
  // ....

  constructor(
    protected http: Http // or public, or private...
  ) {
  }
}
```

## Forms

I can't give a veredict here yet, not enough pain suffered in `Angular 2`.

I would just say it change many things, but overal it's just changes
nothing in particular missing. Maybe with ReactiveForms `Angular 2` take the
lead...


## Observables vs Promises vs Callbacks

There is no end in the stupidity intrusion in JavaScript.
Any solution from other languages are being actively ported, maintained and
adopted in JavaScript.

But, in the end, none work, Observables is just another failure
to solve easy problems. Slightly better than Promises.
`RxJs` is a complete new paradigm to learn.

I just dislike the syntax and concept. I would prefer plain CB and use
[async](http://caolan.github.io/async/) library if needed.


## Testing

First, I'm no friend of unit testing.
None of the real-world problems can be tested without your app context and
backend, so unit-test is mainly to develop in early stages. Then move to
broader test (e2e in our case).

Typescript generate a bunch of code that's never tested in Karma decreasing
you code coverage. I'm working on testing the real writed TypeScript code and 
not the Javascript generated.


## Create an entire App configuration/bootstrap in each test

Why? There is so much verbose in the Module initialization, and there is almost no value.

As always this things force you to workaround.
I now have two files together that initialize my app for browser and karma.


I'm not saying that `AngularJS` did a good job here either.


## I miss the good old $httpBackend

$httpBackend usage is more comfortable than Mockbackend. As I said, I develop a library
to create mocks easily that it's shared in browser/karma context, just to avoid the 
pain introduced by MockBackend design.

But this means that Mock in browser is easy and cleaner than before.

## There is no $interpolate

If you need to interpolate your strings before sending the text to a view, you can't (easily).
Use the Angular 2 parser directly... painfully undocumented

## Angular 2 weight 1 mb

Nothing more to say. Just heavier and slower to compile.

# Recap, final words

Everything is about feelings, `Angular 2` is not bad but if you taste
`AngularJS` you feel that not your favorite ice-cream flavor.

The learning surface in `Angular 2` has increased and not everything is worth
it. There are many changes, some are good, and some are very bad. As always
coding is an exercise of solving problems, and `AngularJS` exceed in
this matter.

`Angular 2` community is smaller, and I really think that `C#` and `Java` people
will push `Angular 2` while `node` people wil push `AngularJS`.
As always a battle of tastes and productivity. (Java and productivity in the same
line...)

The real [`issue 9`](https://github.com/golang/go/issues/9) is:

> Why naming it `Angular 2` when they should named `AngularReact`...
