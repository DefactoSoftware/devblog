---
layout: post
title: "What We Learned From Opensourcing Our Pet Project"
date: 2014-08-11 16:25:27
author: hugo
categories: [open source]
---

Last week we [opensourced](https://news.ycombinator.com/item?id=8151777) our internal pet project [Hours](https://github.com/DefactoSoftware/Hours). Hours is a very simple timetracking app that was conceived in a secret location one morning because we were fed-up with the software we were using.

Our guiding principles while developing Hours were:

* It should be extremely easy to make entries.
* There should be as little limitations as possible (stuff like: only managers can create projects and assign users to projects, etc.).
* Everyone should be able to signal how projects are going, or which a colleagues have disappeared down the rabbit-hole.
* We won't mindlessly add features just because people are used to something.


<img src="http://i.imgur.com/L6cCxPd.png" width=500 alt="Projects overview" />
> We've been using _Hours_ internally for a couple of months now.

After using Hours internally for a while, we decided it would be cool to clean it up a little and [make it available](https://happyhours.io) for others - both the source as a hosted version (free for small teams).

# So here's what we learned
--

1. __You have to be able to stand heavy criticism__. The internet always knows better - and they aren't always nice about it. So yeah, we got slammed for our onboarding experience, for missing features, for not matching the clock in our [favicon](https://happyhours.io/assets/favicon-cda301768f23447b0684fc792f77e140.ico) to our [logo](https://happyhours.io/assets/logo-white-shadow-e39a142fa42f9f5985e44b06ec6cf432.png). And in all of those cases (and a bunch of others), we could only agree.

2. __Don't be afraid__. You can never think of everything before opensourcing your project, and even if you would, people are still gonna find things that are sub-par. The criticism may be harsh, but above all it's amazingly __valuable__. There is also something different about comments from random people around the world, compared to listening to the same old colleagues nagging about user experience ;). The criticism motivated us like nothing else and within a couple of hours, we adressed some of the major issues.

3. __Opensourcing a project is inspiring for developers__. Our company mainly operates in the business-to-business space, and needless to say there are moments when we feel our developers aren't getting the appreciation they deserve. To open source a project means playing for the most tech-minded audience there is - which can be very exiting. And isn't being on githubs list of trending developers along with the likes of google and facebook for a day our 15 minutes of fame?

4. __Documentation is key__. We thought we were pretty anal about our documentation, but as soon as we made our repository public we realized (and heard) we forgot to add one or two key facts to the readme and docs. Ouch.

5. __Someone had an [awesome internship]({{ site.url }}/images/intern.png) at our company in 1991__.

All in all, our experience was very positive. Should we think about opensource all our projects?

And what would that mean with regard to having a sustainable business?
