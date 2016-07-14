# WTF are HTML custom elements?
Custom elements is a W3C _working draft_ [specification][spec] that
provides a mechanism for defining custom behaviors (such as dynamic
content or interactivity) for HTML elements with custom names.

Custom elements are just HTML elements! The only constraint is that
they _must_ contain at least one hyphen (`-`):

```html
<my-element></my-element>

<date-picker value="2016-06-12"></date-picker>

<super-section>
  <p>Custom elements can contain content!</p>
  <super-button>And other custom elements!</super-button>
</super-section>
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

## The Spec
**Don't let the fact that the [Custom Elements specification][spec] is a
working draft scare you!** As of this writing, Chrome has already
implemented an early version of the spec, and both Safari and Firefox
are working on their own implementations. Regardless of which browsers
end up implementing the spec, the [v1 spec](#v1) is "frozen" (meaning it
will not change), and there are several solid [polyfills](#polyfills)
that bring custom elements v1 support to just about every browser
(including IE8!).

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

The custom elements API observes the following instance (prototype)
methods:

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

## v2
Since then, a new version (informally referred to as "v2") of the [spec]
has been developed, and which consists of a slightly different API:

```js
customElements.define('element-name', ElementClass, options);
```

In both the v1 and v2 APIs, the `ElementClass` can be either a first-class
constructor that extends the [HTMLElement] base class (or any of its
subclasses).


[spec]: https://www.w3.org/TR/custom-elements/
[v1 spec]: https://www.w3.org/TR/2016/WD-custom-elements-20160226/
[caniuse]: http://caniuse.com/#feat=custom-elements
[HTMLElement]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement
