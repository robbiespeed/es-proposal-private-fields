# Encapsulation with WeakMaps
[Modified example](https://github.com/tc39/proposal-private-fields/blob/master/FAQ.md#how-can-you-model-encapsulation-using-weakmaps) of how WeakMaps could be used to get near identical functionaility.
```
function createPrivateMapAccessor () {
  const internalMap = new WeakMap();
  return (publicObj) => {
    if (!internalMap.has(publicObj)) {
      const privateProxy = new Proxy({}, {
        set (privateObj, property, value) {
          if (typeof value === 'function') {
            privateObj[property] = function () {
              return this === privateProxy ?
                value.apply(publicObj, arguments) :
                value.apply(this, arguments);
            };
          }
          else {
            privateObj[property] = value;
          }
          return true;
        }
      });
      internalMap.set(publicObj, privateProxy);
    }
    return internalMap.get(publicObj);
  }
}

const Person = (function(){
  const _private = createPrivateMapAccessor();
  let ids = 0;
  return class Person {
    constructor(name, makeGreeting) {
      this.name = name;
      const privateThis = _private(this);
      privateThis.id = ids++;
      privateThis.makeGreeting = makeGreeting;
    }
    equals(otherPerson) {
      return _private(this).id === _private(otherPerson).id;
    }
    greet(otherPerson) {
      // console.log(this, _private(this));
      return _private(this).makeGreeting(otherPerson.name);
    }
  }
})();
let alice = new Person('Alice', name => `Hello, ${name}!`);
let bob = new Person('Bob', name => `Hi, ${name}.`);
console.log(alice.equals(bob)); // false
console.log(alice.greet(bob)); // === 'Hello, Bob!'

let mallory = new Person('Mallory', function(name) {
  this.id = 0;
  return `o/ ${name}`;
});
console.log(mallory.greet(bob)); // === 'o/ Bob'
console.log(mallory.equals(alice)); // false
```
