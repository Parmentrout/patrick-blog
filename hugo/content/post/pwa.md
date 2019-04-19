---
title: 'Angular Progressive Web Apps'
description: 'Build your first Angular PWA!'
author:
  name: 'Patrick Armentrout'
  desc: ' '
date: 2019-04-09
draft: false
categories:
  - tutorials
---

# Purpose

Let's build a Progressive Web App with Angular 7!

# First - wtf is a PWA?

Before we get started what is a PWA (Progressive Web App)? The technical definition is that a PWA is a set of WC3 standards that allows a web application to behave like a mobile application. In layman's terms, if you're ever browsing on your phone and you see that popup, "Do you want to add [app] to your home screen?", this is technically a PWA. When you click on it, an icon is added to your phone.

Here is a full list of PWA behavior:

Progressive to work for every user/browser

- Responsive / mobile first design
- “Add to home screen” feature allows for users to add to their app library
- Ability to work offline or over slow connections using service workers
- Manifest file to control the mobile user experience
- Icons, splash screen, name, theming, etc..
- Ability to send push notifications
- Easily discoverable by search engines due to manifest and service worker registration scope
- Ability to interact with phone hardware

If you're reading this on a phone or tablet right now and asking yourself, "Wait a second, why isn't his blog website PWA enabled!?". The answer is because I'm lazy ;), but I will add it in a future release.

# Getting Started

We're going to be building an app from scratch, however I'll be using a recent PWA I built for examples:  [Bachelor Tracker](https://bachelor-pwa.firebaseapp.com).  The source code is [here](https://github.com/Parmentrout/bachelor-pwa).  It is exactly what it sounds like, a way for us to track who is eliminated from the TV show the Bachelor, don't judge me please.


{{< highlight powershell >}}

    ng new bachelor-app

    ng add @angular/pwa --project bachelor-app

  Installed packages for tooling via npm.
    CREATE ngsw-config.json (441 bytes)
    CREATE src/manifest.webmanifest (1073 bytes)
    CREATE src/assets/icons/icon-128x128.png (1253 bytes)
    CREATE src/assets/icons/icon-144x144.png (1394 bytes)
    CREATE src/assets/icons/icon-152x152.png (1427 bytes)
    CREATE src/assets/icons/icon-192x192.png (1790 bytes)
    CREATE src/assets/icons/icon-384x384.png (3557 bytes)
    CREATE src/assets/icons/icon-512x512.png (5008 bytes)
    CREATE src/assets/icons/icon-72x72.png (792 bytes)
    CREATE src/assets/icons/icon-96x96.png (958 bytes)
    UPDATE angular.json (3998 bytes)
    UPDATE package.json (1387 bytes)
    UPDATE src/app/app.module.ts (604 bytes)
    UPDATE src/index.html (476 bytes)

{{< /highlight >}}

This gives us all the fundamental parts of an Angular PWA.  We're going to go through all these changes in 2 parts.  Service Worker and Manifest setup.

## Service Worker Configuration

The `ngsw.config` is our service worker configuration file.  It drives all of the caching in our application.  You'll notice this file is broken up into 2 sections:

### assetGroups

AssetGroups are all the static assets for our website.  By default Angular will give you the following setup:

{{< highlight json >}}

"assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": [
          "/favicon.ico",
          "/index.html",
          "/*.css",
          "/*.js"
        ]
      }
    }]


{{< /highlight >}}

* `name` - unique identifier for our asset group.  Notice it's an array so you can set up caching rules differently
* `installMode`
  * `prefetch` - tells the service worker to fetch and cache all listed resources (heavy up-front workload)
  * `lazy` - loads the resources as needed and caches accordingly
* `updateMode`
  * `prefetch` | `lazy` - Catch is that if installMode is lazy, updateMode must also be lazy!
* `resources/files`
  *