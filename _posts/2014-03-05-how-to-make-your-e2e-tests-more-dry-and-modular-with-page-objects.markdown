---
layout: post
title: "how to make your E2E tests more DRY and modular with Page objects"
date: 2014-03-05 15:24:18
author: alexander
categories: [Angular,Rails]
---

## What is the pageObject design pattern ?.
One of the goals of integration tests is to test the UI of your web App and how
users interact with it.

A pageObject is a collection of elements and methods that manipulate those
elements.

Most of the things your doing in your integration tests like selecting
a DOM element or clicking a button basically everything that goes prior to the
expectation/evaluation is moved to a pageObject.

#### things you would put in a pageObject:
- element selectors: `element(by.selector)`
- element manipulations: `element(by.selector).click()`
- http backend mocks for the current page.

#### things you shouldn't put in a pageObject:
- expect statements:: `expect(condition).toBe(true)`
- describe blocks.
- it blocks.

## Why would I want to use pageObjects ?.

Having all the element selectors and the methods where you manipulate those in a
separate file for each actual page on your website has a lot of benefits.

First of all it reduces the size of your integration tests and improves the
readability of your test suites.

Another benefit is that this approach reduces the amount of duplicated code and
that means that if the UI changes, the fix need only be applied in one place.

The reusability of pageObjects is in my opinion the most important benefit.
How often have you come across different suites that both need access to the
same set of API mocks.The use of pageObjects solves this issue without having to
duplicate the mock modules.


## How to create a pageObject.
A pageObject is just a JavaScript class with the element selectors assigned as
public properties and the element manipulation methods as public functions.

```javascript
function pageObjectName() {
  // example of a element
  this.elemenName = element(by.selector('value'));

  // example of a public method
  this.clickElementName = function () {
    this.elementName.click();
  };
};
module.exports = pageObjectName;
```

## How to add a pageObject to your integration test.
Adding a pageObject to your integration test is done in two steps.

1. Importing the pageObject.
2. Creating a new instance of the pageObject (necessary because its a JavaScript
class)

#### importing the pageObject
Importing a pageObject can be done with a single line of code.

```javascript
var pageObjectClass = require('pathToPageObject');
```

After this code is executed the pageObject can be accessed/instantiated through
the `pageObjectClass` variable.

#### Creating a new instance of the pageObject
Because the pageObject is a JavaScript class and not a singleton we need to
instantiate it. This is also done with a single line of code.

``` javascript
  var pageObjectIntance= new pageObjectClass();
```

## how to add mock modules to your pageObject.
Adding mock modules to a pageObject isn't much different from adding it to your
integration test. The only thing we need to keep in mind is that we need to make
the mock module publicly accessible to our test.

```javascript
function pageObjectName() {
  // example of a mock module inside a pageObject
  this.httpBackendMock = function () {
    angular.module('httpBackendMock', ['ngMockE2E', 'yourAngularApp'])
      .run(function ($httpBackend) {
        // backend mocks here
      });
  };
  // rest of pageObject comes here
};
module.exports = pageObjectName;
```

Now the mock module is accessible as a public property of the pageObject class.

## How to access mock modules and elements contained in a pageObject from within
## your integration tests.

```javascript

var pageObject = require('./pageObjects/pageObject.js'); // import pageObject.
describe('E2E: awesome integration test', function () {
  var page = new pageObject(); // instanciate pageObject.
  beforeEach(function () {
    browser.addMockModule('httpBackendMock', page.httpBackendMock);//mock module
    page.loadPage(); // call method on pageObject
  });

  afterEach(function () {
    browser.clearMockModules();
  });

  it('should do stuff', function () {
    page.button.click();
    expect(page.elementX.isDisplayed()).toBe(true); // expect on element
  });

```
As you can see I repeated some concepts discussed earlier like importing the
pageObject and instantiating it. Here are some other examples as to how to
access various methods and elements inside the pageObject.

#### adding a mock Module contained in a pageObject
```javascript
browser.addMockModule('httpBackendMock', page.httpBackendMock); //mock module
```

#### calling a method defined in a pageObject:
```javascript
page.loadPage(); // call method on pageObject
```

#### doing a expectation on a element contained in a pageObject:
```javascript
expect(page.elementX.isDisplayed()).toBe(true); // expect on element
```

## example case
In this section I will show a login E2E test  that is refactored using
pageObjects first I will show you the original code then the refactored version.

