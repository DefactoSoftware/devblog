---
layout: post
title: "Benefits of Protractor in a Angular+Rails API application stack"
date: 2014-02-22 18:13:16
author: alexander
categories: [Angular,Rails]
---

## AngularJs with a separate Rails API
We just finished a Web application that facilitates in the process of 360
feedback. During the development process we learned a lot about how to use
Angular alongside a Rails API especially when it comes to testing.
With this blog we want to share some of our insights.

### Why we chose to split the Angular frontend from the Rails app.
There were several reasons why we chose for the Angular + Rails API solution.

1. We had some developers that were really familiar with Rails, and some
   developers that were less familiar with Rails.
2. We wanted to develop a Ios app aswell and not having to write a separate API
   for the Ios app saves a lot of development time.
3. Rails has a fast development time, is highely cachable and lends itself
   really well for API's.

By splitting the development of the frontend and the Rails backend we could
distribute the work over two development teams and keep the application as a
whole very extensible.

### The downsides of our approach
By splitting the application over different repositories you gain flexibility
but that comes at a cost. <a href="https://twitter.com/drapergeek">Jason
Draper</a> recently wrote an article about
<a href="http://robots.thoughtbot.com/emberjs-with-a-separate-rails-api">
EmberJs with a separate Rails API
</a>
In this article he writes the following about his experience with a similar
approach:
>The other problem is that by having the applications in two seperate repos, you
>can't easily have a full integration test. In order to do this you would need
>to run both applications on the same machine simultaneously. You would also
>need some way to make sure that both of the applications were on the same
>revision for the test.

We came to the same conclusion when we implemented E2E tests. the solution we
chose is the same as the one Jason's team chose:

>We decided to test the apps seperately and trust that the API is what we've
>said it was. This can be frustrating but in our case, it didn't turn out to be
>a real problem. Once you have wired your API to the UI, you should never change
>your UI without also changing the API. This was enforced in code review only.

In my opinion this is the best approach when you split your frontend and backend
however not everyone shares that opinion. In this blogpost i will cover why we
believe that testing the apps seperatly and thus mocking the API requests is the
way to go.

## why mocking API requests in E2E test is OKAY and dummy endpoints in the backend are NOT.
It is common belief that end to end (E2E) tests should cover the whole application stack. In other words only covering the frontend aspects of the
application and mocking the backend is evil and you shouldn't do it.

During my research on how to implement protractor and mock several API requests in the process,
I came across several forum/blog/github posts that looked like this:

>First of all the goal of integration testing is to test everything. Therefore mocking the backend in this case isn't a good idea.
>From my perspective the real problem with testing in protractor (and with any other tool for integration/browser testing)
>is how to load database fixtures or how to setup the initial state of the app.
>
>In ruby world we have excellent capybara gem which can directly hook into the database but in the javascript world things are much more complicated.
>Especially when the backend is written in the other language, framework, etc.
>
>In my sample project I workaround this problem with dummy API endpoint for loading db fixtures
>(obviously it should be exposed only in the test environment).
>Inside the integration test I'm using this endpoint to load fixtures in order to ensure that each test has the same initial state.

So the opinion from this developer is that instead of mocking the backend/API requests you should aim to set a initial state of a test database.
Although this is a valid solution it has a lot of downsides.

1. Setting the initial state of a test database requires a lot of database configuration.
2. The state of the database has to be reset to different states depending on the behaviour and page you want to test.
3. hooking into a database with pure Javascript is next to impossible and exceeds the language's design and purpose.
4. Dummy API endpoints for loading database fixtures is a nasty solution that pollutes the backend.

The solution from this developer is one I really don't like his method requires a lot of database setup/configuration and forces us to change the
backend by adding extra endpoints just to make tests that focus on various aspects of the user interface work. In my opinion the backend is the worst
place to pollute in this instance. You should aim to keep frontend and user-interface oriented tests and code in the frontend.

## why do we need to intercept / reroute API requests anyway..
The problem is that in order to test various parts of the frontend you come across several pages that contain API requests that cause errors or
pollute your production/development database.

In order to prevent this from happening we need to either reroute the request to another endpoint or intercept the API request and respond with a JSON
mock as response.

## why mocking API requests is the preferred solution
mocking the API requests has several advantages over creating separate endpoints.
1. No need to have a internet connection or rails server running.
2. Its faster to intercept the request and return a JSON object then communicate with the backend.
3. It allows you to keep frontend and backend code more separated.
4. more flexibility what requests to pass through and which ones to intercept.
5. works out of the box no additional database configuration needed.

## if we only cover the front-end aspects of the application stack is it still worth it to write integration tests ?
YES, the main purpose of integration tests is to test how our application behaves when it's being used by a real user. The focus of integration tests lies on testing
the behaviour/workflow of our frontend when a user is doing certain tasks on certain pages.

Many developers want to test too much in their integration tests we don't want to test API requests or application logic.
We only want to know if the application behaves as we expect it does in certain scenarios.

### Things you should test..
1. If I leave certain required fields empty does a error message appear and do I stay on the same page?
2. If I fill all required fields, do i get redirected to page X ?
3. If I navigate to a resource that is only accessible when logged in, do I get redirected if I'm not ?
 
### Things you should avoid testing..
1. If I login with a mock user object do i recieve a request from the backend containing the right mocked authentication token ?
2. If I send a request to change my password do i get an error message if my request contains an invalid current password field ?
