# Packaging HTML custom elements

## Exporting the right thing
If you want to give users the ability to register your element
using their own custom element name, then you should package
the class:

```js
class MyCustomElement extends HTMLElement {
}
export default MyCustomElement;
```

Otherwise, you should use the [v1 API](README.md#v1) to register
your class with its known name:

```js
// class MyCustomElement etc.
export default customElements.define('my-element', MyCustomElement);
```

Exporting the defined class constructor (the second example)
ensures that all of the right options are passed to
`customElements.define()`, as in when you're using
[type extension](README.md#type-extension):

```js
class XInput extends HTMLInputElement {
}
export default customElements.define('x-input', XInput, {is: 'input'});
```

## JavaScript syntax
The above examples use the ES2015 [class syntax], which is a much
simpler way to write single-statement class definitions than the
ES5 equivalent. If you distribute your class in ES2015, be sure
to provide an ES5 version for people who aren't using Babel or
another transpiler on their end.

### ES5 syntax
A ES5 custom element class definition typically looks something
like this:

```js
// one-liner
var MyCustomElement = {

  // static/class methods and properties go here
  get observedAttributes() { return ['x', 'y', 'z']; },
  
  prototype: Object.create(
    // the prototype of the class you're extending
    HTMLElement.prototype,
    {
      // instance methods and properties go here
      someMethod: {value: function() {
      }},
      
      someProperty: {
        get: function() { },
        set: function(value) { }
      }
    }
  )
};
```

Alternatively, you can write the more "classic" ES5 prototypal style:

```js
// note: the constructor is always empty!
var MyCustomElement = function() { };

MyCustomElement.prototype = Object.create(HTMLElement.prototype);

MyCustomElement.prototype.someMethod = function() {
};

Object.defineProperties(MyCustomElement.prototype, {
});
```