### Original E2E test
```javascript
describe('E2E: Testing Login Controller', function () {
  var httpBackendMock = function () {
    angular.module('httpBackendMock', ['ngMockE2E', 'yourAngularApp'])
      .run(function ($httpBackend) {
        var mockUser = {
          'first_name': 'River',
          'last_name': 'Kovacek',
          'email': 'beatrice@stroman.name',
          'auth_token': 'A1yP279QRfyacspCz9se',
          'user_role': 'user',
          'group_ids': []
        };

        // mock the API requests
        $httpBackend.whenPOST('http://backendUrl/api/v1/sign_in').respond(
          200,
          mockUser
        );
        $httpBackend.whenDelete('http://backendUrl/api/v1/sign_out').respond(
          200,
          mockUser
        );
      });
  };

  beforeEach(function () {
    browser.addMockModule('httpBackendMock', page.httpBackendMock);
    browser.get('/login')
  });

  afterEach(function () {
    browser.clearMockModules();
  });

  it('should not log in without credentials', function () {
    // set username field
    element(by.model('email')).clear();
    element(by.model('email')).sendKeys('');

    // set password field
    element(by.model('password')).clear();
    element(by.model('password')).sendKeys('');

    element(by.css('[ng-click="login()"]')).click().then(function () {
      browser.waitForAngular();
    }); //click login button

    expect(browser.getCurrentUrl()).toEqual(browser.baseUrl + '/login');
    expect(element(by.className('alert')).isDisplayed()).toBe(true);
  });

  it('should log in with credentials', function () {
    // set username field
    element(by.model('email')).clear();
    element(by.model('email')).sendKeys('beatrice@stroman.name');

    // set password field
    element(by.model('password')).clear();
    element(by.model('password')).sendKeys('welkom');

    element(by.css('[ng-click="login()"]')).click().then(function () {
      browser.waitForAngular();
    }); //click login button

    expect(browser.getCurrentUrl()).toEqual(browser.baseUrl + '/menu');
    expect(element(by.className('alert')).isDisplayed()).toBe(false);
  });

});
```



#### login E2E test after refactoring:
```javascript
describe('E2E: Testing Login Controller', function () {
  var loginPage = new login();
  var menuPage = new menu();

  beforeEach(function () {
    browser.addMockModule('httpBackendMock', page.httpBackendMock);
    loginPage.loadPage();
  });

  afterEach(function () {
    browser.clearMockModules();
  });

  it('should not log in without credentials', function () {
    loginPage.setUsername('');
    loginPage.setPassword('');
    loginPage.clickLoginBtn();
    expect(loginPage.isLoaded()).toBe(true);
    expect(loginPage.getErrorMessage().isDisplayed()).toBe(true);
  });

  it('should log in with credentials', function () {
    loginPage.setUsername('beatrice@stroman.name');
    loginPage.setPassword('welkom');
    loginPage.clickLoginBtn();

    expect(menuPage.isLoaded()).toBe(true);
    expect(loginPage.getErrorMessage().isDisplayed()).toBe(false);
    menuPage.clickLogoutLink();
  });

}
```

#### login pageObject:
```javascript
'use strict';

function loginPage() {
  this.httpBackendMock = function () {
    angular.module('httpBackendMock', ['ngMockE2E', 'yourAngularApp'])
      .run(function ($httpBackend) {
        var mockUser = {
          'first_name': 'River',
          'last_name': 'Kovacek',
          'email': 'beatrice@stroman.name',
          'auth_token': 'A1yP279QRfyacspCz9se',
          'user_role': 'user',
          'group_ids': []
        };

        // mock the API requests
        $httpBackend.whenPOST('http://backendUrl/api/v1/sign_in').respond(
          200,
          mockUser
        );
        $httpBackend.whenDelete('http://backendUrl/api/v1/sign_out').respond(
          200,
          mockUser
        );
      });
  };

  this.loginBtn = element(by.css('[ng-click="login()"]'));
  this.username = element(by.model('email'));
  this.password = element(by.model('password'));

  this.loadPage = function () {
    browser.get('/login');
  };

  this.isLoaded = function () {
    return browser.getCurrentUrl().then(function (url) {
      return url === browser.baseUrl + '/login';
    });
  };

  this.clickLoginBtn = function () {
    this.loginBtn.click().then(function () {
      browser.waitForAngular();
    });
  };

  this.setUsername = function (value) {
    this.username.clear();
    this.username.sendKeys(value);
  };

  this.setPassword = function (value) {
    this.password.clear();
    this.password.sendKeys(value);
  };
}

module.exports = loginPage;
```

