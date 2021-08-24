---
title: Interface Types
slug: /types/interfaces
---

Classes in Flow are nominally typed. This means that when you have two separate
classes you cannot use one in place of the other even when they have the same
exact properties and methods.

```js flow-check
// @flow
class Foo {
  serialize() { return '[Foo]'; }
}

class Bar {
  serialize() { return '[Bar]'; }
}

// $ExpectError
const foo: Foo = new Bar(); // Error!
```

Instead, you can use `interface` in order to declare the structure of the class
that you are expecting.

```js flow-check
// @flow
interface Serializable {
  serialize(): string;
}

class Foo {
  serialize() { return '[Foo]'; }
}

class Bar {
  serialize() { return '[Bar]'; }
}

const foo: Serializable = new Foo(); // Works!
const bar: Serializable = new Bar(); // Works!
```

You can also declare an anonymous interface:

```js flow-check
// @flow
class Foo {
  a : number
}

(new Foo() : interface { a : number });
```

You can also use `implements` to tell Flow that you want the class to match an
interface. This prevents you from making incompatible changes when editing the
class.

```js flow-check
// @flow
interface Serializable {
  serialize(): string;
}

class Foo implements Serializable {
  serialize() { return '[Foo]'; } // Works!
}

class Bar implements Serializable {
  // $ExpectError
  serialize() { return 42; } // Error!
}
```

You can also use `implements` with multiple interfaces.

```js flow-check
class Foo implements Bar, Baz {
  // ...
}
```

## Interface Syntax {#toc-interface-syntax}

Interfaces are created using the keyword `interface` followed by its name and
a block which contains the body of the type definition.

```js flow-check
interface MyInterface {
  // ...
}
```

The syntax of the block matches the syntax of object types and has all of the
same features.

### Interface Methods {#toc-interface-methods}

You can add methods to interfaces following the same syntax as class methods. Any `this` parameters you
provide are also subject to the same restrictions as class methods.

```js
interface MyInterface {
  method(value: string): number;
}
```

Also like [class methods](./classes#toc-class-methods), interface methods must also remain bound to the interface on which they were defined.

### Interface Properties {#toc-interface-properties}

You can add properties to interfaces following the same syntax as class
properties.

```js
interface MyInterface {
  property: string;
}
```

Interface properties can be optional as well.

```js
interface MyInterface {
  property?: string;
}
```

### Interfaces as maps {#toc-interfaces-as-maps}

You can create ["indexer properties"](./objects#toc-objects-as-maps) the same
way as with objects.

```js
interface MyInterface {
  [key: string]: number;
}
```

### Interface Generics {#toc-interface-generics}

Interfaces can also have their own [generics](./generics/).

```js
interface MyInterface<A, B, C> {
  property: A;
  method(val: B): C;
}
```

Interface generics are [parameterized](./generics#toc-parameterized-generics).
When you use an interface you need to pass parameters for each of its generics.

```js
// @flow
interface MyInterface<A, B, C> {
  foo: A;
  bar: B;
  baz: C;
}

var val: MyInterface<number, boolean, string> = {
  foo: 1,
  bar: true,
  baz: 'three',
};
```

<!-- [TODO: Overloading interface methods -->

## Interface property variance (read-only and write-only) {#toc-interface-property-variance-read-only-and-write-only}

Interface properties are [invariant](../lang/variance/) by default. But you
can add modifiers to make them covariant (read-only) or contravariant
(write-only).

```js
interface MyInterface {
  +covariant: number;     // read-only
  -contravariant: number; // write-only
}
```

#### Covariant (read-only) properties on interfaces {#toc-covariant-read-only-properties-on-interfaces}

You can make a property covariant by adding a plus symbol `+` in front of the
property name.

```js
interface MyInterface {
  +readOnly: number | string;
}
```

This allows you to pass a more specific type in place of that property.

```js flow-check
// @flow
interface Invariant {  property: number | string }
interface Covariant { +readOnly: number | string }

var x : { property : number } = { property : 42 };
var y : { readOnly : number } = { readOnly : 42 };

var value1: Invariant = x; // Error!
var value2: Covariant = y; // Works
```

Because of how covariance works, covariant properties also become read-only
when used. Which can be useful over normal properties.

```js flow-check
// @flow
interface Invariant {  property: number | string }
interface Covariant { +readOnly: number | string }

function method1(value: Invariant) {
  value.property;        // Works!
  value.property = 3.14; // Works!
}

function method2(value: Covariant) {
  value.readOnly;        // Works!
  // $ExpectError
  value.readOnly = 3.14; // Error!
}
```

#### Contravariant (write-only) properties on interfaces {#toc-contravariant-write-only-properties-on-interfaces}

You can make a property contravariant by adding a minus symbol - in front of
the property name.

```js flow-check
interface InterfaceName {
  -writeOnly: number;
}
```

This allows you to pass a less specific type in place of that property.

```js flow-check
// @flow
interface Invariant     {  property: number }
interface Contravariant { -writeOnly: number }

var numberOrString = Math.random() > 0.5 ? 42 : 'forty-two';

// $ExpectError
var value1: Invariant     = { property: numberOrString };  // Error!
var value2: Contravariant = { writeOnly: numberOrString }; // Works!
```

Because of how contravariance works, contravariant properties also become
write-only when used. Which can be useful over normal properties.

```js flow-check
interface Invariant     {   property: number }
interface Contravariant { -writeOnly: number }

function method1(value: Invariant) {
  value.property;        // Works!
  value.property = 3.14; // Works!
}

function method2(value: Contravariant) {
  // $ExpectError
  value.writeOnly;        // Error!
  value.writeOnly = 3.14; // Works!
}
```