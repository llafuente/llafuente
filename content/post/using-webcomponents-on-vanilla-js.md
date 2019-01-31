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

In this series I implemented a Web component on Angular 6 / 7, Polymer and
React. All are *the same*, so it should be trivial to use them... let's see.

First I build everything.

{{< highlight bash >}}
cd poc-webc-angular6/
ng build --prod --outputHashing=none
cd ..

cd poc-webc-angular7/
ng build --prod --outputHashing=none
cd ..

cd poc-webc-react16/
npm run build
cd ..

cd poc-webc-polymer3/
npm run build --no-hash
cd ..
{{< highlight bash >}}

Then I copied everything in poc-use-vanilla-js
{{< highlight bash >}}

mkdir -p poc-use-vanilla-js/poc-webc-angular6/
mkdir -p poc-use-vanilla-js/poc-webc-angular7/
mkdir -p poc-use-vanilla-js/poc-webc-react16/
mkdir -p poc-use-vanilla-js/poc-webc-polymer3/

cp -rf poc-webc-angular6/dist/poc-webc-angular6/* poc-use-vanilla-js/poc-webc-angular6/
cp -rf poc-webc-angular7/dist/poc-webc-angular7/* poc-use-vanilla-js/poc-webc-angular7/
cp -rf poc-webc-react16/build/* poc-use-vanilla-js/poc-webc-react16/
cp -rf poc-webc-polymer3/dist/* poc-use-vanilla-js/poc-webc-polymer3/

{{< /highlight >}}


And face the first big problem. Angular 6 / 7 and React can't live in the
same page. And the problem is webpack.

the solution is to use:

* https://webpack.js.org/configuration/output/#output-jsonpfunction

So again a tool chain problem. Add `jsonpfunction` to all tool chain and
everything work as expected? Not really.

React render the component empty, no attributes/properties read/used.
Angular and Polymer listen to changes after creation, but React not.

After `document.createElement` React is just ready and stop. I have to add
the hook: `webComponentAttributeChanged` to listen to attributes changes and
manually call it for properties.
