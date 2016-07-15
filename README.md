# All about HTML Custom Elements
Custom Elements is a W3C "working draft" [specification][spec] that
provides a mechanism for defining new behaviors (such as dynamic
content or interactivity) for HTML elements with custom names.

Custom elements are just HTML elements, with all of the methods and
properties of other, built-in elements. The only real constraint[*](#gotchas)
is that their names must contain at least one hyphen (`-`).

```html
<message-element>Hi!</message-element>

<super-section>
  <p>Custom elements can contain content!</p>
  <message-element>And other custom elements!</message-element>
</super-section>

You can also extend built-in elements using the "is" attribute:
<input is="date-picker" type="date">
```

## Table of Contents
1. [Why custom elements?](#why-custom-elements)
1. [How do they work?](#how-do-they-work)
1. [Browser support](#browser-support)
1. [Custom Elements v1 API](#v1)
1. [Type extension](#type-extension)
1. [Custom Elements v2 API](#v2)
1. [Gotchas](#gotchas)
1. [Polyfills](#polyfills)
1. [Frameworks](#frameworks)


## Why Custom Elements?
* **Encapsulation.** Custom element names avoid ambiguity in
  markup (versus, say, a `<div>` or `<span>` with a "special"
  class), and provide a solid foundation for scoped styles.

  * If you've ever felt like it was wrong to define reuseable
    components with "special" class names initialized by jQuery
    selections that can only be called after the DOM is ready (or
    rely on mutation observers to watch for new instances), then
    custom elements could be your new jam.

  * In native implementations (see [browser support](#browser-support)),
    you can target custom elements before (`:unresolved` in the
    [v1 spec]) and after (`:defined` in the [v2 spec][spec]) they've
    been registered via JavaScript.

* **The DOM _is_ the API.** Components built with tools like jQuery
  can be cumbersome to build, modify, and maintain because they often
  introduce another layer of abstraction (such as the `jQuery` object
  and its API) on top of the DOM. And while there's nothing to stop
  one from building custom elements that use jQuery (or D3, or even
  [React]) under the hood, I've found custom elements made with
  vanilla JS to be easier to grok and read.

* **Web Standards.** Custom elements are a working draft W3C
  [specification][spec]. That means that—in theory, at least—they
  will _eventually_ be implemented natively in all modern browsers
  with the same API. The same cannot be said for other tools that
  implement the more general concept of _web components_, such as
  [Angular], [React], or [Vue.js]. ([Polymer] and [X-Tag], on the
  other hand, are based _entirely_ on draft standard technologies,
  including custom elements.)


## How do they work?
Custom element behaviors are added at runtime (whenever the
"registration" occurs in JavaScript), and can hook into a number of
different element _lifecycle events_:

1. **When an instance of the custom element is created**, either via
   the class constructor or `document.createElement()`; or when
   existing custom elements are registered.
1. **When an instance is attached** (or "connected") to the document,
   either directly or indirectly. This may be called multiple times,
   and is generally the best place to establish event handlers.
1. **When an instance is removed** (or "disconnected") from the
   document, either directly or indirectly. As with
   `attachedCallback()`, this may be called multiple times.
1. **When an attribute is changed**. You can also subscribe to specific
   [observed attributes](#observed-attributes) if you only care about
   attributes unique to your element.

## Browser Support
**Don't let the fact that the [Custom Elements specification][spec] is a
working draft scare you!**

As of this writing, Chrome has already [implemented][Chrome] the [v1 spec],
and both [Safari] and [Firefox] are working on their own implementations of
[v2](#v2). Regardless of which browsers end up implementing which spec
(if any), the v1 spec is "frozen" (meaning it will not change), and
there are several solid [polyfills](#polyfills) that bring custom
element support to just about every browser—including IE8!

That said, when using custom elements—or anything involving JavaScript,
for that matter—**you should always design experiences for progressive
enhancement**, and plan for the possibility that JavaScript isn't
enabled or available.

## The APIs

### v1
The working draft spec was formally "frozen" and dubbed [v1][v1 spec]
in early 2016, and [several browsers][caniuse] have already shipped
native implementations. The v1 API consists of a single function:

```js
document.registerElement('element-name', ElementClass);
```

Where `ElementClass` can either be a first-class constructor that extends
[HTMLElement], or an object with a `prototype` that extends it:

```js
// ES5
var ElementClass = {
  prototoype: Object.create(
    HTMLElement.prototype,
    {
      // descriptors here
    }
  )
};

// ES2015/ES6 
class ElementClass extends HTMLElement {
  // methods and accessors here
}
```

The v1 API observes the following instance (prototype) methods:

1. `createdCallback()` is called when the element is created.
1. `attachedCallback()` is called whenever the element is added to
   the document, either directly or indirectly.
1. `detachedCallback()` is called whenever the element is removed from
   the document.
1. `attributeChangedCallback(attr, oldValue, newValue)` is called when
   an _observed_ attribute (see below) is changed.

and the following static (class) properties:

1. `extends` denotes the native HTML element that your custom element
   will extend via the HTML `is` attribute. See
   [type extension](#type-extension) for more info.
1. `observedAttributes` is an optional array of attribute names for
   which the `attributeChangedCallback()` will be called. If you do not
   provide this property, the callback will fire for all attributes.

  ```js
  // ES5
  var CounterElement = {
    observedAttributes: ['value'],
    prototype: Object.create(
      HTMLElement.prototype,
      {
        attributeChangedCallback: {value: function(name, old, value) {
          // we can safely ignore name here because 'value' is the only
          // observed attribute
          this.value = value;
        }},
        
        value: {
          get: function() {
            return ('_value' in this)
              ? this._value
              : (this._value = 0);
          },
          set: function(value) {
            this._value = value;
          }
        }
      }
    )
  };
  ```

## Type Extension
Custom elements that extend built-in HTML elements with special
semantics or behaviors (such as `<button>` or `<input>`) must use a
feature called _type extension_ via the HTML `is` attribute:

```html
<button is="fancy-button">I am fancy</button>
<!-- this will NOT inherit button's semantics or behavior: -->
<fancy-button>I am not fancy</fancy-button>
```

When using type extension, your custom element class must also include
a static `extends` property (_not_ a prototype property):

```js
// ES5
var FancyButton = {
  extends: 'button',
  prototype: Object.create(
    HTMLButtonElement.prototype, // note the new base class
    {
      // descriptors here
    }
  )
};

// ES2015/ES6
class FancyButton extends HTMLButtonElement {
  static get extends() { return 'button'; }
  // methods and accessors
}
```

And if you want to create an extended element at runtime, you must pass
the custom element name as the second argument to `document.createElement()`:

```js
var fancy = document.createElement('button', 'fancy-button');
```

## v2
Since then, a new version (informally referred to as "v2") of the [spec]
has been developed, and which consists of a slightly different API for
registering custom elements:

```js
// similar to v1's document.registerElement():
customElements.define('element-name', ElementClass);
// except that 'extends' is passed in the options argument
customElements.define('fancy-image', ElementClass, {extends: 'img'});
```

As with the [v1 API](#v1), the `ElementClass` can be either a first-class
constructor that extends the [HTMLElement] base class (or any of its
subclasses), or an object with a `prototype` that extends the base class's.
In the case of [type extension](#type-extension), you must pass an `options`
object with an `extends` property (rather than defining it as a static
property of `ElementClass` in the v1 API):

```js
customElements.define('fancy-button', FancyButton, {extends: 'button'});
```

Similarly, when creating a custom element at runtime, you must pass an
options object with an `is` property:

```js
var custom = document.createElement('button', {is: 'fancy-button'});
```

The v2 API observes the following instance (prototype) methods:

1. `connectedCallback()` is the new name for v1's `attachedCallback()`
1. `disconnectedCallback()` is the equivalent of v1's `detachedCallback()`
1. `attributeChangedCallback(name, oldValue, newValue)` behaves exactly like
   its counterpart in the [v1 spec], including observed attributes.

The v2 API [specifies][CustomElementsRegistry] two additional methods on
the global `customElements` object:

* `customElements.get('element-name')` returns the constructor of the
  provided custom element name, or `undefined`.
* `customElements.whenDefined('element-name')` returns a Promise that
  resolves if/when the named custom element is defined via
  `customElements.define()`.

## Gotchas

### Custom Events
You can listen for and dispatch [custom events][CustomEvent] in custom
elements. The only bummer is that, even though most modern browsers
support the `CustomEvent` constructor, it's missing in all versions of IE
and in older versions of [PhantomJS], which is used for lots of "headless"
integration testing. My advice is to include
[this polyfill](https://www.npmjs.com/package/custom-event), which falls
back on the native implementation. Here's how you could have your component
"announce" its readiness to the rest of the document, for instance:

```js
var CustomEvent = require('custom-event');
document.registerElement('my-element', {
  prototype: Object.create(
    HTMLElement.prototype,
    {
      attachedCallback: {value: function() {
        this.dispatchEvent(new CustomEvent('my-element-ready'));
      }}
    }
  )
});
```
### SVG and Namespaces
Because you need a [polyfill] and namespaces are tricky, it's basically
impossible to reliably extend SVG elements, or any element that requires
an XML namespace. Your best bet is to write a component that wraps `<svg>`
elements or creates them at runtime if they don't exist.

### Class Definition
One of the trickiest things about custom elements is the magical incantation
for defining element classes that extend `HTMLElement` or its subclasses.
There are a couple of ways to do this:

1. Create an object literal (rather than a proper constructor function) with
   a `prototype` that extends `HTMLElement.prototype`. The only way to do this
   in a single expression is to use [`Object.create()`][Object.create], which
   extends the first argument with _descriptors_ in the second. The important
   thing to note here is that because these are property descriptors, methods
   must be provided as objects with a `value` property:

  ```js
  // ES5
  var CustomElement = {
    prototype: Object.create(
      HTMLElement.prototype,
      {
        // this will NOT work:
        createdCallback: function() {
        },
        // but this will:
        createdCallback: {value: function() {
        }},
        
        // accessors look like this:
        someValue: {
          get: function() { /* ... */ },
          set: function(value) { /* ... */ }
        }
      }
    )
  };
  ```
  
  **Note:** if you need to support older browsers such as IE8 or below,
  you will also need a polyfill or shim for ES5 standard APIs, such as
  [aight] or [es5-shim].
  
1. A variation on the above method uses [`Object.create()`][Object.create]
   but assigns methods directly:

  ```js
  // ES5
  var CustomElement = {
    prototype: Object.create(HTMLElement.prototype)
  };
  
  CustomElement.prototype.someMethod = function(arg) { /* ... */ };
  
  // any accessors not passed to Object.create() can be defined like so.
  // note that this is *exactly* what Object.create() is doing under the
  // hood!
  Object.defineProperties(CustomElement.prototype, {
    someValue: {
      get: function() { /* ... */ },
      set: function(value) { /* ... */ }
    }
  });
  ```

1. Use prototypal inheritance.
  * In the [v1 API](#v1) the constructor is essentially ignored, so you don't
    have to call the superclass constructor, and all of your constructor-like
    logic goes in the `createdCallback()` class method:

    ```js
    // ES5
    var CustomElement = function() { /* this is never called */ };
    CustomElement.prototype = Object.create(HTMLElement.prototype, {
      createdCallback: {value: function() {
        console.log("it's alive!");
      }
      // other descriptors here
    });
    ```
    
  * In the [v2 API](#v2) the constructor _is_ called, and there is no
    `createdCallback()`. In ES2015/ES6, this couldn't be simpler:

    ```js
    // ES2016/ES6
    class CustomElement extends HTMLElement {
      constructor() {
        super();
        // created logic goes here
      }
      
      connectedCallback() { /* ... */ }

      someOtherMethod() { /* ... */ }
      
      get someValue() { /* ... */ }
      set someValue(value) { /* ... */ }
    }
    ```
    
    :construction: **I'm honestly not sure how this will be handled
    natively in ES5.** Does the constructor get called, and is the
    superclass constructor essentially ignored?

## Polyfills
There are at least two polyfills that are worth trying out:

1. [document-register-element] is a small, standalone, light-weight (3K gzipped)
   polyfill that has served me well on several projects and offers browser
   support back to IE9 out of the box, and IE8 with some additional scripts.
   **If you're targeting the v1 API, this is a solid choice.**

1. The [WebComponents.js] suite of polyfills includes a Custom Elements
   "shim" was made specifically to support web component libraries built on
   top of web standards, such as [Polymer], [Bosonic], and [X-Tag]. The
   custom elements shim alone is about 5K gzipped, though one of its maintainers
   [boasts](https://github.com/WebReflection/document-register-element/issues/58#issuecomment-226890046)
   that the v2 implementation is only 1.7K gzipped. **If you're targeting the
   v2 API, then [this](https://github.com/webcomponents/webcomponentsjs/tree/v1/src/CustomElements/v1)
   is the one you want.**

## Frameworks
There are a number of tools built on top of the custom elements spec(s)
that handle a lot of the nitty-gritty details of component implementation.
The two big ones are:

* [Polymer] is a Google project with tons of features and a massive,
  ongoing development effort behind it. I have a couple of qualms with it,
  though, namely:

  * It tries to do too much. If you don't need or want two-way data binding,
    or many of the other whiz-bang features that Polymer offers, you're
    still stuck with at least 40K of polyfills _plus_ 120K for Polymer "core".
    Furthermore, you can't just use Polymer to create custom elements; you
    pretty much have to deliver your components as HTML imports. If your
    components are markup-heavy and/or rely on the Shadow DOM, this might
    be a good thing; but if you're not, then your users are paying a hefty
    price for your convenience.
  * Polymer still suggests Bower as its package manager, even though one of
    Bower's maintainers has suggested [not to use it anymore](https://gofore.com/stop-using-bower/).

  Bottom line: Polymer is more of an application development framework than
  a custom element framework.

* [Skate](https://customelements.io/skatejs/skatejs/) "focuses on size,
  performance and is built around a functional rendering pipeline", weighing
  in at just 4K gzipped.

* [X-Tag] is a succinct wrapper around the custom elements v1 API that
  abstracts away a lot of the boring and/or tricky things about component
  development, such as event delegation (listening for events at the
  component level that originate from specific elements) and attribute
  reflection (syncing attribute and property values). X-Tag was originally
  made, then promptly abandoned, by Mozilla; but it's now actively
   maintained by Microsoft.

[spec]: https://www.w3.org/TR/custom-elements/
[v1 spec]: https://www.w3.org/TR/2016/WD-custom-elements-20160226/
[caniuse]: http://caniuse.com/#feat=custom-elements
[HTMLElement]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement
[CustomElementsRegistry]: https://www.w3.org/TR/custom-elements/#custom-elements-api
[Object.create]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
[aight]: https://github.com/shawnbot/aight
[es5-shim]: https://github.com/es-shims/es5-shim
[Polymer]: https://www.polymer-project.org/
[Angular]: https://angularjs.org/
[React]: https://facebook.github.io/react/
[Vue.js]: https://vuejs.org/
[Chrome]: https://www.chromestatus.com/feature/4696261944934400
[Firefox]: https://bugzilla.mozilla.org/show_bug.cgi?id=889230
[Safari]: https://bugs.webkit.org/show_bug.cgi?id=150225
[WebComponents.js]: https://github.com/webcomponents/webcomponentsjs
[document-register-element]: https://github.com/WebReflection/document-register-element
[Bosonic]: https://bosonic.github.io/
[X-Tag]: https://x-tag.github.io/
[PhantomJS]: http://phantomjs.org/
