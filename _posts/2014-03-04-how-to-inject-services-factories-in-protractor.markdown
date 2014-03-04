---
layout: post
title: "How to inject services/factories in protractor"
date: 2014-03-04 10:56:37
author: alexander
categories: [Angular,Rails]
---

## Why would I want to inject services/factories in my E2E tests?
Sometimes you want to exclude some modules or override them.
Another reason would be because you want to bypass a process by calling
service methods directly. The later is faster then navigating to a page and 
mocking the API request.

## how to inject a service inside a mock module
This is often done to bypass API requests and manipulate services directly by
calling methods in services to get your application in the preferred state.

```javascript
var moduleScript = function () {
    angular.module('moduleName', ['ngMockE2E', 'yourAngularApp'])
      .run(function (serviceToInjectOne, serviceToInjectTwo) {
        //you have access to serviceToInject one and two now.
      };
```

## example case
In this example I will show you an example we've used to bypass the
authentication process in order to test pages that required an authentication
token.

The service we want to mock contains two methods we are interested in:

1. login()
2. logout()

#### Login method:

```javascript
this.login = function (user, success, error) {
  endPoint.all('sign_in').post(user)
    .then(function (response) {
      updateCurrentUser(response);
      success(currentUser);
    }, function (response) {
      if (response.status === 422) {
        error(Tools.parseError2Message(response.data));
      } else {
        error(response.data);
      }
    });
};
```


#### Logout method:

```javascript
this.logout = function (success, error) {
  if (!currentUser.email) {
    success();
    return;
  }
  endPoint.one('sign_out').remove()
    .then(function (response) {
        removeUser();
        success();
      },
      function (response) {
        removeUser();
        if (response.status === 422) {
          error(Tools.parseError2Message(response.data));
        } else {
          error(response.data);
        }
      });
};
```

As you can see there are two API request we need to mock:

- sign\_in
- sign\_out

After we mock those two API requests we need to inject the Auth service and call
the login and logout methods respectively.

## step 1: create a backend mock for the API request that takes place when we
## try to login/logout.

```javascript
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
```

## step 2: create a module where we call the login method of the Auth service

```javascript
var createSession = function () {
  angular.module('createSession', ['ngMockE2E', 'yourAngularApp'])
    .run(function (Auth) {
      Auth.login({
          email: 'beatrice@stroman.name',
          password: 'welkom123'
        },
        function successCallBack() {},
        function errorCallBack() {}
      );
    });
};
```

Now we have two modules one that intercepts the login request and one
that calls the login method from within the Auth service. This removes the need
to navigate to the login page fill in the login form and wait for the request to
fire off.

We're removing the middlemen and force the Auth service to do the
request immediately.

## step 3: create a separate login function where we add the module made in step
## 2;

```javascript
var login = function(){
  browser.addMockModule('createSession',createSession);
  browser.removeMockModule('createSession');
}
```

After we added the module we are logged in, so we can remove the module
immediately afterwards.

## step 4: create a module where we call the logout method of the Auth service.

```javascript
var destroySession = function () {
  angular.module('destroySession', ['ngMockE2E', 'yourAngularApp'])
    .run(function (Auth) {
      Auth.logout(
        function successCallBack() {},
        function errorCallBack() {}
      );
    });
};
```

This module is quite similar to the login one the only difference is that we
call the `Auth.logout()` method instead of the login method.

## step 5: create a separate logout function where we add the module made in
## step 4

```javascript
var logout = function(){
  browser.addMockModule('destroySession',destroySession);
  browser.removeMockModule('createSession');
}
```

Now we can call the login() and logout() methods from within our tests to mock
the login process.
