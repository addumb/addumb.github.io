---
layout: post
title: Mobile App Operability
date: 2020-07-15 00:00:00.000000000 -07:00
type: post
published: true
status: publish
description: Mobile app operability is the trait of mobile apps which makes detection, diagnosis, remediation, and anticipation of errors low-cost and low-effort.
keywords: mobile operations, mobile operability, graceful degradation, mobile disasters
categories:
- Tech
tags:
- Tech
author:
  login: addumb
  email: adam@addumb.com
  display_name: addumb
  first_name: 'Adam'
  last_name: 'Gray'
---

Mobile apps often get a bad rep from backend infrastructure people. Rightly so when comparing operability between backend infrastructure services and mobile applications. First, so that we're on the same page: operability is trait of a software system which makes detection,  remediation, and anticipation of errors is low-effort. What is this for mobile applications? Well... it's the trait of a mobile application which makes detection, remediation, and anticipation of errors is low-effort, of course. Mobile apps are software systems. They're usually best modeled as trivially parallelized distributed software systems which just happen to run on mobile devices rather than on owned infrastructure.

***Mobile operability is the trait of mobile apps which makes detection, diagnosis, remediation, and anticipation of errors low-cost and low-effort.***

Errors here can be anything unwanted: crashes, freezes, error messages, lacking correct responses to interactions, and even having incorrect responses to interactions.

Detection
=========

To consider what good operability is for mobile apps, we need a way to detect these errors. To detect errors, we have to record them and deliver that record to a detection system. That system does not have to be outside of the app! In fact, it's best if your app has a self-diagnostic mechanism built-in to help understand errors in a way that respects your users' privacy. Common detection mechanisms are crashlytics, the Google Play console Vitals reports, and app reviews and star ratings. The app usage analytics system should also be used for collecting errors, though you should use a separate analytics aggregation/processing method here to ensure your error data has as little user attributes as possible. Why not use the same one? Well with normal app analytics, you're subject to GDPR, deletion requests, and heightened security expectations. Errors shouldn't be like this, errors should be pared down so far that you can safely and securely retain them forever.

Detection is focused on efficiently detecting occurrences of errors, not making them easy to debug.

Each error category may have different and multiple avenues of diagnosis. Some likely have none at all, which is why I'm writing this. There are three general mechanisms of delivering errors:

1.  3rd party: things which you don't own, manage, nor control like Google Play console Vitals, app store reviews, star ratings, etc.

2.  Out-of-band: your app aggregates, interprets, and potentially sends error details to a separate backend system than what your app primarily interacts with. Common examples are analytics systems, custom crash interpreters, and local error aggregation.

3.  In-band: your app includes some details of errors within the primary client/server RPC, your server may include some global error information within the primary RPC's as well.

Diagnosis
=========

There is a middle ground between zero context and debug-level context. You need to know the stack trace, view hierarchy, and some information about the device and user which stand-out to help diagnose a problem. You can quickly detect that a crash happened, but diagnosing  why may require far more insight. The middle ground is in the aggregable context of an error: stack trace, activity or view, device type, etc. You should not require a full core dump for understanding that the app crashed, that's for understanding why.

Diagnosis requires great context for the error. This context can come from a few different places:

1.  The app itself: it can submit an error report to you.

2.  The backend(s): maybe your web servers sent an HTTP 500 response which caused the user-visible error.

3.  The session, user, or customer: if you can determine which account experienced a specific error, you may be able to reconstruct context from their user data.

Ever-increasing context is not the path to perfect diagnosis, there should always be a trade-off between increasing context and value to the user. If you want to collect full core dumps, remember that transmitting 4GB+ of data over a cell network can bankrupt some people, so don't do it. Adding the device manufacturer or even device model to the error report may give you critical context for some problems.

Every error may be different. There's no magical set of fields to supply in an error to give sufficient context to efficiently diagnose. You, as the app developer, should have the best idea of what pieces of context will be most useful in each subsystem of your app. For example, if you're implementing the password change flow, you may want to include metadata about the account's password: how old was it, how many times has this person changed it recently, or even how they arrived at the screen if you have multiple paths like an email deep link.

Anticipation
============

There are many scenarios where a quantity in the application may be increasing or decreasing toward a critical threshold: resident set size, battery power, CPU cycles, cloud storage quota, etc. There are a few strategies to know if these are about to produce an error:

-   High/low water marks

-   Quotas

-   Correlational estimates: flow complexity, runtime, user sophistication, and others I have yet to learn or hear about.

High/low watermarks: if your application has a finite amount of a resource, you should implement it so that it operates optimally under a low amount of the resources. Once it reaches a threshold of usage of the resource, it can start disabling or degrading functionality. It's common to implement at least two thresholds here: high and low watermarks. If the low watermark is reached, the app can start to refuse new allocations into the resource pool, or delay unnecessary work. The app should still generally be functioning as expected, though maybe a bit slower. Upon reaching the high water mark, the app should outright disable functionality and evict low priority resource usage.

Example: Let's say an app takes, uploads, and views images. One of the resources it's sure to use is local disk: to cache captured images, cache or even synchronize downloaded images, and to cache different image resolutions or image effect applications. If we set a low watermark of 1GB local storage, the app can switch to a mode where it does all processing on only previously allocated image files. This prevents some increase in storage used. However, if we hit a high watermark, we could have the app actively de-allocate images in storage and force re-fetching, in-memory processing, or even decreased resolution or effect support.

Quotas: these follow a similar concept as watermarks, but they have an inverse usage expectation. Nothing should change in behavior until the quota is met, at which point there can be a user flow for increasing the quota or decreasing the usage.

Correlational estimates: try not to use these. If a critical error occurs, once the dust settles and everything is recovered, it's common to wonder if you could have seen this coming. One common question in the blameless post-mortem is "Was there a leading indicator which could have predicted this issue?" This is grasping at straws. If your application is so complex that there is no direct indicator which you can add, nor but you can fix, you're already in too deep without proper operability. Push back against adding an alert on a correlational leading indicator. Instead, try to reduce the complexity of the app around the failure.

Remediation
===========

If we've detected and diagnosed an error, how do we fix it? This is where things get fun :) The general idea is to use everything at your disposal! The well-trod areas are:

-   Backend remediation

-   In-app feature flags

-   Hotfix

I'd like to urge you to consider another: a failover app wrapper. The failover wrapper is not just a blanket exception handler, it's as separated as possible from the icky, risky, and most problematic part of your app: your code (mine included!). There are many ways to go about this and none of them are sufficient, otherwise it would be a fail-safe. Nothing in your application's logic should be able to get into such a bad state that the best recourse available is to harm the core user experience. You can implement this as a stale view of data in a separate process, a babysitter process which spawns your main application's surfaces, but the best available options are indeed to use a separate process. Within your main application's logic, you can have a mechanism of "crashing" which only exits the child process, not the wrapper which is still able to provide some useful functionality to the user. The IPC between the processes to synchronize the user's state could be done all sorts of ways, but again needs to be overly simple so that the failover wrapper doesn't collect the same bugs as the main app.

Backend remediation is what we end up with when we don't plan for emergencies as we write the app. The backend sends some special configuration telling the app to disable something, or sends response which was carefully crafted to side-step the problematic app code. You'll end up with some of these, and that's a good thing.

In-app feature flags deserve a whole separate post. I'll also split them out into two categories: killswitches and dials. The general idea here is to implement graceful degradation within the app. Similar to the idea above about high/low watermarks, if the app encounters some critical error, the app can have a mechanism to reduce or remove functionality. One simple example is to crash when detecting a privacy error. Rather than show any activity with a privacy problem, just exit the app. This is the bare minimum, though, because how can you fix the privacy problem and stop the app from crashing?

Killswitches: these are simple boolean indicators which the app checks any time it exercises some specific functionality. The functionality is skipped if the killswitch is on. Your implementation may vary in the use of true vs. false to indicate on or off, but make it consistent. These parameters should be delivered through an out-of-band RPC to configure your application, ideally changing while the app is running. Distributing killswitches is a fun distributed systems problem with many complex solutions, just be sure that you're comfortable with the coverage and timeframe of your solution, be it polling, pushing, swarming, or other fancier options I need to learn about.

Warning about killswitches: if you have killswitches for very small pieces of your application, you will end up exercising code which is behind tens or hundreds of conditional killswitch checks. Nobody ever tests every combination of killswitches, because they are killswitches. Use A/B testing to control the on/off choice of these smaller, less disaster-prone surfaces of your application. Killswitches should be reserved for disabling large pieces of functionality.

Dials: some functionality in your application can scale up and down in demand or resource usage, both on the client and on the backend. In the example of watermarks above, image resolution could be a dial from the client to reduce resource usage on the client or on the server. Video resolution, time between polls, push batching, and staleness of data are all examples of dials which you can use to remediate huge categories of errors if you support dialing to 0.

Hotfix: you can always ship a new build of your app with the offending bug fixed. This takes a long time and may take a whole team of people to produce. Even then, you're at the mercy of the app stores, which may take days or even *weeks* to finally push your update. Just be sure only to do a true hotfix: fix the code from the very commit used to build the affected version.

The Perfect App
===============

Remember: "The perfect is the enemy of the good." Having a product is more important than having a perfectly operable product. The flip-side also holds: having an operable product is more important than having a perfect product. Don't go all-in on either, exercise the balance with moderation. Learn from your mistakes. Just as a competitor is about to eat your lunch by getting to market sooner, another competitor may swoop in when you inevitably have a disaster and don't have the operational capability to recover.
