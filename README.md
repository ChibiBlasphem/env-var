# env-var

[![Travis CI](https://travis-ci.org/evanshortiss/env-var.svg?branch=master)](https://travis-ci.org/evanshortiss/env-var)
[![Coverage Status](https://coveralls.io/repos/github/evanshortiss/env-var/badge.svg?branch=master)](https://coveralls.io/github/evanshortiss/env-var?branch=master)
[![npm version](https://badge.fury.io/js/env-var.svg)](https://www.npmjs.com/package/env-var)
[![TypeScript](https://img.shields.io/badge/%3C%2F%3E-TypeScript-blue.svg)](http://www.typescriptlang.org/)
[![npm downloads](https://img.shields.io/npm/dm/env-var.svg?style=flat)](https://www.npmjs.com/package/env-var)
[![Greenkeeper badge](https://badges.greenkeeper.io/evanshortiss/env-var.svg)](https://greenkeeper.io/)

Verification, sanatization, and type coercion for environment variables in
Node.js. This is particularly useful in TypeScript environments.

## Install

```
npm install env-var --save
```

## Usage
In the example below we read the environment variable *DB_PASSWORD*

```js
const env = require('env-var');

const PASSWORD = env.get('DB_PASSWORD')
  // Throws an error if the DB_PASSWORD variable is not set (optional)
  .required()
  // Convert DB_PASSWORD from base64 to a regular utf8 string (optional)
  .convertFromBase64()
  // Call asString (or other methods) to get the variable value (required)
  .asString();

// Read in a port or use a default value of 5432
const PORT = env.get('PORT', 5432).asIntPositive()
```

## TypeScript

```ts
import * as env from 'env-var';

// Read a PORT environment variable and ensure it's a positive number
// An EnvVarError will be thrown if the variable is not set, or is not a number
const PORT: number = env.get('PORT').required().asIntPositive();
```

## Benefits
Fail fast if your environment is misconfigured. Also, this code:

```js
const env = require('env-var');

const MAX_BATCH_SIZE = env.get('MAX_BATCH_SIZE').required().asIntPositive();
```

Is cleaner than this code:

```js
const assert = require('assert');

// Our program requires this var to be set
assert.notEqual(
  process.env.MAX_BATCH_SIZE,
  undefined,
  'MAX_BATCH_SIZE environment variable must be set'
);

// Read the var, and use parseInt to make it a number
const MAX_BATCH_SIZE = parseInt(process.env.MAX_BATCH_SIZE, 10);

// Verify we have a valid number, if not throw
assert(
  typeof MAX_BATCH_SIZE === 'number' && !isNaN(MAX_BATCH_SIZE),
  'MAX_BATCH_SIZE env var must be a valid number'
);

// Verify the number is positive
assert(
  MAX_BATCH_SIZE > 0,
  'MAX_BATCH_SIZE must be a positive number'
);
```

## API

### env.get([varname, [default]])
You can call this function 3 different ways:

```js
const env = require('env-var')

// #1 - Return the requested variable (we're also checking it's a positive int)
const limit = env.get('SOME_LIMIT').asIntPositive()

// #2 - Return the requested variable, or use the given default if it isn't set
const limit = env.get('SOME_LIMIT', '10').asIntPositive()

// #3 - Return the environment object (process.env by default - see env.from() docs for more)
const allvars = env.get()
```

### from(values)
This function is useful if you're not in a typical Node.js environment, or for
testing. It allows you to generate an env-var instance that reads from the
given `values` instead of the default `process.env`.

```js
const env = require('env-var').from({
  API_BASE_URL: 'https://my.api.com/'
})

// apiUrl will be 'https://my.api.com/'
const apiUrl = mockedEnv.get('API_BASE_URL').asUrlString()
```

### variable
A variable is returned by calling `env.get`. It has the exposes the following
functions to validate and access the underlying value.

#### required(isRequired = true)
Ensure the variable is set on *process.env*. If the variable is not set this
function will throw an `EnvVarError`. If the variable is set it returns itself
so you can access the underlying variable.

Can be bypassed by passing `false`, i.e `required(false)`

Full example:

```js
const env = require('env-var')

// Read PORT variable and ensure it's a positive integer. If it is not a
// positive integer or is not set the process will exit with an error (unless
// you catch it using a try/catch or "uncaughtException" handler)
const NODE_ENV = env.get('NODE_ENV').asString()
const PORT = env.get('PORT').required().asIntPositive()

// If mode is production then this is required, else use default
const SECRET = env.get('SECRET', 'bad-secret').required(NODE_ENV === 'production').asString()

app.listen(PORT)
```

#### convertFromBase64()
Sometimes environment variables need to be encoded as base64. You can use this
function to convert them before reading their value.

For example if we run the script script below, using the command `DB_PASSWORD=
$(echo -n 'secret_password' | base64) node`, we'd get the following results:

```js
console.log(process.env.DB_PASSWORD) // prints "c2VjcmV0X3Bhc3N3b3Jk"

// dbpass will contain the value "secret_password"
const dbpass = env.get('DB_PASSWORD').convertFromBase64().asString()
```

#### asEnum(validValues: string[])
Converts the value to a string, and matches against the list of valid values.
If the value is not valid, an error will be raised describing valid input.

#### asInt()
Attempt to parse the variable to an integer. Throws an exception if parsing
fails. This is a strict check, meaning that if the *process.env* value is "1.2",
an exception will be raised rather than rounding up/down.

#### asIntPositive()
Performs the same task as _asInt()_, but also verifies that the number is
positive (greater than zero).

#### asIntNegative()
Performs the same task as _asInt()_, but also verifies that the number is
negative (less than zero).

#### asFloat()
Attempt to parse the variable to a float. Throws an exception if parsing fails.

#### asFloatPositive()
Performs the same task as _asFloat()_, but also verifies that the number is
positive (greater than zero).

#### asFloatNegative()
Performs the same task as _asFloat()_, but also verifies that the number is
negative (less than zero).

#### asString()
Return the variable value as a String. Throws an exception if value is not a
String. It's highly unlikely that a variable will not be a String since all
*process.env* entries you set in bash are Strings by default.

#### asBool()
Attempt to parse the variable to a Boolean. Throws an exception if parsing
fails. The var must be set to either "true", "false" (upper or lowercase),
0 or 1 to succeed.

#### asBoolStrict()
Attempt to parse the variable to a Boolean. Throws an exception if parsing
fails. The var must be set to either "true" or "false" (upper or lowercase) to
succeed.

#### asJson()
Attempt to parse the variable to a JSON Object or Array. Throws an exception if
parsing fails.

#### asJsonArray()
The same as _asJson_ but checks that the data is a JSON Array, e.g [1,2].

#### asJsonObject()
The same as _asJson_ but checks that the data is a JSON Object, e.g {a: 1}.

#### asArray([delimiter: string])
Reads an environment variable as a string, then splits it on each occurence of
the specified _delimiter_. By default a comma is used as the delimiter. For
example a var set to "1,2,3" would become ['1', '2', '3'].

#### asUrlString()
Verifies that the variable is a valid URL string and returns that string. Uses
`is-url` to perform validation, so check that module for validation rules.

#### asUrlObject()
Verifies that the variable is a valid URL string, then parses it using
`url.parse` from the Node.js core `url` module and returns the parsed Object.
See the [Node.js docs](https://nodejs.org/api/url.html#url_url_parse_urlstring_parsequerystring_slashesdenotehost) for more info


### env.EnvVarError
This is the error class used to represent errors raised by this module. Sample
usage:

```js
const env = require('env-var')

try {
  // will throw if you have not set this variable
  env.get('MISSING_VARIABLE').required().asString()

  // if catch error is set, we'll end up throwing here instead
  throw new Error('some other error')
} catch (e) {
  if (e instanceof env.EnvVarError) {
    console.log('we got an env-var error', e)
  } else {
    console.log('we got some error that wasn\'t an env-var error', e)
  }
}
```

## Examples

```js
const env = require('env-var');

// Normally these would be set using "export VARNAME" or similar in bash
process.env.STRING = 'test';
process.env.INTEGER = '12';
process.env.BOOL = 'false';
process.env.JSON = '{"key":"value"}';
process.env.COMMA_ARRAY = '1,2,3';
process.env.DASH_ARRAY = '1-2-3';

// The entire process.env object
const allVars = env.get();

// Returns a string. Throws an exception if not set
const stringVar = env.get('STRING').required().asString();

// Returns an int, undefined if not set, or throws if set to a non integer value
const intVar = env.get('INTEGER').asInt();

// Return a float, or 23.2 if not set
const floatVar = env.get('FLOAT', '23.2').asFloat();

// Return a Boolean. Throws an exception if not set or parsing fails
const boolVar = env.get('BOOL').required().asBool();

// Returns a JSON Object, undefined if not set, or throws if set to invalid JSON
const jsonVar = env.get('JSON').asJson();

// Returns an array if defined, or undefined if not set
const commaArray = env.get('COMMA_ARRAY').asArray();

// Returns an array if defined, or undefined if not set
const commaArray = env.get('DASH_ARRAY').asArray('-');

// Returns the enum value if it's one of dev, test, or live
const enumVal = env.get('ENVIRONMENT').asEnum(['dev', 'test', 'live'])
```

## Contributors
* @caccialdo
* @evanshortiss
* @hhravn
* @itavy
* @MikeyBurkman
* @pepakriz
* @rmblstrp

## Contributing
Contributions are welcomed. If you'd like to discuss an idea open an issue, or a
PR with an initial implementation.

If you want to add a new type it's easy. Add a file to `lib/accessors`,
with the name of the type e.g add a file named `number-zero.js` into that folder
and populate it with code following this structure:

```js
/**
 * Validate that the environment value is an integer and equals zero.
 * @param {Function} raiseError use this to raise a cleanly formatted error
 * @param {String}   environmentValue this is the string from process.env
 */
module.exports = function numberZero (raiseError, environmentValue) {

  // Your custom code should go here...below code is an example

  const val = parseInt(environmentValue)

  if (val === 0) {
    return ret;
  } else {
    raiseError('should be zero')
  }
}
```

Next update the `accessors` Object in `getVariableAccessors()` in
`lib/variable.js` to include your new module. The naming convention should be of
the format "asTypeSubtype", so for our `number-zero` example it would be done
like so:

```js
asNumberZero: generateAccessor(container, varName, defValue, require('./accessors/number-zero')),
```

Once you've done that, add some unit tests and use it like so:

```js
// Uses your new function to ensure the SOME_NUMBER is the integer 0
env.get('SOME_NUMBER').asNumberZero()
```
