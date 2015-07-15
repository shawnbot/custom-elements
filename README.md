# HTML Custom Elements

## What Are They?
Custom elements are **custom HTML tags** for which you can define behaviors in JavaScript, like so:

```js
document.registerElement('custom-element', {
  prototype: {
    // your prototype definition here
  }
});
```

Then, when can use the following HTML in your document, the DOM element will inherit the protoype of your custom element definition:

```html
<custom-element>
</custom-element>
```

If you've got time, read [the draft spec][spec]. Otherwise, read on for some basic usage examples and gotchas.

## [Can I Use Them?](http://caniuse.com/#feat=custom-elements)
If you're using Chrome 36+, **yes**! You can *test* them natively. If you're delivering web software for public use, though, you're going to need [this awesome polyfill](https://github.com/WebReflection/document-register-element) for the [`document.registerElement()`][document.registerElement] API. It even works in IE8 with some additional dependencies!

## Usage

### Custom Element Names
Custom element name **must** contain a `-` (hyphen). There are some [reserved names](http://www.w3.org/TR/custom-elements/#concepts), but they're pretty obtuse. Generic custom elements are sometimes prefixed with `x-`, as popularized by the [X-Tag](http://x-tags.org/) project.

### Base Class
Custom elements should include a "base class" that provides the standard HTML Element interface. Unless you're looking to extend a specific element's API (such as `<img>`), you can extend `HTMLElement.prototype`:

```js
document.registerElement('custom-element', {
  prototype: Object.create(
    HTMLElement.prototype,
    {
      // custom prototype
    }
  )
});
```

:warning: **If you're looking to extend an SVG element, you'll *probably* want to use [type extension](#type-extensions).**

### Type Extensions
In addition to specifying behavior for a custom element *name*, you can also create a [type extension](http://www.w3.org/TR/custom-elements/#dfn-type-extension) that adds behaviors to an existing HTML element with its own [semantics](http://www.w3.org/TR/custom-elements/#semantics) using the `is` attribute:

```html
<dl is="x-glossary">
  <dt>Foo</dt>
  <dd>The definition of "foo".</dd>
</dl>
```

```js
document.registerElement('x-glossary', {
  'extends': 'dl',
  prototype: Object.create(
    HTMLDListElement.prototype,
    {
      // fancy prototype here
    }
  )
});
```

### Prototype Descriptors
[The draft spec][spec] doesn't mention it specifically, but all of the property descriptors in your custom element `prototype` **must** be provided as objects in the form accepted by [`Object.defineProperty()`][Object.defineProperty]. This means that:

1. Method descriptors must be wrapped in an object with a `value` property:

  ```js
  document.registerElement('foo-bar', {
    prototype: Object.create(
      HTMLElement.prototype,
      {
        // element.doSomething()
        doSomething: {value: function() {
        }}
      }
    )
  });
  ```
  
2. Property descriptors should be objects with `get` and (optionally) `set` property.

  ```js
  document.registerElement('foo-bar', {
    prototype: Object.create(
      HTMLElement.prototype,
      {
        // element.baz
        baz: {
          get: function() {
            // get the value
          },
          set: function(value) {
            // set the value
          }
        }
      }
    )
  });
  ```

3. Any descriptor can also specify the `configurable` and `enumerable` properties.

### Lifecycle Callbacks
Custom elements support four "lifecycle" callback functions that you register by adding them to the element's `prototype` definition:

#### Created: `createdCallback()`
This function is called when the custom element in instantiated, either when an element already on the page is registered or when one is created programmatically:

```js
var element = document.createElement('custom-element');
// or, with type extension:
var elemeent = document.createElement('dl', 'x-glossary');
```

#### Attached: `attachedCallback()`
This function is called when the element is attached to the document via `parent.appendChild()`.

#### Attribute changed: `attributeChangedCallback(name, previousValue, newValue)`
This function is called whenever an attribute of the element is changed.

#### Detached: `detachedCallback()`
This function is called when the element is removed from the document via `parent.removeChild()`.

### Events
You can trigger custom events from within your custom elements using the [W3C DOM4 Events][dom4-events] API. For instance, you could define a `content` accessor on your custom element that, when changed, dispatches a `change` event:

```js
document.registerElement('content-thing', {
  prototype: Object.create(
    HTMLElement.prototype,
    {
      content: {
        get: function() { return this.textContent; },
        set: function(content) {
          if (content != this.content) {
            this.textContent = content || '';
            this.dispatchEvent(new CustomEvent('change'));
          }
        }
      }
    }
  )
});
```

[spec]: http://www.w3.org/TR/custom-elements/
[document.registerElement]: https://developer.mozilla.org/en-US/docs/Web/API/Document/registerElement
[Object.defineProperty]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
[dom4-events]: http://www.w3.org/TR/2015/WD-dom-20150618/#events

