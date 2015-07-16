# describe('upNgSpecStyle', function () {

*A style guide to good Angular testing practices*

## Linting

Tests are code, so they should be linted.

## Wrap spec in `describe` block

Follow this convention for the message: `<name-of-fixture>:`.

```javascript
describe('nameOfFixture:', function () {
    'use strict';
});
```

## Wrap conditions in `context` block

```javascript
// Bad
it('should do something when something sets up', function () {
    // ...
});

// Good
context('when something sets up', function () {
    it('should do something', function () {
        // ...
    });
});
```

## Use Sinon stubs

Stubs allow observation of existing methods on fixtures and forcing the code down different paths to test all branches.

```javascript
// Bad
var serviceMock = sinon.mock({});

// Good
var method = sinon.stub(service, 'method');
```

## Browsers

There's no need for fancy Karma launchers. Just launch Karma and visit your IP address at the Karma hosted port (`9876`) on any device on the network.

This is helpful to test in IE 9 and IE 10.

# });
