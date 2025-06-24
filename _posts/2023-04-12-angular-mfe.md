---
title: Building MFEs in Angular
date: 2023-04-12 12:00:00 +0530
categories: [frontend,angular]
tags: [frontend,angular,micro-frontend]
---

# Web Components

Web components are a set of web platform APIs that allow developers to create reusable, encapsulated components for the web. Web components consist of three main parts:

- Custom Elements: Custom elements allow developers to define their own HTML tags and use them in their web applications. Custom elements can be used to create reusable components that can be used across different web applications.
- Shadow DOM: Shadow DOM is a way of encapsulating the styles and markup of a web component, so that it does not affect the rest of the web page.
- HTML Templates: HTML templates are a way of defining reusable HTML structures that can then be reused multiple times as the basis of a custom element's structure.

Web components are supported by most modern web browsers, including Chrome, Firefox, Safari, and Edge. Refer [MDN Doc](https://developer.mozilla.org/en-US/docs/Web/Web_Components) for more information.

# Creating MFE in Angular using Web Components

Angular is a popular framework for building web applications. It provides a powerful set of tools for building complex, interactive applications. Angular can also be used to create micro frontends using the **web components** architecture.

To create a web component in Angular, we can use the `@angular/elements` package. This package provides a way of converting Angular components into web components that can be used in any web application.

Here's a step-by-step guide to create a micro frontend in Angular using web components:

### Step 1: Create an Angular application

First, we need to create an Angular application that will contain our web component. You can create a new Angular application using the Angular CLI:

`ng new mfe-app`

### Step 2: Create an Angular Component

Now we need to create an Angular component that we want to convert into a web component. For this example, we will create a simple component that displays a greeting:

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'my-greeting',
  template: `Hello, {{ name }}! <br />
    <button (click)="onGetGreetingsClick()">Get greetings</button>`,
})
export class GreetingComponent {
  @Input() name!: string;
  @Output() getGreetingClick = new EventEmitter<string>();

  onGetGreetingsClick(): void {
    this.getGreetingClick.emit(`Hello, ${this.name}!`);
  }
}
```

This component takes a name input, displays a greeting and emits an event with the greeting string on button click.

### Step 3: Convert the Angular Component into a Web Component

The `@angular/elements` package provides a way to convert Angular components into custom elements that can be used outside of Angular. It does this by exposing the `createCustomElement` function, which takes an Angular component and converts it into a custom element that can be used with vanilla JavaScript, React, Vue, and other frameworks.

Let's first install the package `@angular/elements`:

`npm install @angular/elements --save`

Next, we need to convert the Angular component into a web component. We can do this using the **createCustomElement** function from the **@angular/elements** package. Here's how we do it in the root module:

```typescript
import { Injector, NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { createCustomElement } from '@angular/elements';
import { GreetingComponent } from './greeting.component';

@NgModule({
  declarations: [GreetingComponent],
  imports: [BrowserModule],
  entryComponents: [GreetingComponent],
})
export class AppModule implements DoBootstrap {
  constructor(private injector: Injector) {}

  ngDoBootstrap(): void {
    const el = createCustomElement(GreetingComponent, {
      injector: this.injector,
    });
    customElements.define('my-greeting', el);
  }
}
```

This code converts our `GreetingComponent` into a web component with the name `my-greeting`.

### Step 4: Build the component

When we build an angular application we get 3 javascript files **main.js**, **polyfills.js** and **runtime.js**. For mfe we need to merge these three files using the below script. Create a file called **build-elements.js** in the root directory of the application and add below code in it and install `fs-extra` and `concat` npm packages as dev dependencies: `npm i -D fs-extra concat`

```javascript
const fs = require('fs-extra');
const concat = require('concat');

(async function build() {
  try {
    const project = process.argv.slice(2)[0];
    const srcPath = `./dist/${project}`;
    const dstPath = `./elements/${project}`;
    const filesToBeMerged = [
      `${srcPath}/runtime.js`,
      `${srcPath}/polyfills.js`,
      `${srcPath}/main.js`,
    ];

    await fs.ensureDir(dstPath);
    await concat(filesToBeMerged, `${dstPath}/${project}-element.js`);
    await fs.copyFile(
      `${srcPath}/styles.css`,
      `${dstPath}/${project}-styles.css`
    );

    await fs.ensureDir(`${srcPath}/assets`);
    await fs.copy(`${srcPath}/assets`, `${dstPath}/assets`);
    console.info('Elements built successfully');
  } catch (e) {
    console.error('Error while building elements', e);
  }
})();
```

Add below script in `package.json`

```json
"build:element": "ng build --configuration prod --output-hashing none && node build-elements.js mfe-app"
```

Then run `npm run build:element` command to get the mfe files (`mfe-app-element.js` and `mfe-app-styles.css` in our case) in the **elements** directory which needs to be deployed.

### Step 5: Use the web component

Reference the `js` and `css` files of the mfe in the host application and use the custom element (`my-greeting` in our case) as a normal html tag:

Note: since the events emitted by the custom elements are of type [`CustomEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent), make sure to use the `detail` property to get the emitted value from the mfe

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <script src="./mfe-app-element.js" type="text/javascript"></script>
    <link href="./mfe-app-styles.css" rel="stylesheet" />
  </head>
  <body>
    <my-greeting name="Beerendra"></my-greeting>
    <script>
      const mfeEl = document.querySelector('my-greeting');
      mfeEl.addEventListener('getGreetingClick', (event) => {
        console.log(event.detail);
      });
    </script>
  </body>
