---
id: sanitization-chain-api
title: Sanitization Chain API
---

The sanitization chain is a middleware, and it _should_ be passed to an Express route handler.

You can add as many sanitizers to a chain as you need.
When the middleware runs, it will modify each field in place, applying each of the sanitizers in the order they were specified:

```js
const { body } = require('express-validator');
app.get('/', body('trimMe').trim(), (req, res, next) => {
  // If req.body.trimMe was originally "  something ",
  // its sanitized value will be "something"
  console.log(req.body.trimMe);
});
```

## Standard sanitizers

All sanitizers listed by validator.js are made available within a Sanitization Chain,
and are called "standard sanitizers" in express-validator.

This means you can use any of those methods, e.g. `normalizeEmail`, `trim`, `toInt`, etc.

> **For a complete list of standard sanitizers and their options**,
> please check [validator.js' docs](https://github.com/validatorjs/validator.js#sanitizers).

> **Note:** Since validator.js only accepts `string` as input, any value (including arrays and objects) that needs to be sanitized by a Standard Sanitizer [is first converted to such type](faq.md#why-arrays-are-not-validatedsanitized-correctly).

## Additional methods

In addition to the standard sanitizers, the following methods are also available within a Sanitization Chain:

### `.customSanitizer(sanitizer)`

- `sanitizer(value, { req, location, path })`: the custom sanitizer function.
  Receives the value of the field being sanitized, as well as the express request, the location and the field path.

> _Returns:_ the current sanitization chain instance

Adds a custom sanitizer to the current sanitization chain. It must synchronously return the new value.

Example:

```js
const { param } = require('express-validator');
app.get(
  '/object/:id',
  param('id').customSanitizer((value, { req }) => {
    return req.query.type === 'user' ? ObjectId(value) : Number(value);
  }),
  objectHandler,
);
```

### `.default(default_value)`

> _Returns:_ the current sanitization chain instance

Replaces the current value with a default one if the current value is included in `['', null, undefined, NaN]`.

```js
app.post('/', body('username').default('foo'), (req, res, next) => {
  // 'bar'     => 'bar'
  // ''        => 'foo'
  // undefined => 'foo'
  // null      => 'foo'
  // NaN       => 'foo'
});
```

### `.replace(values_to_replace, new_value)`

> _Returns:_ the current sanitization chain instance

Replaces the current value with a new one if the current value is included in a given Array.

```js
app.post('/', body('username').replace(['bar', 'BAR'], 'foo'), (req, res, next) => {
  // 'bar_' => 'bar_'
  // 'bar'  => 'foo'
  // 'BAR'  => 'foo'
  console.log(req.body.username);
});
```

### `.run(req)`

> _Returns:_ a promise that resolves when the sanitization chain ran.

Runs the current sanitization chain in an imperative way.

```js
const { check } = require('express-validator');
app.post('/create-post', async (req, res, next) => {
  // BEFORE:
  // req.body.content = ' hey your forum is amazing! <script>runEvilFunction();</script>    ';
  await check('content').escape().trim().run(req);

  // AFTER:
  // req.body.content = 'hey your forum is amazing! &lt;script&gt;runEvilFunction();&lt;/script&gt;';
});
```

### `.toArray()`

> _Returns:_ the current sanitization chain instance

Converts the value to an array. `undefined` will result in an empty array.

```js
app.post('/', [body('checkboxes').toArray()], (req, res, next) => {
  // ['foo', 'bar']  => ['foo', 'bar']
  // 'foo'          => ['foo']
  // undefined      => []
  console.log(req.body.checkboxes);
});
```

### `.toLowerCase()`

> _Returns:_ the current sanitization chain instance

Converts the value to lower case. Non string value will return itself.

```js
app.post('/', [body('username').toLowerCase()], (req, res, next) => {
  // 'Foo'      => 'foo'
  // undefined  => undefined
  // null       => null
  console.log(req.body.username);
});
```

### `.toUpperCase()`

> _Returns:_ the current sanitization chain instance

Converts the value to upper case. Non string value will return itself.

```js
app.post('/', [body('username').toUpperCase()], (req, res, next) => {
  // 'Foo'      => 'FOO'
  // undefined  => undefined
  // null       => null
  console.log(req.body.username);
});
```
