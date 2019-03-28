---
layout: post
title: "How to mock API requests with ngMockE2E and protractor."
date: 2014-03-02 17:05:00
author: alexander
categories: [protractor]
---

In this blog post I will cover how you can mock API/backend requests.

##1. Introduction to the ngMockE2E library.
The AngularJs docs states the following about the ngMockE2E module:
>The ngMockE2E is an angular module which contains mocks suitable for
>end-to-end testing. Currently there is only one mock present in this module
>the e2e `$httpBackend` mock.

The `$httpBackend` mock is a service inside the ngMockE2E module. the
`$httpBackend` mock allows us to do a Fake HTTP backend implementation suitable
for end-to-end testing or backend-less development of applications that use the
$http service.

There are a few differences between the `$httpBackend` that you might be
familiar with while unit-testing.

In an end-to-end testing scenario it is ofter desirable for certain category of
requests to bypass the mock and issue a real http request. 

Additionally we don't want to manually have to flush mocked out requests like we
do during unit-testing. For this reason the EOE `$httpBackend` automatically
flushes mocked out requests, closely simulating the behaviour of the
`XMLHttpRequest` object.

##2. How to mock different types of API requests.
The `$httpBackend` service has several ways to define what endpoint you want to
mock and how you respond to this request. In the examples below I will explain
these methods for each type of HTTP request.

### Respond to GET request on specific endpoint

#### Case #1: respond with a anonymous object.
```javascript
$httpBackend.whenGET('/users/1').respond({
  firstName: 'John',
  lastName: 'Johnson'
});
```

#### Case #2: define a respond object beforehand and respond with that object.
```javascript
var userObject = {
  firstName: 'John',
  lastName: 'Johnson'
};

$httpBackend.whenGET('/users/1').respond(userObject);
```

#### Case #3: define a collection beforehand and respond with a specific item.
```javascript
var users = [
  {
    firstName: 'John',
    lastName: 'Johnson'
  },
  {
    firstName: 'Bob',
    lastName: 'Johnson'
  }
];

$httpBackend.whenGET('/users/latest').respond(function(method,url,data,headers){
  return [200, users[users.length-1], {}];
});
```

### Respond to POST request on specific endpoint.

#### Case #1: add POST data to collection.
this can be used in situations/pages where you post data to the API and later on
want to verify if the posted data is stored in a database.

```javascript
var users = [];
$httpBackend.whenPOST('/users/').respond(function(method,url,data,headers){
  users.push(angular.fromJson(data))
});
```

#### Case #2: respond with a copy of the POST data.
This method is often used because it allows us to return/mirror the data send to
us. Sometimes API'S return the data object send to them when a new account is
created for example.

```javascript
$httpBackend.whenPOST('/users/').respond(function(method,url,data,headers){
  return [200, data, {}];
});
```

### Respond to POST request on specific endpoint with specific data.
```javascript
var userObject = {
  firstName: 'John',
  lastName: 'Johnson'
};

$httpBackend.when('POST','/users/',userObject).respond(function(method,url,data,headers){
  return [200, {message: 'you wanted to know more about John?'}, {}];
});
```

### Respond to POST request on various endpoints by using regular expressions
I use this method often if I don't want to pass through the request and want to
return an authentication token to prevent 401 errors from logging me out.

```javascript
$httpBackend.whenPOST(/^(https?:\/\/)?(backendUrl\.com)([\/\w\.-]*)*\/?$/).respond({
  'auth_token': 'A1yP279QRfyacspCz9se',
})
```

### Respond to POST request on all endpoints by using regular expressions
This is not always a great way to handle things but sometimes pages do allot of
requests to external libraries for social media integration and if your not
testing those then it's safe to just mock them.

Remember that specific endpoint backends will overrule this global backend
definition.

```javascript
$httpBackend.whenPOST(/^\w+.*/).respond({
  'message': 'this will always happen',
});
```

##3. How to let certain requests pass through.
Often you want to pass through certain specific endpoints or match certain type
of endpoints by means of a regular expression, because you need them to touch
the backend in order for the page to function correctly.

### Case #1: Pass through POST request on all endpoints
All requests that aren't mocked will be passed through, however in my opinion it
improves readability to still declare a backend mock that explicitly passes
through all requests that aren't mocked.

```javascript
$httpBackend.whenPOST(/^\w+.*/).passThrough();
```

### Case #2: Pass through GET request to views/templates with regexp.

```javascript
// Don't mock the html views
$httpBackend.whenGET(/views\/\w+.*/).passThrough();

// Don't mock the html templates
$httpBackend.whenGET(/templates\/\w+.*/).passThrough();
```

##4. How to define mock modules.
Declaring a mock module consists of two steps:
1. declaring a function which will contain the angular.module
2. declaring a angular module with a unique name that injects ngMockE2E + your
   angular app.

```javascript
var httpBackendMock = function () {
  angular.module('httpBackendMock', ['ngMockE2E', 'nameOfYourAngularApp'])
    .run(function ($httpBackend) {
      //http backend mocks here
    });
};
```

There are better and cleaner ways to define and access mock modules with
page-Objects. I'll write a blog post about this topic soon.

##5. How to add mock modules to your E2E test.
There are three steps/states in the lifecycle of a mock module:

1. defining the mock module.
2. adding the mock module to the protractor instance.
3. clearing all mock modules when your done.

#### adding the mock module to the protractor instance.
```javascript
browser.addMockModule('nameOfMockModule', nameOfFunctionContainingModule);
```

####clearing all mock modules of the current protractor instance.
```javascript
browser.clearMockModules();
```

##6. Example case: mock registration API request.
I will give a real example of a mocked API request we're currently using in a
project .

I will walk you through the following steps:

1. lookup the required format and endpoint of the request for the registration
   process.
2. define the mock module
3. add the mock module to a beforeEach block.
4. add the browser.clearMockModules to the afterEach block.

### step 1: API docs lookup
before we know what API requests to mock we need to read through the API
documentation and see how we can register a user.

![image](http://f.cl.ly/items/3b0P0v1B1X0a1K1Z1R2G/Screen%20Shot%202014-02-12%20at%2001.45.51.png)

This gives us the endpoint and the API response object.

### step 2: define the mock module:
If your API documentation is well written then in most cases defining the mock
module is just a Mather of copy pasting the API endpoint and response body.

```javascript
var httpBackendMock = function () {
  angular.module('httpBackendMock', ['ngMockE2E', '360FeedbackApp'])
    .run(function ($httpBackend) {
      $httpBackend.whenPOST('https://apiDomain.herokuapp.com/api/v1/users')
        .respond({
          'message': 'A confirmation email has been sent to beulah@lynch.info',
          'user': {
            'id': 1,
            'first_name': 'John',
            'last_name': 'Doe',
            'role': 'user',
            'email': 'beulah@lynch.info',
            'created_at': '2014-01-13T12:53:04.484Z',
            'updated_at': '2014-01-13T12:53:04.484Z',
            'invitation_id': null
          }
        });
    });
};
```

### step 3: add the mock module to a beforeEach block.
In this step we make sure that the mock module is added before each test is run.
```javascript
beforeEach(function () {
  browser.addMockModule('httpBackendMock', httpBackendMock);
});
```

### step 4: clear mock modules in a afterEach block.
```javascript
afterEach(function () {
  browser.clearMockModules();
});
```
