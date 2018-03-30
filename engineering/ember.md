# Ember Style Guide

## Table Of Contents

* [Computed Properties](#computed-properties)
* [Templates](#templates)
* [Tests](#tests)

## General

### Use `get` and `set`

Calling `someObj.get('prop')` couples your code to the fact that
`someObj` is an Ember Object. It prevents you from passing in a
POJO, which is sometimes preferable in testing. It also yields a more
informative error when called with `null` or `undefined`.

Although when defining a method in a controller, component, etc. you
can be fairly sure `this` is an Ember Object, for consistency with the
above, we still use `get`/`set`.

```js
// Good
import { get, set } from '@ember/object';

set(this, 'isSelected', true);
get(this, 'isSelected');

// Bad

this.set('isSelected', true);
this.get('isSelected');
```

## Computed Properties

### Use brace expansion

This allows much less redundancy and is easier to read.

Note that **the dependent keys must be together (without space)** for the brace expansion to work.

```js
// Good
fullName: Ember.computed('user.{firstName,lastName}', {
  // Code
})

// Bad
fullName: Ember.computed('user.firstName', 'user.lastName', {
  // Code
})
```

## Templates

### Don't use partials

Use components instead of partials.

Partials have access to properties in their parent template's scope while
components require any external properties used to be explicitly passed in. This
prevents difficult to diagnose bugs, provides more confidence when
refactoring. Components are also easier to test.

## Tests

### [ember-test-selectors](https://github.com/simplabs/ember-test-selectors)

Import using the following syntax.

```js
import testSelector from 'ember-test-selectors';
```
