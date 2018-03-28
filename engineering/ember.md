# Ember Style Guide

## Table Of Contents

* [Computed Properties](#computed-properties)
* [Templates](#templates)
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

## Templates

### Don't use partials

Use components instead of partials.

Partials have access to properties in their parent template's scope while
components require any external properties used to be explicitly passed in. This
prevents difficult to diagnose bugs, provides more confidence when
refactoring. Components are also easier to test.
