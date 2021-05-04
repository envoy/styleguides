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

``` cli
ember g component my-component --pod
```

## Components

### Performance Considerations

- Consider component loading states. 

A table component might be passed an intial set of data that's resolved when the route loads, but what happens when the user clicks a button to refetch that table data, or maybe filter the data. Without a loading state, this update to the page could seem jarring.

Options for a dropdown menu component might depend on data fetched remotely or from a large set of data already in memory that's expensive to iterate through. We should consider what the user is looking at while this data loads, hopefully not a big white empty space. This specifically is a good place for a skelton view which directs the eye to where the data will be when its fully resolved. 

### Do not create components in zapatos

We used to share components by adding it to zapatos, but we're moving away from this practice. Please share components by adding it to `/components`

## Computed Properties

### Avoid explicit getters

This style has less code and is easier to understand because it's in line with what's in the Ember docs.

An exception to this rule is if you need to define custom `set()` behavior.

```js
import { computed } from '@ember/object';

// Bad

fullName: computed('firstName', 'lastName', {
  get() {
    return [this.firstName, this.lastName].join(' ');
  }
})

// Good

fullName: computed('firstName', 'lastName', function() {
  return [this.firstName, this.lastName].join(' ');
})

// Good - if you need to define set()

fullName: computed('firstName', 'lastName', {
  get() {
    return [this.firstName, this.lastName].join(' ');
  },
  set(key, value) {
    let [firstName, lastName] = value.split(' ');

    this.set('firstName', firstName);
    this.set('lastName', lastName);

    return value;
  }
})

```

## Routes

A quote from the official [ember guides](https://guides.emberjs.com/release/routing/loading-and-error-substates/):

> If you navigate to slow-model, in the model hook using Ember Data, the query may take a long time to complete. During this time, your UI isn't really giving you any feedback as to what's happening. If you're entering this route after a full page refresh, your UI will be entirely blank, as you have not actually finished fully entering any route and haven't yet displayed any templates. If you're navigating to slow-model from another route, you'll continue to see the templates from the previous route until the model finish loading, and then, boom, suddenly all the templates for slow-model load.

### Performance Considerations

- Make use of route loading state templates. This loading substate will be entered immediately without transitioning away from our previous route and before the new route is rightfully entered.
- Don't return an RSVP.hash() from a route model hook unless making use of ember's route loading states. Consider that each promise returned in the hash has potential to delay first render. 
potential to delay first render. 
- For a model hook that returns numberous unrelated queries, consider returning an object with each query assigned to a key such as `{a: this.store.query('a), b: this.store.query('b)}` ... then in your templates you can use the promise state to decide whether that data is fully fetched, ie ... `{{#if (is-pending this.a)}} ... {{/if}}`.
- We're not just responsible for own route hooks, parent route hooks might also take a while to resolve. From a cold page refresh the time it takes for every route in the current heirachy to resolve is how long we're waiting before first render. Without loading states the user can be left in the dust wondering why the page is unresponsive after a click. We've seen customers suffer load times as long as 30 seconds, where 10 seconds of that request was from a parent route.
- The router pauses a route transition until the promises returned from all 3 route hooks are fulfilled.

## Templates

### {{#each}} - Specifying Keys

To improve rendering speed, the `{{#each}}` helper favors reusing it's inner block. Behind the scenes Ember compares the identity of each object in the array. Where there is a difference that inner inner dom will be torn down and rerendered. By passing a `key` param we inform `{{#each}}` that it should only tear down its inner dom when a specific property has changed. This is important because componments with state will lose their state when torn down and rerenderd. Specifying a key is one way to make sure this doesn't happen on accident. [more info](https://api.emberjs.com/ember/3.20/classes/Ember.Templates.helpers/methods/each?anchor=each)...

```hbs
{{#each array key="<someProp>|@index|@identity" |item|}}
```

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

### DO NOT USE Page Objects

~~We use~~ [ember-cli-page-object](http://ember-cli-page-object.js.org/).

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

### Sparse data queries

A 300k/150ms payload for fetching 1000 complete ember data models into the store might seem acceptable, but imagine down the road we require other associated data be included in that response. Our payload just went to 3meg/15seconds.

Do we really need 1000 records in the store in the first place? What if we don't have to worry about the ember data store? What if we can be super specific about the record data we need right down to its attributes or its associated model's attributes? This is the usecase for fetching sparse data. 

Ember data does not yet support sparse data models (or model fragments) and assumes each model is a complete representation of its data. So for now, in order to benefit from sparse data queries we must bypass the the ember data store and consume these json api doc responses directly. 

There are 2 helpers in the codebase to support this approach:

1. A [queryTransient](https://github.com/envoy/garaje/blob/master/app/services/store.js#L9-L18) method on our store service which behaves nearly the same as store.query. 
2. And a [json api response formater](https://github.com/envoy/garaje/blob/master/app/utils/json-api-data-formatter.js) that queryTransient uses internally to normalize the payload with its included associations. 

An example usecase:

``` js
const flowObjs = this.store.queryTransient('flow', {
   include: 'location',
   fields: {
      flows: 'name,type,enabled,location,position',
      locations: 'name,disabled',
   },
})
```

This will return an ArrayProxy that quacks a lot like a DSAdapterPopulatedRecordArray. Then in your templates you can do something like this.

``` hbs
  {{#each flowsObjs as |flowObj|}}
    {{flowObj.id}}
    {{flowObj.type}}
    {{flowObj.data.name}}
    {{flowObj.data.location.id}}
    {{flowObj.data.location.type}}
    {{flowObj.data.location.data.name}}
  {{/each}}
```
