# ECMAScript Private Fields Counter Proposal
_This is a counter proposal to [Private Fields](https://github.com/tc39/proposal-private-fields), which is included in [Class Fields Proposal (Stage 3)](https://github.com/tc39/proposal-class-fields)._

The goal is to provide a easily understandable, and human parsable implementation of private fields.

## Private Keyword
The `private` keyword can be used within a class to access encapsulated private data specific to the class.

[_`private` is a reserved keyword_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Future_reserved_keywords)

### Basic Syntax
```
class Animal {
  constructor (noise) {
    private(this).noise = noise;
  }
  speak () {
    return private(this).noise;
  }
}
```
### Shorthand Syntax
```
class Animal {
  constructor (noise) {
    private.noise = noise;
  }
  speak () {
    return private.noise;
  }
}
```

Both `private` and `private(this)` are accessing the same encapsolated private object.

### Advanced Usage
Using the `private()` syntax, during a method call on one instance we can access the private data of another.
```
class Animal {
  constructor (noise) {
    private.noise = noise;
  }
  speakTogether (otherAnimal) {
    return `${private.noise} ${private(otherAnimal).noise}`;
  }
}
```

In most ways `private` acts like get method for a class specific WeakMap. When adding private methods however the `this` will refrence the public instance.
```
class Person {
  constructor (name, greet) {
    this.name = name;
    private.greet = greet;
  }
  greetOther (otherPerson) {
    return `${private.greet()}\n${private(otherPerson).greet()}`;
  }
}
const jess = new Person('Jess',
  function () {
    return `Hey, I'm ${this.name}`;
  }
);
const tom = new Person('Jess',
  function () {
    return `Hi, my name is ${this.name}`;
  }
);
```

### Class Fields Usage
```
class Item {
  static private id = 0;
  private price = 10;
  constructor (name) {
    this.name = name;
    private.id = private(Item).id++;
  }
  isPriceEqual (other) {
    return private.equals(other) ||
      private.price === private(other).price;
  }
  priceChange (amount) {
    private.price += amount;
  }
  private equals (other) {
    return private(Item).compareId(this, other);
  }
  static private compareId (a, b) {
    return private(a).id === private(b).id;
  }
}
```

