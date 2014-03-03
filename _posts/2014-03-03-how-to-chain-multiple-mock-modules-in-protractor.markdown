---
layout: post
title: "How to chain multiple mock modules in protractor"
date: 2014-03-03 15:31:51
author: alexander
categories: [Angular,Rails]
---

In this blog postI will explain how you can dynamically add and chain mock
modules. Mock modules are very similar to the angular modules you use elsewhere
in you angular application.

The main difference is that mock modules are wrapped inside a function, there
are multiple methods to workaround this limitation all of which i will cover
later on.

## Why would I want to chain mock modules ?
The reason you want to chain mock modules is because you want to avoid mocking
all API requests in a single module. You should split your mocked API requests
over several smaller mock modules preferably creating a separate mock module for
each action/process inside your application.

Keeping your mock modules small allows you to cherry-pick what API requests you
want to mock. This flexibility is the main reason why it's so important to keep
your mock modules small and extensible.

If you want to cherry-pick several mock modules you need to find a way to chain
them together and this blog-post will explain how to achieve
this.

## How to chain multiple mock modules

In order to chain multiple mock modules we first need to define the mock modules
we want to chain then add them one by one with `browser.addMockModule`

```javascript
var httpBackendMock = function () {
  angular.module('httpBackendMock', ['ngMockE2E', 'nameOfYourAngularApp'])
    .run(function ($httpBackend) {
      //http backend mocks here
    });
};

var anotherHttpBackendMock = function () {
  angular.module('anotherHttpBackendMock', ['ngMockE2E', 'nameOfYourAngularApp'])
    .run(function ($httpBackend) {
      //http backend mocks here
    });
};

beforeEach(function(){
  browser.addMockModule('httpBackendMock', httpBackendMock);
  browser.addMockModule('anotherHttpBackendMock', anotherHttpBackendMock);
});
```

## How to remove a specific mock module
Sometimes you want to remove a specific mock module in a It block.
<del>At the moment there are only two methods to remove mock modules</del>.

As of protractor version 
<a href="https://github.com/angular/protractor/releases/tag/0.19.0">V0.19.0</a>
a <a href="https://github.com/angular/protractor/pull/520">
pullrequest</a> of mine is added which makes it possible to remove individual
mock modules with the `removeMockModule('moduleName')` function. from all those
methods `method 3` is by far the preferred option if you want to remove a
specific mock module.

### method1: remove all mock modules then add the ones you want to keep.
```javascript
//load mockmodules
beforeEach(function(){
  browser.addMockModule(httpBackendMock', httpBackendMock);
  browser.addMockModule('HttpBackendMockTwo', HttpBackendMockTwo);
  browser.addMockModule('HttpBackendMockThree', HttpBackendMockThree);
});

it("should have acces to first two mockmodules in this block", function() {
  browser.clearMockModules();
  browser.addMockModule('httpBackendMock',httpBackendMock);
  browser.addMockModule('HttpBackendMockTwo', HttpBackendMockTwo);

  //rest of it block
});
```

### method2: override the mock module with a new empty one
```javascript
browser.addMockModule('nameMockModuleToOveride',function(){

});
```

### method3: Remove individual module with the removeMockModule() function.
```javascript
browser.removeMockModule('nameMockModuleToRemove');
```


## How to remove all mock modules after each test
Sometimes you want to remove all loaded mock modules after each test or in
specific tests.

### remove all mock modules after each test
```javascript
afterEach(function(){
  browser.clearMockModules();
});
```

### remove all mock modules in a It block
This method is used in situations where most of your it blocks require a set of
mock modules which are loaded in a `beforeEach` except for a few it blocks
that don't need those mock modules.

```javascript
//load mockmodules
beforeEach(function(){
  browser.addMockModule('httpBackendMock', httpBackendMock);
  browser.addMockModule('anotherHttpBackendMock', anotherHttpBackendMock);
});

it("should have acces to mockmodules in this block", function() {
  //rest of it block
});

it("should have acces to mockmodules in this block too", function() {
  //rest of it block
});

//... more it statements

it("should not have mock modules in this block", function() {
  browser.clearMockModules(); //remove all modules that are loaded in beforeEach

  //rest of it block
});
```
