---
layout: post
title:  12 Factor Apps (Part 1)
date:   2023-06-26
description: 
tags: C# til TwelveFactor
categories: C#
---

A colleague of mine has brought up [12 Factor Apps](https://12factor.net/) in various discussions and posited that every modern developer should understand and implement ideas from this manifesto. This article will serve as a monologue on my learnings as I work through the ideas presented on the website.

## 1. Codebase - One codebase tracked in revision control, many deploys
> A twelve-factor app is always tracked in a version control system, such as Git

Fairly standard advice, I would definitely not want to code in an professional setting where git or something similar was not in use - that would be wild.

> one-to-one correlation between the codebase and the app. 
>
>If there are multiple codebases, it’s not an app, it's a distributed system

This makes sense. However, I do wonder if the exception to this example is when class libraries are built in a separate code base in order to publish them as nuget packages. They'd technically be used for a single app, but managed separately for the purposes of versioning.

>Multiple apps sharing the same code is a violation of twelve-factor. The solution here is to factor shared code into libraries which can be included through the dependency manager.

My understanding of this is that they're saying the DRY principle for multiple code bases is bad if it couples them  and breaking changes to the shared could break one or both the apps. The dependency manager is a catch for this because have a dependency manager means that the code can be versioned and applications can be deployed independently of what the other application is on.

>A deploy is a running instance of the app. This is typically a production site, and one or more staging sites. Additionally, every developer has a copy of the app running in their local development environment, each of which also qualifies as a deploy.

Makes complete sense

## 2. Dependencies - Explicitly declare and isolate dependencies

> A twelve-factor app never relies on implicit existence of system-wide packages. It declares all dependencies, completely and exactly, via a dependency declaration manifest

YES! Absolutely agree with this - there is nothing worse than a dependency being "Automagically" resolved for you. I used to love this sort of thing when I started developing but have steadily begun to hate it. 

You can never guarantee that those dependencies will always be there by default on the system you're working and it prevents hours of debugging and having the standard "It works on my local" discussion. Furthermore, the moment you start deploying code into pipelines and build agents, using the implicit approach means you're at the mercy of whatever changes your provider decides to make. 

In pursuit of eliminating this "voodoo magic", in any new C# application I will always disable implicit usings in the csproj file.

## 3. Config - Store config in the environment
>Apps sometimes store config as constants in the code. This is a violation of twelve-factor, which requires strict separation of config from code. Config varies substantially across deploys, code does not.

100% Agree with this - not to mention that having config details in code is a surefire way to accidentally expose secrets and have your PM running after you because a Snyk report catches you out.

> Another approach to config is the use of config files which are not checked into revision control

>Note that this definition of “config” does not include internal application config ... This type of config does not vary between deploys

It makes sene that one isn't able to check in config if that config is deployment specific information (like an api endpoint to use) or some secret. In previous projects I've use the `appsettings.json` to indicate that serilog should be logging to console and file. In other applications I've also had serilog logging to a Seq sink which may be deployment specific if you're deploying out copies of an application. I guess teasing the two apart is implementation specific, but agree with everything in principle.

> The twelve-factor app stores config in environment variables

> unlike config files, there is little chance of them being checked into the code repo accidentally; and unlike custom config files

Every dev who's done any kind of meaningful work has accidentally checked in a config file they weren't meant to - I like this idea in principle though!

## 4. Backing services - Treat backing services as attached resources

> A backing service is any service the app consumes over the network as part of its normal operation.

An alternative example to the one's they give is a good old Azure Function App

> The code for a twelve-factor app makes no distinction between local and third party services. To the app, both are attached resources, accessed via a URL or other locator/credentials stored in the config. 

Very much in favour of the loose coupling principle and being able to spin up new instances of the resource as needed.

Also worth noting is the "without code changes" which supports (3) in that no config is kept in code.

## 5. Build, release, run - Strictly separate build and run stages

I do wish they had added in a bit more here about the various ways in which the three stages could overlap. For example, you can't replace the dll of a running application (on windows) so the principle of splitting it up is enforced.

The versioning of releases is a must though

## 6. Processes

> Twelve-factor processes are stateless and share-nothing

> The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job

Agree - a good ol system restart or VM redeploy (with non-persisted hard drives) would muck things up. Treating these things as ephemeral really helps for me.

> Some web systems rely on “sticky sessions” – that is, caching user session data in memory of the app’s process and expecting future requests from the same visitor to be routed to the same process. Sticky sessions are a violation of twelve-factor and should never be used or relied upon. Session state data is a good candidate for a datastore that offers time-expiration, such as Memcached or Redis.

This one caught me red handed - I've done this in the past when keeping objects in memory as they were very expensive to deserialize from the DB. What I do like about this is it encourages systems with TTL (time to live) contracts so the management of these objects in memory is handled for you.


## Conclusion
The first 6 points of the 12 factor application is very interesting. Particular points that stood out to me was how to use environment variables over config files as well as the last point on using something like Redis or Memcache. In the next article I'll go through the next 6 items on the list.

## Additional notes
- The twelve factors were written by developers at Heroku. More information is available on [wikipedia](https://en.wikipedia.org/wiki/Twelve-Factor_App_methodology)