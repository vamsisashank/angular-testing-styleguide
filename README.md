# describe('upNgSpecStyle', function () {

*A style guide to good Angular testing practices*

## Table of Contents

1. [Linting](#linting)
1. [Describe and Context Blocks](#describe-and-context-blocks)
1. [Scope](#scope)
1. [Sinon](#sinon)
1. [ngMock](#ngmock)
1. [Browsers](#browsers)

## Linting
### Why use it?

Tests are code, so they should be linted.

## Describe and Context Blocks
### Wrap spec in `describe` block

Follow this convention for the message: `<name-of-fixture>:`. Do not include the word `spec` in the name

*Why?:* 
Spec has already been established by the filename.

```javascript
describe('nameOfFixture:', function () {
    'use strict';
});
```
### Use `context` blocks for methods/properties
This establishes a standardized structure for tests, allowing future readers to quickly scan and find important parts.

* Instance methods/properties are prefixed with a `#`
* Static methods are prefixed with a `.`

```javascript
// Bad
it('does something when something set up', function () {
    // ...
});

// Better

context('find()', function () {
    it('does something', function () {
        // ...
    });
});

// Best
context('#find()', function () {
    it('does something', function () {
        // ...
    });
});
```

### Use `describe` blocks for circumstantial groups
Think of describe as 'when this...'

```javascript
// Bad
context('when passed a variable', function () {
    // ...
});

// Good
describe('when passed a variable', function () {
    // ...
});
```

### `it` blocks should be declarative
Do not start tests with `should`

*Why?:* `should...` becomes repetitive

*Why?:* Angular is a declarative language and our tests follow that convention. We are not testing what steps
our program *should* go through, we are testing what the output is. Our tests do not care about what
steps are necessary to get to the end point, only that the end point is valid. 

```javascript
// Bad
it('should map arrays', function () {
    // ...
});

// Good
it('maps arrays', function () {
    // ...
};
```

## Scope
### Each test starts and ends in an isolate scope
Ensures that there are no side effects between tests

*Why?:* You test what you expect to test and nothing else

```javascript
// Bad
before(function () {
    // ...
});

after(function () {

});

// Good

beforeEach(function () {
    // ...
});

afterEach(function() {
    // ...
});
```
### Scoped Variables
Each scoped variable needs to be placed inside a beforeEach declaration

- Prevents leaky values and side effects
    
Injected ng service components maintain their `$`

- Maintains readability and expectations


## Sinon
### Use Sinon stubs

Stubs allow observation of existing methods on fixtures and forcing the code down different paths to test all branches.

Mocks have been deprecated and will not be supported in `sinon@2`.

```javascript
// Bad
var serviceMock = sinon.mock({});

// Good
var method = sinon.stub(service, 'method');
```

## ngMock
### Use ngMock `inject`

ngMock’s `inject` function allows you to resolve references to Angular fixtures.

When assigning references to a variable, prefix and suffix the reference name with underscores to avoid naming collisions.

```javascript
// Good
var myService;

beforeEach(inject(function (_myService_) {
    myService = _myService_;
}));
```

### Use ngMock `module`

ngMock’s `module` function allows you to initialize modules and modify providers. The preferred method is to initialize your module and allow it to initialize its dependencies automatically. It is important that your module properly declares its dependencies on other Angular modules.

ngMock publishes the function under `angular.mock.module` or directly on the window under `module` when testing.

```javascript
// Bad
beforeEach(function () {
    angular.module('yourModule', []);
});

// Bad
beforeEach(function () {
    angular.mock.module('yourModule');
});

// Best
beforeEach(function () {
    module('yourModule');
});
```

## Browsers
### Karma Launchers

There's no need for fancy Karma launchers. Just launch Karma and visit your IP address at the Karma hosted port (`9876`) on any device on the network.

This is helpful to test in IE 9 and IE 10.

The `--verbose` flag of the 'up' build system will print your external IP. (`up s --verbose` or `up ta --verbose`)

# });
