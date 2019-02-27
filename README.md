# All about HTML Custom Elements
Custom Elements is a [WHATWG] HTML [specification][spec] that
provides a mechanism for defining new behaviors (such as dynamic
content or interactivity) for HTML elements with custom names.
Custom elements are just HTML elements, with all of the methods and
properties of other, built-in elements. The only real constraint
is that
**[custom element names must contain a hyphen (`-`)](https://html.spec.whatwg.org/multipage/custom-elements.html#valid-custom-element-name)**.

```html
<message-element>Hi!</message-element>

<super-section>
  <p>Custom elements can contain content!</p>
  <message-element>And other custom elements!</message-element>
</super-section>
```

**Note:** The adopted [custom element spec][spec], formerly known
as "v1", differs almost entirely from the original ["v0" spec][v0 spec].
If you've been using `document.registerElement()` from the v0 API,
then read on to see what's changed.

## Table of Contents
1. [Why custom elements?](#why-custom-elements)
1. [How do they work?](#how-do-they-work)
1. [Browser support](#browser-support)
1. [Customized built-in elements](#customized-built-in-elements)
1. [Observed attributes](#observed-attributes)
1. [Polyfills](#polyfills)
1. [Further Reading](#further-reading)

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
    [v0 spec]) and after (`:defined` in the [v1 spec][spec]) they've
    been registered via JavaScript.

* **The DOM _is_ the API.** Components built with tools like jQuery
  can be cumbersome to build, modify, and maintain because they often
  introduce another layer of abstraction (such as the `jQuery` object
  and its API) on top of the DOM. And while there's nothing to stop
  one from building custom elements that use jQuery, D3, React, or
  whatever under the hood, I've found custom elements made with
  vanilla JS to be easier to grok and read.

  Put another way: **The DOM isn't going away any time soon**, and
  custom elements provide a solid conceptual _and_ technical
  foundation on which all sorts of amazing things can be built.

* **Web Standards.** Custom elements are an adopted [WHATWG] HTML
  [specification][spec]. That means that—in theory, at least—they
  will _eventually_ be implemented natively in all modern browsers
  with the same API.

  Keep in mind is that **custom elements are not mutually exclusive
  of other web component technologies**. In fact, I see them as a
  powerful force multiplier of any technology that leverages them:
  When designed well, custom elements put their power into the hands
  of anyone who can write HTML. Great React components, for instance,
  can be made even greater by packaging them up as custom elements.

## How do they work?
Custom element behaviors are added at runtime (whenever the
"registration" occurs in JavaScript), and can hook into a number of
different element _lifecycle events_:

1. **When an instance of the custom element is created**, either via
   the class constructor or `document.createElement()`; or when
   existing custom elements are registered.
1. **When an instance is "connected"** to the document,
   either directly or indirectly. This may be called multiple times,
   and is generally the best place to add event handlers.
1. **When an instance is removed** (or "disconnected") from the
   document, either directly or indirectly. This may also be called
   multiple times.
1. **When an attribute is changed**. You can also subscribe to specific
   [observed attributes](#observed-attributes) if you only care about
   attributes unique to your element, or to implement [attribute reflection].

## Browser Support
As of summer 2018:

* [Chrome has implemented][chrome] both the [v0 spec] and the adopted ["v1" spec][spec].
* [Safari has implemented][safari] so-called "autonomous" custom elements, but has
  declined to support extension of built-in elements.
* [Firefox has implemented][firefox] the spec behind a feature flag.

Regardless of your support targets, you should use a [polyfill](#polyfills).

⚠️ When using custom elements—or anything involving JavaScript,
for that matter—**always design experiences for progressive enhancement**,
and plan for the possibility that JavaScript isn't enabled or available.

## The API
The custom elements API consists mainly of a [CustomElementRegistry] object
that can be used to register class constructors for custom elements by name:

```js
window.customElements.define('element-name', ElementClass);
```

Where `ElementClass` is a class that extends [HTMLElement]:

```js
// ES2015
class ElementClass extends HTMLElement {
  constructor() {
    super() // <-- this is required!
    this.created()
  }
  
  created() {
    console.log('hi!', this)
  }
}
```

Custom element classes may implement any of the following lifecycle
(instance) methods:

1. `connectedCallback()` is called whenever the element is added to
   the document, either directly (`document.body.appendChild(el)`) or indirectly
   (as part of a fragment or a DOM tree that's added to the document).
1. `disconnectedCallback()` is called whenever the element is removed from
   the document.
1. `attributeChangedCallback(attr, oldValue, newValue)` is called when
   an _observed_ attribute (see below) is changed.

and the following static (class) properties:

1. `observedAttributes` is an optional array of attribute names for
   which the `attributeChangedCallback()` will be called. If you do not
   provide this property, the callback will fire for all attributes.

  ```js
  // ES5
  class CounterElement extends HTMLElement {
    static get observedAttributes() { return ['value'] }

    attributeChangedCallback(name, old, value) {
      // we can safely ignore name here because 'value' is the only
      // observed attribute
      this.value = value
    }
        
    get value() {
      return ('_value' in this)
        ? this._value
        : (this._value = 0)
    }
    
    set value(value) {
      this._value = value
    }
  }
  ```

## Customized built-in elements

⚠️ **Warning:** Safari does not _yet_ implement [this portion of the
spec][customized built-ins]. If you wish to use it, you will need a [polyfill](#polyfills).

Custom elements may extend built-in HTML elements with special
semantics or behaviors (such as `<button>` or `<input>`). Here's how
they work:

1. Register the element with an additional argument indicating which
   element name it extends:
   
    ```js
    window.customElements.define('fancy-button', FancyButton, {
      extends: 'button'
    })
    ```

1. Instantiate the element in HTML with the `is` attribute of the
   extended built-in set to the name of the custom element:
   
    ```html
    <!-- this: -->
    <button is="fancy-button">I am fancy</button>

    <!-- NOT this: -->
    <fancy-button>I am not fancy</fancy-button>
    ```
  
1. Instantiate the element in JavaScript by passing an additional
   argument to `document.createElement()` with the `is` property
   set to the name of the custom element:
   
    ```js
    var fancy = document.createElement('button', {is: 'fancy-button'})
    ```


## Observed Attributes
Observed attributes are attributes that fire the `attributeChangedCallback()`
lifecycle method. If the custom element class has a (static)
`observedAttributes` array, the callback will fire for _only_ the listed
attributes:

```js
// ES2015
class CustomElement extends HTMLElement {
  static get observedAttributes() { return ['foo', 'bar']; }
  
  attributeChangedCallback(attr, old, value) {
    // `attr` will only ever equal 'foo' or 'bar'
    switch (attr) {
      case 'foo':
        break
      case 'bar':
        break
    }
  }
}
```

Otherwise, the callback will be fired for _all_ attributes.

```js
// ES2015
class CustomElement extends HTMLElement {
  attributeChangedCallback(attr, old, value) {
    // `attr` could be anything
  }
}
```

📝 When implementing [attribute reflection], please observe the
[W3C API Design Principles](https://w3ctag.github.io/design-principles/#api-surface).

## The Custom elements registry
The [CustomElementRegistry] object available at `window.customElements`
has two additional methods for querying its state and responding to when
specific custom elements are registered:

* `customElements.get('element-name')` returns the class constructor
  of the provided custom element name (or `undefined` if it hasn't been
  defined).

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
Because you need a [polyfill](#polyfills) and namespaces are tricky, it's
basically impossible to reliably extend SVG elements, or any element that
requires an XML namespace. Your best bet is to write a component that wraps
`<svg>` elements or creates them at runtime if they don't exist.

### Class Definition
One of the trickiest things about custom elements is the magical incantation
for defining element classes that extend `HTMLElement` or its subclasses,
especially in "legacy" ES5 environments that don't support the `class` keyword
or `super()` calls. There are a couple of ways to pull it off:

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

1. Use [Babel] and the [custom-element-classes transform](https://github.com/github/babel-plugin-transform-custom-element-classes#readme). Your `.babelrc` should look something like this:

    ```json
    {
      "presets": ["env"],
      "plugins": [
        "transform-custom-element-classes"
      ]
    }
    ```
    
    which should make it possible to write classes like:
    
    ```js
    class Widget extends HTMLElement {
      constructor() {
        super()
      }
    }
    
    window.customElements.define('widget-element', Widget)
    ```

## Polyfills
The semi-official [webcomponents/custom-elements] polyfill is what GitHub uses,
and it provides a bunch of workarounds for the spec rules involving class
constructors and the `new` keyword. You should use it, too!

**Please note:**
- that extended built-in custom elements aren't supported,
- [known limitations](https://github.com/webcomponents/custom-elements#known-bugs-and-limitations)
- and [you may encounter slowness](https://github.com/webcomponents/custom-elements#customelementspolyfillwrapflushcallback).

The most bullet proof and battle tested polyfills I found are provided by @WebReflection:
- [document-register-element](https://github.com/WebReflection/document-register-element) (supports V1 despites it's name) has [fantastic browser support](https://github.com/WebReflection/document-register-element#tested-on) and a [public test page](http://webreflection.github.io/document-register-element/test/), please note the [`constructor` caveat](https://github.com/WebReflection/document-register-element#v1-caveat).
- [built-in-element](https://github.com/ungap/custom-elements-builtin) again fantastic browser support and [live test page](https://ungap.github.io/custom-elements-builtin/test/es5/), please note the [`constructor` caveat](https://github.com/ungap/custom-elements-builtin#constructor-caveat).

## Further reading
* [Custom Elements v1: Reusable Web Components (Google)](https://developers.google.com/web/fundamentals/web-components/customelements) is a great introduction to custom elements.
* [Introducing Custom Elements (WebKit)](https://webkit.org/blog/7027/introducing-custom-elements/) contains some nice implementation tips. 
* [Using Custom Elements (MDN)](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements) is, like pretty much everything else on MDN, a solid reference.
* [Custom Elements Everywhere](https://custom-elements-everywhere.com/) rates popular web frameworks for compatibility with custom elements.
* The [WebComponents.org introduction](https://www.webcomponents.org/introduction) explains how custom elements fit into the broader landscape of native web components and complement technologies like the Shadow DOM and `<template>` elements.

[spec]: https://html.spec.whatwg.org/multipage/custom-elements.html#custom-element
[old spec]: https://www.w3.org/TR/custom-elements/
[v0 spec]: https://www.w3.org/TR/2016/WD-custom-elements-20160226/
[caniuse]: https://caniuse.com/#feat=custom-elementsv1
[HTMLElement]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement
[CustomElementRegistry]: https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry
[Object.create]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
[aight]: https://github.com/shawnbot/aight
[es5-shim]: https://github.com/es-shims/es5-shim
[Chrome]: https://www.chromestatus.com/feature/4696261944934400
[Firefox]: https://bugzilla.mozilla.org/show_bug.cgi?id=889230
[Safari]: https://bugs.webkit.org/show_bug.cgi?id=150225
[PhantomJS]: http://phantomjs.org/
[CustomEvent]: https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent
[attribute reflection]: https://github.com/domenic/webidl-html-reflector
[event delegation]: https://davidwalsh.name/event-delegate
[custom element registry]: https://html.spec.whatwg.org/multipage/custom-elements.html#customized-built-in-element
[WHATWG]: https://whatwg.org/faq#what-is-the-whatwg
[customized built-ins]: https://html.spec.whatwg.org/multipage/custom-elements.html#customized-built-in-element
[Babel]: http://babeljs.io/
[webcomponents/custom-elements]: https://github.com/webcomponents/custom-elements#readme
