+++
draft = false
title = "Arquitecture decisions: Web components."
date = "2018-12-01T17:31:00+01:00"
tags = ["Angular2", "AngularJs", "React", "Web Components"]
categories = ["FrontEnd", "Web Components"]
series = []
images = []
description = ""
summary = """
Introduce a new series of articles about Web components usage.
"""

+++

Web components is here to stay, so I start a new research about them.

Do you want to know what they are? read
[Web Components - Introduction](https://www.webcomponents.org/introduction).
Here we will implement and use them.

## Current state of mind

Web Components are not really that good IMHO for 95% of the daily front-end
work. It makes migrations and sharing very small pieces easy, because it can
be implemented using almost every mayor framework and used virtually everywhere,
I hope it's true... But I don't see managing and entire SPA using
Web Components, and it's obvious it's not its purpose, so you will need a
framework after all.

My main use case is to develop new components in Angular 7 that will be used
by Angular 4 & AngularJS applications that will be migrated... some day :)
I will consider implementing Web Components in React/Polymer/Vue
as an alternative to Angular 7.

## What to expect from this articles

In this series I will implement a basic Web Components in mayor all-purpose frameworks:
Angular 7, React, Polymer and maybe Vue and Vanilla JS.

After the implementation I will use those components inside: Angular 7/4,
AngularJS, React and in plain JS/HTML. Focus on comunications:

* Attribute vs property
* Typing
* Custom events

I suspect that Web Components on ie11 will be harder because the lack of
es6 support, but should be more a tool chain issue that a coding.
Also React, don't have official support. All frameworks have
its own way of thinking about HTML properties and attributes, that will mess
things a lot a make difficult to handle. Will see...

Everything will be tested on: chrome, firefox, edge, ie11
(because is [still alive](https://www.youtube.com/watch?v=Y6ljFaKRTrI)).
The final veredic will be if a framework is usable to implement or to use
Web Components.

## What Web Componets are?

Web Components are an custom HTML tags with your logic and design.

* [Custom Elements](https://w3c.github.io/webcomponents/spec/custom/):
Custom elements provide a way for authors to build their own fully-featured
DOM elements

* [Shadow DOM](https://w3c.github.io/webcomponents/spec/shadow/):
Allow HTML/CSS encapsulation to isolate your component from the rest of the
document.

* [HTML Imports](https://w3c.github.io/webcomponents/spec/imports/):
HTML Imports are a way to include and reuse HTML documents in other HTML
documents

* [HTML Template](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element/):
The template element is used to declare fragments of HTML that can be cloned and inserted in the document by script.

## References

* [Shadow DOM on MDN](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
* [Can I use? Shadow DOM v1](https://caniuse.com/#feat=shadowdomv1)
* [Can I use? Custom elements v1](https://caniuse.com/#feat=custom-elementsv1)
* [webcomponents.org](https://www.webcomponents.org/introduction)
* [Web Component standard status](https://www.w3.org/standards/techs/components#w3c_all)
* [custom-elements-everywhere](https://custom-elements-everywhere.com/)
