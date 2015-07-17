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

Mocks have been deprecated and will not be supported in `sinon@2`.

```javascript
// Bad
var serviceMock = sinon.mock({});

// Good
var method = sinon.stub(service, 'method');
```

## Use ngMock `inject`

ngMock’s `inject` function allows you to resolve references to Angular fixtures.

When assigning references to a variable, prefix and suffix the reference name with underscores to avoid naming collisions.

```javascript
// Good
var myService;

beforeEach(inject(function (_myService_) {
    myService = _myService_;
}));
```

## Use ngMock `module`

ngMock’s `module` function allows you to initialize modules and modify providers. The preferred method is to initialize your module and allow it to initialize its dependencies automatically. It is important that your module properly declares its dependencies on other Angular modules.

ngMock publishes the function under `angular.mock.module` or directly on the window under `module` when testing.

```javascript
// Bad
beforeEach(function () {
    angular.module('yourModule', []);
});

// Better
beforeEach(function () {
    angular.mock.module('yourModule');
});

// Best
beforeEach(function () {
    module('yourModule');
});
```

## Browsers

There's no need for fancy Karma launchers. Just launch Karma and visit your IP address at the Karma hosted port (`9876`) on any device on the network.

This is helpful to test in IE 9 and IE 10.

# });