</html>
```

# Deployment and Versioning Options

There are several deployment options available for micro frontends built using web components, regardless of the specific framework used to build them. Here are a few:

1. **Deploy on CDN**: Each micro frontend can be built as a standalone web component using web standards like Custom Elements and Shadow DOM. You can then package each web component as a separate file and deploy it to a CDN or hosting service, where it can be consumed by other applications.

   When serving micro-frontend files directly from a CDN, versioning your files is an important consideration. Versioning your files can help ensure that changes to your micro-frontends are propagated to users and that they don't experience issues due to stale files being served from the CDN cache. Here are a few strategies for versioning micro-frontend files in a CDN:

   a. **Query parameter versioning**: One approach is to use a query parameter to version your micro-frontend files. For example, you might append a query parameter like ?v=1.0.0 to the URLs of your JavaScript and CSS files. When you make changes to your micro-frontend files, you can update the version number to force the CDN to serve the updated files to users.

   b. **Filename versioning**: Another approach is to version your files by including the version number in the filename. For example, you might include the version number in the filename like mfe-element.1.0.0.js. When you make changes to your files, you can update the version number in the filename to force the CDN to serve the updated files to users.

2. **Deploy as a static application**: You can also deploy micro frontend files as a static application, which can be hosted on its own domain or subdomain. This approach allows you to deploy each micro frontend independently and scale them separately based on their usage.

   a. **URL path versioning**: To make it clear which version of a micro frontend a user is accessing, include the version number in the URL. For example, instead of using a URL like `https://myapp.com/mfe-element.js`, use a URL like `https://myapp.com/v1.0/mfe-element.js`.

   b. **Filename versioning**: Another approach is to version your files by including the version number in the filename. For example, you might include the version number in the filename like `mfe-element.1.0.0.js`.

   In addition to including the version number in your static file names or paths, you can use **cache-busting techniques** to ensure that new versions of your files are downloaded by the browser. One common technique is to append a unique query string to the file reference in your HTML, like this:

   ```html
   <link rel="stylesheet" href="styles/mfe-element-styles.css?v=1.0" />
   ```

   This query string will force the browser to download the latest version of the file, even if it has a cached version with the same filename.

3. **Deploy as an NPM package**: If you're using a build tool like Webpack or Rollup to package your web components, you can also package each micro frontend as a separate NPM package and deploy it to a private or public NPM registry. This approach allows you to version and distribute each micro frontend independently.

   For handling versioning we can use **semantic versioning**. Semantic versioning is a widely adopted standard for versioning software components. It uses three numbers separated by periods (e.g., 1.2.3) to indicate major, minor, and patch versions. Major versions indicate major changes that may not be backward compatible, minor versions indicate new features or functionality that are backward compatible, and patch versions indicate bug fixes or small changes that are backward compatible. This can help other developers who consume your micro frontends understand what changes are included in each version.
