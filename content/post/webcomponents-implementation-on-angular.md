+++
draft = false
title = "Web components implementation on Angular 7 / 6."
date = "2018-12-07T12:08:00+01:00"
tags = ["Angular", "Web Components"]
categories = ["FrontEnd", "Web Components"]
series = []
images = []
description = ""
summary = """
Dive in Web components implementation on Angular 7 / 6 on all mayor browsers!
"""

+++

There is no difference between Angular 6 and 7, just how you install the cli and
some names (if you want to be clear), so I cover both in the same article.

Also I will be very concise just `step by step reference`.


## Angular 7 create project

{{< highlight bash >}}
npm i -g @angular/cli@7
ng new poc-webc-angular7 --prefix wc-ng7

cd poc-webc-angular7
{{< /highlight >}}

## Angular 6 create project

{{< highlight bash >}}
npm i -g @angular/cli@6
ng new poc-webc-angular7 --prefix wc-ng6

cd poc-webc-angular7
{{< /highlight >}}


## Add polyfills

To run and create Web Components on Angular you need polyfill for all browsers,
even chrome that it's supose to fully support Web Components native.

{{< highlight bash >}}
# this will add custom-element support to Angular
ng add @angular/elements

# polyfills are necessary for all browsers
# fix edge: ERROR TypeError: Object doesn't support property or method 'createShadowRoot'
# fix chrome: elements.js:384 Uncaught TypeError: Failed to construct 'HTMLElement': Please use the 'new' operator, this DOM object constructor cannot be called as a function.
# fix firefox: TypeError: Constructor HTMLElement requires 'new'
npm i @webcomponents/custom-elements @webcomponents/webcomponentsjs --save

{{< /highlight >}}

Edit file: `src/polyfills.ts`

{{< highlight typescript >}}
// uncomment import 'core-js* to support ie11

//...

/***************************************************************************************************
 * APPLICATION IMPORTS
 */
//
// add Web Components polyfills
//

import "@webcomponents/custom-elements/src/native-shim";

// chrome (✓) firefox (✓) edge (✓) ie11 (x)
// import "@webcomponents/custom-elements/src/custom-elements.js";

// ie11 do not support es6 class, so it need to be es5
// chrome (✓) firefox (✓) edge (✓) ie11 (✓)
import "@webcomponents/webcomponentsjs/custom-elements-es5-adapter.js";
{{< /highlight >}}


## Create the webcomponent

This step is key. I give you all the error in all browsers depending on
the encapsulation you choose.

Each encapsulation has it's limitations (I'm sure!) so you should refer to
Angular docs about
[ViewEncapsulation](https://angular.io/api/core/ViewEncapsulation)
but I can't find more specific information...

{{< highlight bash >}}
# now create a TestComponent... but it's key how
# remember all browsers: chrome, firefox, edge, ie11

# ViewEncapsulation.Native: chrome (✓) firefox (x) edge (x) ie11 (x)
# edge error: Object doesn't support property or method 'createShadowRoot'
# firefox error: hostEl.createShadowRoot is not a function
ng g component test --inline-style --inline-template -v Native

# ViewEncapsulation.ShadowDom: chrome (✓) firefox (✓) edge (x) ie11 (x)
# edge: Object doesn't support property or method 'attachShadow'
ng g component test --inline-style --inline-template -v ShadowDom


# ViewEncapsulation.Emulated: chrome (✓) firefox (✓) edge (✓) ie11 (✓)
# this works on every browser
ng g component test --inline-style --inline-template -v Emulated
{{< /highlight >}}

So to support all browsers we will need `ViewEncapsulation.Emulated`.

## Module declaration

Default Angular Module bootstrap a Component.
For Web Components we just want to declare those components
as `custom elements`.

Edit: `src/app/app.module`

{{< highlight typescript >}}
// ...
import { Injector } from '@angular/core';
import { createCustomElement } from "@angular/elements";
// ...

@NgModule({
  // remove bootstrap, leave the rest
  // add
  entryComponents: [TestComponent]
})
export class AppModule {

  constructor(private injector: Injector) {}

  ngDoBootstrap() {
    const el = createCustomElement(TestComponent, {
      injector: this.injector
    });

    customElements.define("ng7-test", el); // or ng6-test
  }
}
{{< /highlight >}}


Edit: `/src/index.html` @ head

{{< highlight html >}}
<head>
  <!-- add -->
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
</head>
{{< /highlight >}}


Edit: `/src/index.html` @ body

{{< highlight html >}}
<body>
    <ng7-test></ng7-test>
    <!-- <ng6-test></ng6-test> -->
</body>
{{< /highlight >}}

Now test it on all browsers!

{{< highlight bash >}}
ng build
# or ng build --prod
static dist\poc-webc-angular7\
# or static dist\poc-webc-angular6\
{{< /highlight >}}

## Conclusions

Implementation was easy and direct. We obviously `fake` Web Components to
make it work on all browsers, but just a single change at `ViewEncapsulation`
makes everything fully compliant.

I can't make it work on firefox with `ViewEncapsulation.Native` I suspect
the polyfills are not ready yet, when you read this, it may work.

Also edge is in development: [ShadowDom](https://developer.microsoft.com/en-us/microsoft-edge/platform/status/shadowdom/?q=ShadowDom) and [custom elements](https://developer.microsoft.com/en-us/microsoft-edge/platform/status/customelements/?q=compon) will be eventualy compatible.

Maybe in one/two years, all browsers are ready and ie11 finally dead, so
everything can be Native.

Project size: 312Kb. I'm very pleased :)
