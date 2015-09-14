# describe('upNgSpecStyle', function () {

*A style guide to good Angular testing practices*

**THIS IS A PREVIEW DOCUMENT, RULES ARE SUBJECT TO CHANGE**

## Table of Contents

1. [Linting](#linting)
1. [Unit Testing Framework](#unit-testing-framework)
1. [Describe and Context Blocks](#describe-and-context-blocks)
1. [Scope](#scope)
1. [Sinon](#sinon)
1. [ngMock](#ngmock)
1. [Browsers](#browsers)

## Linting
### Why use it?

Tests are code, so they should be linted.

## Unit Testing Framework

### [Karma](http://karma-runner.github.io/)

> A simple tool that allows you to execute JavaScript code in multiple real browsers.

Karma manages the configuration and setup for our unit testing.

### [Mocha](http://mochajs.org/)

> Mocha is a feature-rich JavaScript test framework running on Node.js and the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases.

Mocha is the core of our testing framework. Mocha tests are structured into blocks of `describe`, `context`, and `it` calls. `describe` blocks describe a thing, `context` blocks describe a set of context in which a unit is run, and `it` describes the expected behavior.

```javascript
describe('Thing', function () {
    context('on init', function () {
        it('returns `true`', function () {
            // ...
        });
    });
});
```

### [Chai](http://chaijs.com/)

> Chai is a BDD / TDD assertion library for node and the browser that can be delightfully paired with any javascript testing framework.

Chai handles our assertions. We use the BDD style assertions.

```javascript
expect(result).to.be.true();
expect(obj).to.have.property('foo', 'bar');
```

### [Sinon](https://github.com/domenic/sinon-chai)

> Standalone and test framework agnostic JavaScript test spies, stubs and mocks.

Sinon provides our mocking functionality. We prefer to use mocks as much as possible as they allow you to finely control the control flow of your program.

```javascript
var contextService = {
   getToolContext: sinon.stub()
};

// ...

contextService.getToolContext.returns({});
```

### [Sinon-Chai](https://github.com/domenic/sinon-chai)

> Sinon–Chai provides a set of custom assertions for using the Sinon.JS spy, stub, and mocking framework with the Chai assertion library. You get all the benefits of Chai with all the powerful tools of Sinon.JS.

We prefer to use Sinon assertions without wrapping mocks in an `expect` call. Use `calledWithExactly` over `calledWith` when possible as it more strictly defines expectations. Use `calledOnce` and other specifiers for the amount of times when possible as well.

```javascript
contextService.getToolContext.should.have.been.calledOnce
    .and.calledWithExactly('color', 'Purple Mountains Majesty');
```

## Describe and Context Blocks
### Wrap spec in `describe` block

Follow this convention for the message: `<name-of-fixture>:`. Do not include the word “spec” in the name

*Why?:* 
Spec has already been established by the filename.

```javascript
describe('nameOfFixture:', function () {
    'use strict';
});
```
### Use `describe` blocks for methods/properties
This establishes a standardized structure for tests, allowing future readers to quickly scan and find important parts.

* Instance methods/properties are prefixed with a `#`
* Static methods are prefixed with a `.`
* Functions are suffixed with a `()`
* Properties have no suffix

*Why?:* Standardized indicators help establish the scope of a test

```javascript
// Bad
it('find does something when something set up', function () {
    // ...
});

// Better

describe('find()', function () {
    it('does something', function () {
        // ...
    });
});

// Best
describe('#find()', function () {
    it('does something', function () {
        // ...
    });
});
```

### Use `context` blocks for circumstantial groups
Think of context as 'when this...'

```javascript
// Bad
describe('when passed a variable', function () {
    // ...
});

// Good
context('when passed a variable', function () {
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

```javascript
describe('something', function () {
    var $rootScope;
    var element;
    var scope;

    beforeEach(inject(function ($compile, $rootScope) {
        scope = $rootScope.$new();

        element = $compile('<div></div>')(scope);

        scope.$apply();
    }));

    it('is something', function () {
        // ...
    });
});
```

## Sinon
### Use Sinon stubs

Stubs allow observation of existing methods on fixtures and forcing the code down different paths to test all branches.

Mocks have been deprecated and will not be supported in `sinon@2`.

```javascript
// Bad
var serviceMock = sinon.mock({});

// Good
var method = sinon.stub(service, 'method');

var contextService = {
    getToolContext: sinon.stub()
};
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

Use the alias to `angular.mock.module` ngMock publishes directly on the window under `module` when testing.

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

### Use `angular.module` for mocking other modules

To mock out your dependencies on other modules, declare them with `angular.module`.

```javascript
beforeEach(function () {
    angular.module('someoneElsesModule', []);
});
```

### Use `$provide` to mock services from other modules

To mock out a service from another module, use ngMock’s `module` function to modify provided services. Passing a function to to `module` works like an Angular `config` statement, which is run before the application is bootstrapped.

```javascript
beforeEach(module(function ($provide) {
    upContextService = {
        getToolContext: sinon.stub()
    };

    $provide('upContextService', upContextService);
}));
```

## Browsers
### Karma Launchers

There's no need for fancy Karma launchers. Just launch Karma and visit your IP address at the Karma hosted port (`9876`) on any device on the network.

This is helpful to test in IE 9 and IE 10.

The `--verbose` flag of the 'up' build system will print your external IP. (`up s --verbose` or `up ta --verbose`)

# });
