+++
draft = false
title = "Web components implementation on React."
date = "2019-01-12T12:08:00+01:00"
tags = ["React", "Web Components"]
categories = ["FrontEnd", "Web Components"]
series = []
images = []
description = ""
summary = """
Dive in Web components implementation on React.
"""

+++

Oh my... how difficult was to make it work.

First, Web components are not fully supported by React, period.
The example in their web, is not complete and not fully working. You have to
rely on another library, that it's also buggy: [react-web-component](https://github.com/spring-media/react-web-component)

So you will face (atm) some issues (I could found more...):

* [react-web-component/13](https://github.com/spring-media/react-web-component/issues/13)
* [react/9242](https://github.com/facebook/react/issues/9242)
* [react/10422](https://github.com/facebook/react/issues/9242)
* [react/11347](https://github.com/facebook/react/issues/11347)
* [react/7901](https://github.com/facebook/react/issues/7901)
* [Props are Read-Only](https://reactjs.org/docs/components-and-props.html#props-are-read-only)


With all that in mind I modify `react-web-component` so:

* works with `connectedCallback` thanks to a
[react/11 pull-request](https://github.com/spring-media/react-web-component/pull/11)
was easy.

* foward the host element and attributes to a hook, so it can be used inside
the component.


## Create React App

{{< highlight bash >}}
npx create-react-app poc-webc-react16
cd poc-webc-react16

# We will install my fork atm. because contains all fixes.
# npm i --save react-web-component
# npm i --save git@github.com/llafuente/react-web-component.git#c63b9b3d14ce78f0ea16c0e2a3eb780d0678ddda
npm install --save https://github.com/llafuente/react-web-component/tarball/master

{{< /highlight >}}


## Polyfills

{{< highlight bash >}}
npm i --save @webcomponents/custom-elements @webcomponents/shadydom
{{< /highlight >}}

Edit `/src/index.js`, add at the top:

{{< highlight jsx >}}
import 'react-app-polyfill/ie9';
import 'react-app-polyfill/ie11';

import '@webcomponents/custom-elements';
import '@webcomponents/shadydom';
{{< /highlight >}}


## Disable react App and define web-component


This is the final `/src/index.js` (just easier to read)

{{< highlight jsx >}}
import 'react-app-polyfill/ie9';
import 'react-app-polyfill/ie11';

import '@webcomponents/custom-elements';
import '@webcomponents/shadydom';

import './index.css';
import App from './App';
import ReactWebComponent from 'react-web-component/src/dev/';

ReactWebComponent.create(<App />, 'react16-test');
{{< /highlight >}}

Now your entire React App is a custom-element.


## Create Component

Just to test, edit App directly `/src/App.js`.

{{< highlight jsx >}}
import React, { Component } from 'react';

// do not import any CSS on this project or will be leacked
// import './App.css';

class App extends Component {
  hostElement;

  // react-web-componet hook
  webComponentConstructed(hostElement, attributes) {
    this.hostElement = hostElement;

    attributes = attributes || {};
    this.setState(attributes);
  }

  render() {
    // no state no render
    if (!this.state) {
      return (<div></div>);
    }

    // state -> render

    // CSS must fo into a variable!
    var css = `h3 {
  color: red;
}`

    // here HTM
    return (
      <div className="App">
        <style type="text/css">{styles}</style>

        <h3>This is a webcomponent</h3>

        <pre>mad {state.mad}</div>pre>

        <button onClick={this.onFire.bind(this)}>Fire an event</button>
      </div>
    );
  }

  onFire() {
    this.hostElement.dispatchEvent(new CustomEvent("fire", {detail: "all missiles!"}))
  }
}

export default App;
{{< /highlight >}}


## Use it

For development you can use this `/public/index.html`

{{< highlight html >}}
<!-- replace this body -->
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <react16-test mad="science"></react16-test>
  </body>
{{< /highlight >}}

But it will give you the following error when building your app: `npm run build`

```
  - htmlparser.js:240 new HTMLParser
    [poc-webc-react16]/[html-minifier]/src/htmlparser.js:240:13

  - htmlminifier.js:966 minify
    [poc-webc-react16]/[html-minifier]/src/htmlminifier.js:966:3

  - htmlminifier.js:1326 Object.exports.minify
    [poc-webc-react16]/[html-minifier]/src/htmlminifier.js:1326:16

  - index.js:411 HtmlWebpackPlugin.postProcessHtml
    [poc-webc-react16]/[html-webpack-plugin]/index.js:411:34

  - index.js:246 Promise.all.then.then
    [poc-webc-react16]/[html-webpack-plugin]/index.js:246:27
```

Remember that editing index.html dont reload the view,
you need a hard-reload (F5)

To fix it: instance the component using JS:

{{< highlight html >}}
<!-- replace this body -->
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <script type="text/javascript">
      // fix: 'npm run build' using document.createElement
      // - htmlparser.js:240 new HTMLParser
      // [poc-webc-react16]/[html-minifier]/src/htmlparser.js:240:13
      // ...

      var el = document.createElement("react16-test")
      document.body.appendChild(el);
      el.setAttribute("mad", "science");
    </script>
  </body>

{{< /highlight >}}


## Conclusions

React 16 is not prepared for Web Components. I found many problems in a small,
but enough, time:

* Any CSS file imported, will be leacked globally.
Do not `import '*.css'`

* You are forced to write CSS, HTML and JS in the same file.
Should be in different files but I don't mind...
but declare CSS in a variable and inject that into a *style* tag
is inconvenient.

* Project can't be built if the Web Component is used at `index.html`... really
:sob: but has a work around...

* In React everything is a property, while using Web Component the easy way
is to use attributes.

* No clear future, there are many discussions on the fly and some already
promise breaking changes.

* There is no official support.

Just: Do not use React 16 to implement Web Components,
wait more or use another framework.

Project size: under 300Kb.


## References

* [Use Shadow DOM on react](https://github.com/Wildhoney/ReactShadow) maybe
an alternative to my implementation.

* [Create a New React App](https://reactjs.org/docs/create-a-new-react-app.html)

* [react-web-component](https://github.com/spring-media/react-web-component)
