# Ember Style Guide

## Table Of Contents

* [General](#general)
* [Computed Properties](#computed-properties)
* [Components](#components)
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

### Use `config` object to check environment

If you're in a situation where you need to check if the code you're running is in testing mode. Example: to prevenet a
timer or do to an early return. Do not import `Ember` just for checking testing environment.

```js
// BAD
import Ember from 'ember';
if (Ember.testing) { ... }
```

```js
// Good
import config from 'app/config/environment';

if (config.environment === 'test') { ... }

// or if multiple conditionals

const testing = config.environment === 'test';
```

### Pods

We use [pods structure](https://ember-cli.com/user-guide/#using-pods)

When using ember-cli to generate files, use `--pod`

```
ember g component my-component --pod
```
## Components

### Do not create components in zapatos

We used to share components by adding it to zapatos, but we're moving away from this practice. Please share components by adding it to `/components`

## Computed Properties

### Use brace expansion

This allows much less redundancy and is easier to read.

Note that **the dependent keys must be together (without space)** for the brace expansion to work.

```js
import { computed } from '@ember/object';

// Good

fullName: computed('user.{firstName,lastName}', {
  // Code
})

// Bad

fullName: computed('user.firstName', 'user.lastName', {
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

Use test selectors for any selectors added for testing purposes.

Example:

```hbs
<button data-test-save onclick={{action "save"}}>
  Save
</button>
```

### Use async/await when writing a test method

This makes tests cleaner and easier to read.

```js
test('page title is awesome', async function(assert) {
  assert.expect(1);

  await visit('/');

  assert.equal(this.element.querySelector('.page-title').textContent, 'Awesome Title!');
});
```

References:

- https://dockyard.com/blog/2018/01/11/modern-ember-testing
- https://dockyard.com/blog/2018/01/18/test-helpers-the-next-generation

### Page Objects

We use [ember-cli-page-object](http://ember-cli-page-object.js.org/).

Example:

```js
import { create } from 'ember-cli-page-object';

export default create({
  visit: visitable('/awesome'),
  title: {
    scope: '[data-test-title]'
  }
```

```js

test('page title is awesome', async function(assert) {
  assert.expect(1);

  await page.visit;

  assert.equal(page.title.text, 'Awesome Title!');
});
```
