+++
draft = false
title = "Angular 2. +1 year."
date = "2018-08-25T19:56:35+01:00"
tags = ["Angular2", "AngularJs"]
categories = ["AngularJs", "FrontEnd"]
series = []
images = []
description = ""
summary = """
After many, many hours of development, some change and some remain.
"""

+++

First things first, my mental image about `Angular 2` has
improved over time mainly because:

* Necessary changes are introduced in Angular 4/5/6
* TypeScript is awesome

I will review what I said in the past.

## Scope

`Angular 2` work as isolated scope always. You can "get" other scope with `#xxx`
(variable declaration in templates) it's just perfect.

`AngularJS` scopes feels easier to write any application and connect it, but
there is no control over it, and maintainance could be painful.


## $http vs Http


`Http` evolve into `HttpClient` that it's closer to `$http` usage.

This time we have something very close to AngularJS interceptors.
It's a nice addition.


### over-engineering-design

This remain mostly the same, the abuse of classes instead of interfaces force
to write many `new class(xxx, yyy, zzz)`.

Some new API (HttpClient for example) has interfaces... but still an issue.


## Templates

Error reporting has improved, was my main concern. Now report the line and
file and useful, errors but the real improvement here is when building
for production, the compiler will find undeclared variables and,
if typed, some type errors.

Can still be improved, as always, but now is good enough.


### Can't Override templates

Noting improve. AOT is default engine and could be problematic... 0 chance.


### Directive function template

Noting improve. AOT again? 0 chance.


## Stack traces / Errors

`zone.js` is still confusing. I couldn't find anything useful from using it
because most of the stack is always Angular/vendor.


## TypeScript

I start to love it, so close to .Net and so close to JavaScript at the
same time...

It improves my productivity, I started to use it on server-side too.


### The import madness!

Noting improve in this manner. It's unavoidable because angular does not use
`namespaces`. it starts to use TypeScript 3 so maybe in the future...

### Lodash types

I can't really make it work on client-angular side.
But it works like charm on server-side (node), I didn't find time and effort for
debugging my tool chain, it will be mostly my fail for sure.

Angular 6 improves the tool chain (webpack 4). I really try to avoid any
contact with the build-step now, just use `ng-cli`.

### Module bundle

`ng-cli` include the concept of projects, and that's precisely what I needed.

## Debuggability

Nothing really improves. Debug and Angular app It's still a bit painful.

Errors outside templates are very confusing, and error in application
bootstrap are horrendous.

Angular should start thinking about loggin something.

## Forms

ReactiveForms are verbose and force me to write many, many unnecessary
things over and over. But now I undestand that the have many good things.

So I write a wrapper above so I can write just a simple JSON with the
basic functionality and avoid verbosity :smile:


## Observables vs Promises vs Callbacks

`rxjs` is a library to love and hate. I see improvements and also
annoyances, but mostly same code size/complexity for the same functionality.

My main concern here, it's the learing time, still too high.


## Testing

First, I'm no friend of unit testing.
None of the real-world problems can be tested without your app context and
backend, so unit-test is mainly to develop in early stages. Then move to
broader test (e2e in our case).

Also pure unit-test only test your TypeScript, and that's simply stupid.
You need to test what the user will see, and continue to be difficult in
`AngularJS`, `Angular` and anything else out there.


## Create an entire App configuration/bootstrap in each test

Why? There is so much verbose in the Module initialization, and there is almost
no value.

As always this things force you to workaround.
I now have two files together that initialize my app for browser and karma:
`AppModule` and `AppModuleTest`.


I'm not saying that `AngularJS` did a good job here either.
`Angular` just makes the test longer to write.


## I miss the good old $httpBackend

$httpBackend usage is more comfortable than Mockbackend.
As I said, I develop a library to create mocks easily that it's shared
in browser/karma context, just to avoid the pain introduced by MockBackend
design.

But this means that Mock in browser is easy and cleaner than before.

## There is no $interpolate

Nothing improve.
I write my Apps differently now, so I dont really miss it.

## Angular 2 weight/size 1Mb

It very small now. 150-200Kb + polyfills.

# Recap, final words

I see `Angular` as a winner because `TypeScript` in it's core.
Also some `AngularJS` maintainers move to `Angular`, so google is moving
towards `Angular` and sadly, `AngularJS` will die eventually.
