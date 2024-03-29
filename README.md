# describe('upNgSpecStyle', function () {

*A style guide to good Angular testing practices*

## Table of Contents

1. [File Naming](#file-naming)
1. [Linting](#linting)
1. [Unit Testing Framework](#unit-testing-framework)
    1. [Karma](#karma)
    1. [Mocha](#mocha)
    1. [Chai](#chai)
    1. [Sinon](#sinon)
    1. [Sinon-Chai](#sinon-chai)
1. [Coverage](#coverage)
1. [Describe and Context Blocks](#describe-and-context-blocks)
    1. [Wrap spec in `describe` block](#wrap-spec-in-describe-block)
    1. [Use `describe` blocks for methods/properties](#use-describe-blocks-for-methodsproperties)
    1. [Use `context` blocks for circumstantial groups](#use-context-blocks-for-circumstantial-groups)
    1. [`it` blocks should be declarative](#it-blocks-should-be-declarative)
    1. [`.only`](`.only`)
1. [Scope](#scope)
    1. [Each test starts and ends in an isolate scope](#each-test-starts-and-ends-in-an-isolate-scope)
    1. [Scoped Variables](#scoped-variables)
1. [Sinon](#sinon)
    1. [Spies](#spies)
    1. [Stubs](#stubs)
    1. [Mocks](#mocks)
1. [Async](#async)
    1. [`$httpBackend`](#httpbackend)
        1. [`expect`](#expect)
        1. [`flush`](#flush)
        1. [Verify No Outstanding Expectations or Requests](#verify-no-outstanding-expectations-or-requests)
    1. [`$timeout`](#timeout)
    1. [`done`](#done)
1. [`$controller`](#controller)
1. [Digest Cycle](#digest-cycle)
1. [`$compile`](#compile)
1. [ngMock](#ngmock)
    1. [Use ngMock `inject`](#use-ngmock-inject)
    1. [Use ngMock `module`](#use-ngmock-module)
    1. [Use `angular.module` for mocking other modules](#use-angularmodule-for-mocking-other-modules)
    1. [Use `$provide` to mock services from other modules](#use-provide-to-mock-services-from-other-modules)
    1. [Don’t use `$provide` when testing controllers](#dont-use-provide-when-testing-controllers)
1. [Browsers](#browsers)
    1. [Karma Launchers](#karma-launchers)
1. [Examples](#examples)
    1. [Services](#services)
    1. [Controllers](#controllers)
    1. [Directives](#directives)


## File Naming
### Naming and Location

Using consistent names for file names is as important as properly named functions and methods. We use `feature.type.spec.js` for testing file names.

Spec files reside next to the code they are testing. There are no `test` directories in any front-end Uptake code. 

*Why?:* Naming conventions allow you to find code at a glance

Example: 
```
└── auth
    ├── auth.directive.js
    ├── auth.directive.spec.js
    ├── auth.html
    ├── auth.service.js
    └── auth.service.spec.js
```

## Linting
### Why use it?

Tests are code, so they should be linted.

## Unit Testing Framework

### [Karma](http://karma-runner.github.io/)

> A simple tool that allows you to execute JavaScript code in multiple real browsers.

Karma manages the configuration and setup for our unit testing. Karma also provides our coverage reporting.

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

Sinon provides our mocking functionality. We prefer to use stubs as much as possible as they allow you to finely control the control flow of your program.

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

## Coverage

Code coverage measures how much of the source code of a project is executed by tests. It indicates coverage by statements, branches, functions, and lines. 

Code coverage should not be treated as a quality target, but a tool to discover untested code.

It is important to actually test all parts of a program, not just execute all parts. 

Assert that methods have the expected effect, that directives compile correctly, and that config and run blocks execute as expected.

## Describe and Context Blocks
### Wrap spec in `describe` block

Follow this convention for the message: `<name-of-fixture>`. Do not include the word “spec” in the name.

*Why?:* 
Spec has already been established by the filename.

```javascript
describe('nameOfFixture', function () {
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

#### Nested `describe` blocks
Only in very rare cases are nested `describe` blocks appropriate to use outside of the initial file set up. 

If a method returns an object that has methods, it would be an appropriate to use a nested `describe` block. Almost all cases can be solved using a [context](#context) block.

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

#### Nested `context` blocks
Use nested `context` blocks when your `it` statement includes words like `when, if, where, while, etc...`

*Why?:* These are logical operators and means there are several situations that should to be tested  
*Why?:* Helps break up tests into groups that represent functionality

Used in situations similar to an `if` statement. It is frowned up to have `it` statements and `context` blocks on the same level.
 
*Why?:* If one situation requires more context, all situations can benefit from further explanation. 

Example:
```javascript
describe('#updateTool()', function () {
    beforeEach(function () {
        // set up
    });

    context('when valid', function () {
        context('when optional param is passed', function () {
            // do not put an it statement here if more context blocks are possible
            context('when when api call succeeds', function () {
                // it statements 
            }); 
        });
        context('when optional param is not passed', function () {
            // it statements 
        });
    });

    context('when invalid', function () {
        // it statements
    });
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

### `.only`

Adding `.only` to a `describe`, `context`, or `it` block will run only the specified suite. This is helpful to focus on testing areas of code.

**Always** remove your `.only` before pushing your code.

See: [Exclusive Tests](https://mochajs.org/#exclusive-tests)

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
### [Spies](http://sinonjs.org/docs/#spies)

A test spy intercepts the data used to call a function, along with other information, and allows it to continue on with normal operation. Only use spies when testing if an angular core module (e.g. $rootScope) was called, but not changing its functionality.  

```javascript
// While not in `contextService`
// Bad
beforeEach(function () {
    sinon.spy(contextService, 'getToolContext');
}

// Good
beforeEach(function () {
    sinon.spy($rootScope, '$emit');
}
```
#### Example

In this service when `#resetCache()` is called it triggers `$rootScope.$emit('up.core.navigationBar.reinit');`. The test needs to ensure this is called just once, but also needs the original functionality to happen. 

```javascript
beforeEach(module('up.core.applications'));

beforeEach(inject(function(_$rootScope_) {
    $rootScope = _$rootScope_;

    sinon.spy($rootScope, '$emit');
}));

afterEach(function () {
    $rootScope.$emit.restore();
});

describe('#resetCache()', function () {
    it('emits "up.core.navigationBar.reinit"', function() {
        upContextService.resetCache();

        $rootScope.$emit.should.have.been.calledOnce
            .and.calledWith('up.core.navigationBar.reinit');
    });
});
```
Here $rootScope is still functioning as expected, but allows insight into how is is being called.

### [Stubs](http://sinonjs.org/docs/#stubs)

Generally stubs are the best option when testing code. Stubs allow observation of existing methods on fixtures and forcing the code down different paths to test all branches.

```javascript
// Worst
var serviceMock = function () { return 'string' };

// Bad
var serviceMock = sinon.mock({});

// Good
var method = sinon.stub(service, 'method');

var contextService = {
    getToolContext: sinon.stub()
};
```

#### Example

In this example there is an injection of the service `upUserService` which has a method `getCurrentUser` used by `LogOutController`. Since this test covers a controller, the service functionality needs to be mocked out. 

Without `sinon.stub()`:
```javascript
beforeEach(function () {
    upUserService = {
        getCurrentUser: function () {
            return {name: 'foo', email: 'bar'}
        }
    }
    
    LogOutController = $controller('LogOutController', {
        upUserService: upUserService
    }
    
    describe('#logout()', function() {
        context('when a user is not logged in', function() {
            it('does not call the logout API', function () {
                LogOutController.logout();
                // This test doesn't work 
            });
        });
    });
}
```

Because every time `getCurrentUser` is called it returns an instance of user, how can a test check out when the return value is `null`? 

With `sinon.stub()`:
```javascript
beforeEach(function () {
    upUserService = {
        getCurrentUser: sinon.stub()
    }
    
    LogOutController = $controller('LogOutController', {
        upUserService: upUserService
    }
    
    describe('#logout()', function() {
        context('when a user is not logged in', function() {
            beforeEach(function () {
                upUserService.getCurrentUser.returns(null);
            });
            
            it('does not call the logout API', function () {
                LogOutController.logout();
                // Test if the API was called
            });
        });
    });
}
```

As demonstrated in this example, the tests can now cover both positive and negative cases of `getCurrentUser` without relying on `upUserService`.  
This can also be used to test how many times `getCurrentUser` was called. If the expectation is for it to be called twice, but it was only called once an easily missed bug is caught. In the original implementation there was no possible way to test this. 

Stubs are extremely useful in the unit testing. Use them whenever possible.

### [Mocks](http://sinonjs.org/docs/#mocks)

Mocks will be deprecated with sinon@2. Do not use them and replace any instance with spies or stubs.

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

To mock out a service from another module, use ngMock’s `module` function to modify provided services. Passing a function to to `module` works like an Angular `config` statement, which is run before the application is bootstrapped. The order of mocked modules is important, so put your mocks intended for early running `config` and `run` statements before other mocks.

```javascript
beforeEach(module(function ($provide) {
    upContextService = {
        getToolContext: sinon.stub()
    };

    upNavigationProvider = {
        registerEntitySearchState: angular.noop,
        $get: {}
    };

    $provide.value('upContextService', upContextService);
    $provide.provider('upNavigation', upNavigationProvider);
}));
```

#### Use `$controller` when testing controllers

Controllers allow you to directly provide services. This makes overriding provided services unnecessary.

```javascript
// Bad:
$provide('upContextService', upContextService);
$controller('ExampleController');

// Good:
$controller('ExampleController', {
     upContextService: upContextService
});
```

## Async
### [`$httpBackend`](https://docs.angularjs.org/api/ngMock/service/$httpBackend)

> Fake HTTP backend implementation suitable for unit testing applications that use the $http service.

Use ngMock's fake `$httpBackend` implementation to test your use of `$http` without actually making XHR requests.

#### [`expect`](https://docs.angularjs.org/api/ngMock/service/$httpBackend#expect)

> Request expectations provide a way to make assertions about requests made by the application and to define responses for those requests. The test will fail if the expected requests are not made or they are made in the wrong order.

We use request expectations as the expectations are helpful to assert that calls are being made as expected.

`$httpBackend` also includes aliases for the standard HTTP methods:

* `expectGET`
* `expectHEAD`
* `expectDELETE`
* `expectPOST`
* `expectPUT`
* `expectPATCH`
* `expectJSONP`

```javascript
var thing = {
    name: 'foo'
};

$httpBackend.expectPOST('/api/v1/thing').respond(200, thing);

$httpBackend.expectGET('/api/v1/thing').respond(200, thing);

$httpBackend.expectDELETE('/api/v1/thing').respond(200, thing);

$httpBackend.expectGET('/api/v1/thing').respond(404, '');
```

#### [`flush`](https://docs.angularjs.org/api/ngMock/service/$httpBackend#flushing-http-requests)

Make sure to flush your requests using `$httpBackend.flush()`. This causes pending HTTP requests to be processed.

> This preserves the async api of the backend, while allowing the test to execute synchronously.

```javascript
$httpBackend.expectGET('/foo').respond(401, '');

$httpBackend.flush();
```

#### [Verify No Outstanding Expectations or Requests](https://docs.angularjs.org/api/ngMock/service/$httpBackend#verifyNoOutstandingExpectation)

`$httpBackend` provides two methods to ensure no outstanding requests or expectations were made. If outstanding requests or expectations are found, an exception is thrown. Include this in an `afterEach` block for your test suite.

```javascript
describe('UpController', function () {
    afterEach($httpBackend.verifyNoOutstandingExpectation);
    afterEach($httpBackend.verifyNoOutstandingRequest);
});
```

### [`$timeout`](https://docs.angularjs.org/api/ngMock/service/$timeout)

> This service is just a simple decorator for $timeout service that adds a "flush" and "verifyNoPendingTasks" methods.

ngMock’s `$timeout` implementation allows you to cycle through timeout schedules by calling `flush` with an interval to flush up to. The ngMock implementation emulates an advance in time without having to wait for the actual duration.

ngMock’s `$timeout` also has a verification method, `verifyNoPendingTasks`, that verifies that all timeouts have been flushed.

```javascript
// advance 1 second
$timeout.flush(1000);

$timeout.verifyNoPendingTasks();
```

### `done`

Mocha’s `it` blocks provide a `done` handler which allows you to specify when an async test is complete. Even though ngMock makes our tests relatively synchronous, it is possible for tests to execute and skip callbacks in which key assertions are made. Avoid missing these callbacks by using `done`, which will fail tests that do not call the done callback.

```javascript
it('does something async', function (done) {
    $httpBackend.expectGET('/contexts')
        .respond(200, {});

    upContextService.getToolContext()
        .then(function (response) {
            expect(response).to.be.an('object');
            done();
        });

    $httpBackend.flush();
});
```

## [`$controller`](https://docs.angularjs.org/api/ngMock/service/$controller)

> `$controller` service is responsible for instantiating controllers.

The `$controller` services allows you to instantiate a controller for testing. It requires the name of the controller as a string.

Optionally, you can specify locals, which allow you to add mock dependencies which override the standard provided dependencies. This is helpful for mocking services and injecting dependencies coming from state resolves.

ngMock’s implmentation of `$controller` is:

> A decorator for `$controller` with additional `bindings` parameter, useful when testing controllers of directives that use `bindToController`.

```javascript
UpThingController = $controller('UpThingController', {
    upContextService: upContextService
}, {
    context: {
        foo: 'bar'
    }
});
```

## Digest Cycle

Angular uses the scope digest cycle for many internal functionalities, not just modifying scopes.

* **`$httpBackend`:**
    `$httpBackend.flush()` calls `$apply` behind the scenes. You should **not** call `$scope.$apply()` immediately after `$httpBackend.flush()` as it is redundant.
* **`$q`:**
    `$scope.$apply()` must be called to advance promises to their resolved state.
* **`$timeout`:**
    In absence of a `process.nextTick()` method, using `$timeout` without a delay pushes a callback into the next digest cycle. It is necessary to advance the digest cycle to call these callbacks.

## [`$compile`](https://docs.angularjs.org/api/ng/service/$compile)

> Compiles an HTML string or DOM into a template and produces a template function, which can then be used to link `scope` and the template together.

It is important to `$apply` the scope after rendering the element, as the initial scope must be applied for the HTML to be generated and the directive to be bootstrapped.

The resulting JQLite wrapper allows you to inspect the DOM element, its scope, and isolated scope.

It is frequently helpful to define an wrapper function which compiles and renders with the scope.

```javascript
var scope;

beforeEach(function () {
    scope = $rootScope.$new();
});

function getElement(scope) {
    return $compile('<up-example></up-example>')(scope);
}

it('does something', function () {
    var element = getElement(scope);

    scope.$apply();

    var isolateScope = element.isolateScope();
    var scope = element.scope();
});
```

## Browsers
#### Karma Launchers

There's no need for fancy Karma launchers. Just launch Karma and visit your IP address at the Karma hosted port (`9876`) on any device on the network.

This is helpful to test in IE 9 and IE 10.

The `--verbose` flag of the 'up' build system will print your external IP. (`up s --verbose` or `up ta --verbose`)

## Examples
### Services

Testing a service is as easy as injecting a dependency. Keep naming consistent to improve readability of tests.
 
```javascript
beforeEach(module('up.context-editor', function ($provide) {
    upApplicationsService = { 
        save: sinon.stub()
    };
    
    $provide.value('upApplicationService', upApplicationService);
}));

beforeEach(inject(function (_upCeContextsService_) {
    upCeContextsService = _upCeContextsService_;
}));

describe('#saveApplication()', function () {
    context('with an organizationId on the model', function () {
        it('uses it immediately with the save', function () {
            upCeContextsService.saveApplication(appInput);

            upApplicationsService.save.shoud.have.been.calledOnce
                .and.calledWith({'organizationId': 'MAW'});
        });
    });
});

```

### Controllers

Use an `init` function, this allows you to change or test any behavior before a controller is initialized. Keep naming consistent to improve readability of tests. 

```javascript
beforeEach(module('up.context-editor.edit'));

beforeEach(inject(function (_$controller_) {
    $controller = _$controller_;

    stateParams = {
        appId: 'sampleId'
    };
}));

function init() {
    CeSelectLevelController = $controller('CeSelectLevelController', {
        $stateParams: stateParams
    });
}

describe('#appId', function () {
    beforeEach(function () {
        init();
    });

    it('is defined', function () {
        expect(CeSelectLevelController.appId).to.equal('sampleId');
    });
});
```

### Directives

When testing directives the HTML needs to be set to be compiled, using [`$compile`](https://docs.angularjs.org/api/ng/service/$compile). This allows angular to link `scope` and the directive together.

```javascript
beforeEach(module('up.layout'));

beforeEach(module(function ($provide) {
    upUserService = {
        getActiveUser: sinon.stub()
    };

    $provide.value('upUserService', upUserService);
}));

beforeEach(inject(function ($compile, _$rootScope_, _$timeout_) {
    scope = _$rootScope_.$new();
    $timeout = _$timeout_;
    $compile('<nav up-header ></nav>')(scope);
}));

describe('#isOpen', function () {
    context('when the user just logged in', function () {
        beforeEach(function () {
            upUserService.justLoggedIn = true;
        });

        it('opens the dropdown', function () {
            $timeout.flush();
            expect(scope.isOpen).to.be.true;
        });
    });

    context('when the user was already logged in', function () {
        beforeEach(function () {
            upUserService.justLoggedIn = false;
        });

        it('does not open the dropdown', function () {
            scope.$apply();
            expect(scope.isOpen).to.be.false;
        });
    });
});
```

If the directive has data bindings use a `compile` function to isolate the HTML elements of a directive. This allows you to test different situations and types of data that may be passed.

```javascript
beforeEach(module(function ($provide) {
 
    upCeContextInputService = {
        readPredefinedContextFile: sinon.stub()
    };
    upConfirmationModalService = {
        open: sinon.stub()
    };

    $provide.value('upCeContextInputService', upCeContextInputService);
    $provide.value('upConfirmationModalService', upConfirmationModalService);
}));

beforeEach(inject(function (_$compile_, _$q_, _$rootScope_) {
    scope = _$rootScope_.$new();
    $rootScope = _$rootScope_;
    $compile = _$compile_;
    $q = _$q_;
}));

function compile(toolObject) {
    html = $compile(angular.element('<form name="myForm"><up-ce-context-input tool="tool"></up-ce-context-input></form>'))(scope);
    scope.$apply();
    isolateScope = angular.element((html.children())[0]).isolateScope();
}

describe('set predefinedContexts', function () {
    context('when predefinedContexts is already set', function () {
        it('does not overwrite', function () {
            compile({
                toolName: 'aTool',
                predefinedContexts: {something: 'here'}
            });

            expect(scope.tool.predefinedContexts).to.deep.equal({something: 'here'});
        });
    });
}):
```

# });
