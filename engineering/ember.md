# Ember Style Guide

## Table Of Contents

* [Computed Properties](#computed-properties)
* [Tests](#tests)

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

## Tests

### [ember-test-selectors](https://github.com/simplabs/ember-test-selectors)

Import using the following syntax.

```js
import testSelector from 'ember-test-selectors';
```
