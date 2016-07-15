# WTF are HTML custom elements?
Custom elements is a W3C _working draft_ [specification][spec] that
provides a mechanism for defining custom behaviors (such as dynamic
content or interactivity) for HTML elements with custom names.

Custom elements are just HTML elements! The only constraint is that
they _must_ contain at least one hyphen (`-`):

```html
<message-element>Hi!</message-element>

<super-section>
  <p>Custom elements can contain content!</p>
  <message-element>And other custom elements!</message-element>
</super-section>

You can also extend built-in elements using the "is" attribute:
<input is="date-picker" type="date">
```

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

As of this writing, Chrome has already implemented the [v1 spec], and
both Safari and Firefox are working on their own implementations of
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
// similar to v1's document.registerElement(), but with options
customElements.define('element-name', ElementClass, options);
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

* `customElements.get('element-name')` returns the constructor 

[spec]: https://www.w3.org/TR/custom-elements/
[v1 spec]: https://www.w3.org/TR/2016/WD-custom-elements-20160226/
[caniuse]: http://caniuse.com/#feat=custom-elements
[HTMLElement]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement
[CustomElementsRegistry]: https://www.w3.org/TR/custom-elements/#custom-elements-api
