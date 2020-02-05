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

```xml
<resources>
  <library name="React" version=">=1" order="1">
    <packaged_library path="../node_modules/react/umd/react.production.min.js" version="16.8.6" />
  </library>
  <library name="ReactDOM" version=">=1" order="2">
    <packaged_library path="../node_modules/react-dom/umd/react-dom.production.min.js" version="16.8.6" />
  </library>
  <code path="index.ts" order="3"/>
<resources>
```

Once you have this done, you can proceed with creating a sample React Component like this (_control.tsx_ in the same directory where _index.ts_ is):

```typescript
import * as React from 'react';
 
interface IProps {
    compiler: string,
    framework: string,
    bundler: string
}
 
export class Hello extends React.Component<IProps, {}> {
    render() {
        return <h1>This is a {this.props.framework} application using {this.props.compiler} with {this.props.bundler}</h1>
    }
}
```

Next you have to modify _index.ts_ in order to render your custom React Component. Simply import your create control:

```typescript
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { Hello } from './control';
```

Then in _init_, all you have to do is call the render method to render the created component:

```typescript
public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    ReactDOM.render(React.createElement(Hello, {
        bundler: "Webpack",
        compiler: "Typescript",
        framework: "React"
    }), container);
}
```

![](/uploads/2019/06/pcf1-1024x161.jpg)

**Yay! Now we have React working within PCF!**

Lets take it a step further - lets add support for Office Fabric UI! To start with, you need to add Office Fabric UI via npm:

    npm install office-ui-fabric-react@6

_Tip:_ When including different versions, make sure that React, ReactDOM and Office UI Fabric React is compatible with each other (I spent about an hour trying to figure out what's wrong).

Next, you need to modify the _ControlManifest_ to include the library (and change the load order):

```xml
<resources>
  <library name="React" version=">=1" order="1">
    <packaged_library path="../node_modules/react/umd/react.production.min.js" version="16.8.6" />
  </library>
  <library name="ReactDOM" version=">=1" order="2">
    <packaged_library path="../node_modules/react-dom/umd/react-dom.production.min.js" version="16.8.6" />
  </library>
  <library name="OfficeFabricUI" version=">=1" order="3">
    <packaged_library path="../node_modules/office-ui-fabric-react/dist/office-ui-fabric-react.js" version="6.180.0" />
  </library>
  <code path="index.tsx" order="4"/>
</resources>
```

Next we go back to _control.tsx_ and modify it to include Office Fabric UI and add some component:

```typescript
import * as React from 'react';
import { TagPicker, ITag } from 'office-ui-fabric-react';
 
const _testTags: ITag[] = [
    'black', 'blue', 'brown', 'cyan', 'green', 'magenta', 'mauve', 'orange', 'pink', 'purple', 'red', 'rose', 'violet', 'white', 'yellow'
].map(item => ({ key: item, name: item }));
 
export class HelloFabric extends React.Component<{}> {
    render() {
        return (
            <TagPicker
                onResolveSuggestions={this._onFilterChanged}
                getTextFromItem={this._getTextFromItem}
                pickerSuggestionsProps={{
                    suggestionsHeaderText: 'Suggested Tags',
                    noResultsFoundText: 'No Color Tags Found'
                }}
                itemLimit={2}
                inputProps={{
                    onBlur: (ev: React.FocusEvent<HTMLInputElement>) => console.log('onBlur called'),
                    onFocus: (ev: React.FocusEvent<HTMLInputElement>) => console.log('onFocus called'),
                    'aria-label': 'Tag Picker'
                }}
            />
        );
    }
    private _getTextFromItem(item: ITag): string {
        return item.name;
    }
    private _onFilterChanged = (filterText: string, tagList: ITag[] | undefined): PromiseLike<ITag[]> => {
        return new Promise((resolve, reject) => {
            resolve(filterText
                ? _testTags
                    .filter(tag => tag.name.toLowerCase().indexOf(filterText.toLowerCase()) === 0)
                    .filter(tag => !this._listContainsDocument(tag, tagList))
                : []);
        });
    };
    private _listContainsDocument(tag: ITag, tagList?: ITag[]) {
        if (!tagList || !tagList.length || tagList.length === 0) {
            return false;
        }
        return tagList.filter(compareTag => compareTag.key === tag.key).length > 0;
    }
}
```

And then all you have to do is to call it from _index.ts_:

```typescript
public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    ReactDOM.render(React.createElement(HelloFabric, {}), container);
}
```

Once you run the control, you will end up with Office Fabric UI control working!

![](/uploads/2019/06/pcf3-1024x150.jpg)

Great! What about changing _index.ts_ to _index.tsx_? By default, it doesn't work, but if you want it to work, you have to do a _little hack_. You have to go to the _pcf-scripts_ module (_node_modules\pcf-scripts\controlcontext.js_) and modify the line 34 (at the time of writing) to include _.tsx_ files:

```typescript
if (fileType !== '.ts' &amp;&amp; fileType !== '.tsx') {
```

Now, you can have _index.tsx_ file and render the components like you are used to in regular React apps:

```typescript
public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    var hello = container.appendChild(document.createElement("div"));
    var helloFabric = container.appendChild(document.createElement("div"));
    ReactDOM.render(<Hello compiler="Typescript" framework="React" bundler="Webpack" />, hello);
    ReactDOM.render(<HelloFabric />, helloFabric);
}
```

Sidenote regarding the resource loading:

In [some posts](https://www.linkedin.com/feed/update/urn:li:activity:6541973822571196416/), you can find another way to add React support by forcing the use of React by adding it into the rendering template. While this is going to work - because PowerApps UI is built in React, there is no guarantee that React and other dependencies will be available (like Office Fabric UI).

Also, internally, Dynamics has a way to figure out the dependencies in a single page for multiple components (and load them only once - super smart!), so you should be definitely using the manifest way, rather than relying on React and other libraries being actually present on the page.

![](/uploads/2019/06/pcf2-1024x109.jpg)

_Footnote:_ This article doesn't demonstrate end to end integration (input and output), however it should be super simple after this point.