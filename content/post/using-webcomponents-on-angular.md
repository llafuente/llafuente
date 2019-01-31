+++
draft = false
title = "Using Web components on Angular."
date = "2019-01-12T11:37:00+01:00"
tags = ["Angular", "Web Components"]
categories = ["FrontEnd", "Web Components"]
series = []
images = []
description = ""
summary = """
After implementing web components in a variety of technologies let's start
using them.
"""

+++

In this series I will try to cover Inputs and Outputs of a Web Components,
I'm afraid that is `framework dependant` so Angular/React/Polymer will treat
attributes and properties in different way.


I won't cover toolchains or how to create the distribution. Let's just say
we have the same component implemented in all frameworks and I will add it all
to an Angular App.

## Attribute/property naming

You should know how Angular works.

```ts
@Input() stringInput: string = "default";
```

Attribute name is: string-input

Property name is: stringInput

## Gotchas!


### Initialization attributes only

Angular component will bootstrap with the attributes values and ignore
properties, then it will listen attributes and properties.
If you set properties before Angular bootstrap, the component won't have those
values.

### Events don't fire

A component has a [lifecycle-hooks](https://angular.io/guide/lifecycle-hooks)
managed by Angular.
EventEmitter won't work if the "Angular component" is not attached to DOM for
example: ngOnInit, ngAfterContentInit, ngAfterViewInit, ngOnDestroy, even first
executions of `ngAfterViewChecked`. It's unreliable to use `emit`.

To fire events detached you need to use `nativeElement` directly.

```ts
// ...
  constructor(elementRef: ElementRef) {
    (elementRef.nativeElement as HTMLElement).dispatchEvent(new CustomEvent("ready"))
  }
// ...
```
### Attributes are strings

Nothing new... but that means every class property will be a string.

If you want types Inputs, you need to use properties or setters.

#### setters

```ts
//...
  _arrayInput: string[] = [];

  get arrayInput(): string[] {
    return this._arrayInput;
  }

  @Input('array-input')
  set arrayInput(value: string[]) {
    if ("string" === typeof value) {
      this._arrayInput = JSON.parse(value);
    } else {
      this._arrayInput = value;
    }
  }
//...
```

#### properties

```html
<ng7-test id="ng7-test-property"></ng7-test>
```

```js
var ng7Test = document.getElementById("ng7-test-property");

// wait angular to start listening properties
ng7Test.addEventListener("ready", function (event) {
  // properties follow the same name as class properties.

  ng7Test["stringInput"] = "hello";
  ng7Test["numberInput"] = 300;
  ng7Test["objectInput"] = {"beep":"boop"};
  ng7Test["arrayInput"] = [1, 2, 3];
});
```
