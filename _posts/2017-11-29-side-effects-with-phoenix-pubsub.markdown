---
layout: post
title: "Side effects with Phoenix PubSub"
date: 2017-11-29 12:19:40
author: Marcel
categories: []
---

# How we used Phoenis.Pubsub to do our side effects

In a normal cycle of a request we do a couple of things before we show something to the user.
We receive a CRUD request we do something on the database and return the data within the page that should
be shown to the user. All of this takes some time before the users gets shown that what he did succeeded.

In some requests there are side effects. What i see as a side request is everything in the request
that has been done that the user doesn't directly care for to know that something succeeded. If a user
creates an event and adds users there may occure multiple side effects. For example:
- Audit logger
- Activity (for an activity dashboard)
- Emails to all the users that are added
- Analytics
- External api calls

All these things might occure but the only thing the user wants to see is if he entered the correct
data to create the event.
