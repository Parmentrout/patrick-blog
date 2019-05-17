---
title: 'Angular Virtual Scroll'
description: 'Setting up virtual scroll with server side pagination'
author:
  name: 'Patrick Armentrout'
  desc: ' '
date: 2019-05-17
draft: false
categories:
  - tutorials
---

#Purpose

A cool thing that came out with the release of Angular 7 was the cdk virtual scroll!  Think Pinterest, as your user scrolls down on the page new data is constantly loaded for them.  Probably not great for their phone addiction, but good for your mobile UI strategy.

Prior to that release, you'd have to use a lot of customized rxjs and screen events for any server side data pagination.

In this article we are going to play around with the cdk and mock some server side virtual scroll goodness.

## Set up

Like all good Angular apps, let's get started with the cli and create our new application:

{{< highlight typescript >}}

// First validation you have the latest Angular cli installed (7.3.something)

ng new virtual-scroll

{{< /highlight >}}

You could also just create a Stackblitz Angular project too.

Our project will be here for reference: <a href="https://stackblitz.com/edit/angular-zhunbu?embed=1" target="_blank">Virtual Bananas</a>

## Setting up some fake data

First things first, let's set up a fake data source in your application.

Create a service called `banana.service.ts` provided in root with the following method: