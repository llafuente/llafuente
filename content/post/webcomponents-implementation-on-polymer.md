+++
draft = false
title = "Web components implementation on Polymer."
date = "2018-12-15T12:08:00+01:00"
tags = ["Polymer", "Web Components"]
categories = ["FrontEnd", "Web Components"]
series = []
images = []
description = ""
summary = """
Dive in Web components implementation on Polymer 3 using PolymerElement and LitElement on all mayor browsers!
"""

+++

Unlike Angular this won't be a tutorial because I can't find a way
to easily scaffold a Polymer 3 project with TypeScript, so I have to create
most of them, it's boring to do tool chains and I won't bore you.

So we will jump directly to the component itself.

### Create a Web Component using PolymerElement

PolymerElement has types since `3.0.5`, I tested `3.1.0` lucky! Older versions
will give you many problems, I don't downgrade successfully without many
unnecessary castings, but should be possible.

{{< highlight typescript >}}
import { PolymerElement, html } from '@polymer/polymer/polymer-element';
import { property } from '@polymer/decorators';
import * as view from './polymer-element.template.html';

export class MyApp extends PolymerElement {
  @property({ type: String })
  stringInput: string = "default";
  @property({ type: Number })
  numberInput: number = 0;
  @property({ type: Object })
  objectInput: any = {};
  @property({ type: Array })
  arrayInput: string[] = [];

  constructor() {
    super();
  }

  static get template() {
    console.log("template", view);

    //return html`${view}`;
    return html`
<h2>polymer3-polymer-element</h2>

<h3>Inputs</h3>

<pre>string: [[stringInput]]
number: [[numberInput]]
objectInput: [[objectInput]]
objectInput.beep: [[objectInput.beep]]
type objectInput: [[typeof objectInput]]
arrayInput: [[arrayInput]]
arrayInput access first element: [[arrayInput.0]]
</pre>

<h3>Outputs</h3>
<button on-click="clickOutputString">output string</button>
<button on-click="clickOutputNumber">output number</button>
<button on-click="clickOutputObject">output object</button>


<h3>Slot</h3>

<slot></slot>
`;
  }

  clickOutputString() {
    this.dispatchEvent(new CustomEvent('outputString', {detail: "this is a string!"}));
  }

  clickOutputNumber() {
    this.dispatchEvent(new CustomEvent('outputNumber', {detail: 300}));
  }

  clickOutputObject() {
    this.dispatchEvent(new CustomEvent('outputObject', {detail: { success: true }}));
  }
}

// use directly the API
customElements.define('polymer3-polymer-element', MyApp)
{{< /highlight >}}

### Create a Web Component using LitElement

LitElement is the future of Polymer, it's written in TypeScript and it's safer
to use than PolymerElement. Safer does not mean safe... because for performance
reasons they still include some "non-typed" functions that can ruin your day.

{{< highlight typescript >}}
import { LitElement, html, property, customElement } from '@polymer/lit-element';

@customElement('polymer3-lit-element')
export class MyApp extends LitElement {
  @property({ type: String, attribute: 'string-input' })
  stringInput: string = "default";
  @property({ type: Number, attribute: 'number-input' })
  numberInput: number = 0;
  @property({ type: Object, reflect: true, attribute: 'object-input' })
  objectInput: any = {};
  @property({ type: Array, attribute: 'array-input' })
  arrayInput: string[] = [];

  constructor() {
    super();
  }

  render() {
    return html`
<h2>polymer3-lit-element</h2>

<h3>Inputs</h3>

<pre>string: ${this.stringInput}
number: ${this.numberInput}
objectInput: ${JSON.stringify(this.objectInput)}
objectInput.beep: ${this.objectInput.beep}
type objectInput: ${typeof this.objectInput}
arrayInput: ${this.arrayInput}
arrayInput access first element: ${this.arrayInput}
</pre>

<h3>Outputs</h3>

<button @click="${this.clickOutputString}">output string</button>
<button @click="${this.clickOutputNumber}">output number</button>
<button @click="${this.clickOutputObject}">output object</button>


<h3>Slot</h3>

<slot></slot>
`;
  }

  clickOutputString() {
    this.dispatchEvent(new CustomEvent('outputString', {detail: "this is a string!"}));
  }

  clickOutputNumber() {
    this.dispatchEvent(new CustomEvent('outputNumber', {detail: 300}));
  }

  clickOutputObject() {
    this.dispatchEvent(new CustomEvent('outputObject', {detail: { success: true }}));
  }
}
{{< /highlight >}}

#### Instancing outside (Plain HTML)

{{< highlight html >}}
<polymer3-lit-element
  id="lit-test"
  string-input="hello"
  number-input="300"
  object-input={"beep":"boop"}
  array-input="[1, 2, 3]">slot-contents!</polymer3-lit-element>
{{< /highlight >}}

#### Instancing inside Polymer

{{< highlight html >}}
<polymer3-lit-element
  id="lit-test"
  string-input="hello"
  number-input="300"
  .object-input="{{yourObjectVar}}"
  .array-input="{{yourArrayVar}}">slot-contents!</polymer3-lit-element>
{{< /highlight >}}

### Add polyfills

Polyfills in polymer is handled by you. Instead of bundling all inside a
single js, like the other, I do what everyone does... just add script at
HTML.head

{{< highlight html >}}
<!DOCTYPE html>
<html>

<head>
    <!-- ... -->
    <script src="webcomponentsjs/bundles/webcomponents-sd-ce-pf.js"></script>
</head>
{{< /highlight >}}



## Conclusions

Found a TypeScript tool chain to start was the worst part.
On Vanilla JS I have a test component after 30 minutes.
Versus a few hours to make the entire TypeScript tool chain.

Also I can't find a way to include the template as a file.
I have to declare the template inside the TypeScript file.
I don't really mind, but some people does...

Big thumbs up for LitElement typing (PolymerElement it's more verbose). Also
parsing JSON-strings to JS Object/Array was a big surprise and productivity
boost. LitElement was the right move that Polymer needs to keep up with
other frameworks.

Project size: under over 375Kb (polymer) + 107Kb (polyfills). Surprise!
