---
author: Jan Hajek
title: Getting React and Office UI Fabric work with PowerApps component framework
slug: getting-react-and-office-ui-fabric-work-with-powerapps-component-framework
id: 661
date: '2019-06-06 09:00:45'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
  - Open Source
tags:
  - Office UI Fabric
  - PowerApps
  - PowerApps Component Framework
  - React
---

[PowerApps component framework](https://docs.microsoft.com/en-us/powerapps/developer/component-framework/overview) has been in public preview for a while now. While it allows you to create wonderful customizations, you may want to make use of React and Office Fabric UI for your components. In this article, we are going to show you how.

First, you need to [install PowerApps CLI](https://docs.microsoft.com/en-us/powerapps/developer/component-framework/create-custom-controls-using-pcf#prerequisites-to-use-powerapps-cli) and then [scaffold a component](https://docs.microsoft.com/en-us/powerapps/developer/component-framework/create-custom-controls-using-pcf#creating-a-new-powerapps-component-framework-component):

    pac pcf init --namespace TheNetworg --name PCFReact --template field

Next, you need to add React into your project. Start with adding the packages (React and ReactDOM) via npm:

    npm install react@16 react-dom@16

Next, you need the component to actually carry the library with itself and load it if not available. In order to properly include the library, you need to go to the _ControlManifest_ and modify the _resources_ element:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-1-xml">View this gist on GitHub</a></noscript>

</div>

Once you have this done, you can proceed with creating a sample React Component like this (_control.tsx_ in the same directory where _index.ts_ is):

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-2-tsx">View this gist on GitHub</a></noscript>

</div>

Next you have to modify _index.ts_ in order to render your custom React Component. Simply import your create control:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-3-tsx">View this gist on GitHub</a></noscript>

</div>

Then in _init_, all you have to do is call the render method to render the created component:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-4-ts">View this gist on GitHub</a></noscript>

</div>

<figure class="wp-block-image">![](/uploads/2019/06/pcf1-1024x161.jpg)</figure>

**Yay! Now we have React working within PCF!**

Lets take it a step further - lets add support for Office Fabric UI! To start with, you need to add Office Fabric UI via npm:

    npm install office-ui-fabric-react@6

_Tip:_ When including different versions, make sure that React, ReactDOM and Office UI Fabric React is compatible with each other (I spent about an hour trying to figure out what's wrong).

Next, you need to modify the _ControlManifest_ to include the library (and change the load order):

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-5-xml">View this gist on GitHub</a></noscript>

</div>

Next we go back to _control.tsx_ and modify it to include Office Fabric UI and add some component:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-6-tsx">View this gist on GitHub</a></noscript>

</div>

And then all you have to do is to call it from _index.ts_:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-7-ts">View this gist on GitHub</a></noscript>

</div>

Once you run the control, you will end up with Office Fabric UI control working!

<figure class="wp-block-image">![](/uploads/2019/06/pcf3-1024x150.jpg)</figure>

Great! What about changing _index.ts_ to _index.tsx_? By default, it doesn't work, but if you want it to work, you have to do a _little hack_. You have to go to the _pcf-scripts_ module (_node_modules\pcf-scripts\controlcontext.js_) and modify the line 34 (at the time of writing) to include _.tsx_ files:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-8-ts">View this gist on GitHub</a></noscript>

</div>

Now, you can have _index.tsx_ file and render the components like you are used to in regular React apps:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/hajekj/09d7b548a3f2009f642be96320a887bb#file-661-9-tsx">View this gist on GitHub</a></noscript>

</div>

Sidenote regarding the resource loading:

In [some posts](https://www.linkedin.com/feed/update/urn:li:activity:6541973822571196416/), you can find another way to add React support by forcing the use of React by adding it into the rendering template. While this is going to work - because PowerApps UI is built in React, there is no guarantee that React and other dependencies will be available (like Office Fabric UI).

Also, internally, Dynamics has a way to figure out the dependencies in a single page for multiple components (and load them only once - super smart!), so you should be definitely using the manifest way, rather than relying on React and other libraries being actually present on the page.

<figure class="wp-block-image">![](/uploads/2019/06/pcf2-1024x109.jpg)</figure>

_Footnote:_ This article doesn't demonstrate end to end integration (input and output), however it should be super simple after this point.