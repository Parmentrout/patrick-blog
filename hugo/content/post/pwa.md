---
title: 'Angular Progressive Web Apps'
description: 'Build your first Angular PWA!'
author:
  name: 'Patrick Armentrout'
  desc: ' '
date: 2019-04-23
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

## Example

Here is a quick example of PWA behavior on a phone, of an app I just build about The Bachelor TV show:  

<a href="https://photos.google.com/share/AF1QipOAqQI36QOV1nhAevN3cbhYvF0TX8dpc2OD2rQC_OmbjB5UTXdvvYleK4w5T9kOkw/photo/AF1QipPCPab5I6j8PM9isV7kvR-_LIU0C68zUqgz5w2y?key=RnkwanBJVl81dmVZYUhheklCZk8tX0F5cmJaZ053" target="_blank">Demo</a>

# Getting Started

We're going to be building an app from scratch. To get started in Angular is very easy:

    ng new bachelor-app

    ng add @angular/pwa --project bachelor-app

{{< highlight powershell >}}

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

This gives us all the fundamental parts of an Angular PWA.  Here is a high level of all the changes that were made to our solution:

* ngsw-config.json - Service Worker config file 
* manifest.json - Manifest File 
* Sample Icons for our manifest
* angular.json - add service worker flag (this tells the build to register the service worker on a prod build)
* app.module.ts - Registers ServiceWorkerModule which allows us to catch events in code (like updates available and push notifications)
* index.html - Registers manifest file and adds a meta theme color 

## Service Worker Configuration

The `ngsw.config` is our service worker configuration file.  It drives all of the caching in our application.  If you haven't heard the term service worker before, it is a stateful worker running in the background of the browser.  Basically it intercepts all outbound and incoming traffic and builds caching based on configuration.  Here is a great visual from <a href="https://asc.altkom.pl/en/wp-content/uploads/sites/6/2018/11/Service-Worker%E2%80%99s-mechanism.png" target="_blank">Altkom Sofware and Consulting</a>.

The config file is broken up into 2 groups:  Asset Groups, and Data Groups:

### Asset Groups

Asset Groups are all the static assets for our website.  Typically you won't need to change much in this section however it's good to know what is going on here. The type is structured like the following:

{{< highlight typescript >}}

interface AssetGroup {
  name: string;
  installMode?: 'prefetch' | 'lazy';
  updateMode?: 'prefetch' | 'lazy';
  resources: {
    files?: string[];
    @deprecated
    versionedFiles?: string[];
    urls?: string[];
  };
}

{{< /highlight >}}

* `name` - unique identifier for our asset group. The json is `Array<AssetGroup>` so you can set up caching rules for different configurations.
* `installMode`
  * `prefetch` - tells the service worker to fetch and cache all listed resources (heavy up-front workload)
  * `lazy` - loads the resources as needed and caches accordingly (good for images or large files)
* `updateMode`
  * `prefetch` | `lazy` - Catch is that if installMode is lazy, updateMode must also be lazy!
* `resources/files` | `resources/urls`
  * A list of static files, urls and filepaths to cache

<br>
#### *Quick Caching Note

Typically the pattern here is prefetch for static js, css, html files and lazy loading for image and video files.  With prefetch, if the user has already visited the page then it will pull the cached version of the app until they reload.  That's why it's crucial to add an "Updates Available" button that we'll get to later in this article :)

### Data Groups

Data Groups handle all of our external api caching.  So in addition to being able to cache our website with Asset Groups, we can also cache our api calls if we want.  This is where you might want to do heavy configuration to improve your site's performance. The type looks as follows:

{{< highlight typescript >}}

export interface DataGroup {
  name: string;
  urls: string[];
  version?: number;
  cacheConfig: {
    maxSize: number;
    maxAge: string;
    timeout?: string;
    strategy?: 'freshness' | 'performance';
  };
}

{{< /highlight >}}

* `name` - unique identifier for the group
* `urls` - a list of urls to include in this cache group.  Wildcards allowed, (i.e. `https://fonts.googleapis.com/**`)
* `version` - a semantic version for this group.  This is important, because new versions will bypass the cache to fetch fresh data
* `maxSize` - the maximum number of responses to hold onto (I'm not sure why this would ever be more than one, but it has to be populated so the cache doesn't grow indefinitely)
* `maxAge` - the invalidation age, you can use shorthand here (i.e. `60d`, `30s`, `100ms`)
* `timeout` - the timeout on the api call in which your api will fall back to read from cache
* `strategy`
  * `performance` - "cache-first" meaning the api will pull from cache before it tries over the network.  This is invalidated on a new dataGroup version though or if the maxAge has expired.  In that case the call will hit the network, then update the cache.
  * `freshness` - "network-first" meaning the api will try to hit external first.  For api's that update frequently this is the better option.  If the api fails however, it will fall back the most recent cached response

### Real world example

This is a data group configuration for the demo project above:

{{< highlight typescript >}}

 "dataGroups": [{
      "name": "scoreboard-api",
      "urls": [
        "https://bachelor-pwa.firebaseio.com/scoreboard.json"
      ],
      "version": "1.0",
      "cacheConfig": {
        "maxSize": 1,
        "maxAge": "60d",
        "timeout": "10s",
        "strategy": "performance"
      }
    },
    {
      "name": "contestants-api",
      "urls": [
        "https://bachelor-pwa.firebaseio.com/contestants.json"
      ],
      "version": "1.0",
      "cacheConfig": {
        "maxSize": 1,
        "maxAge": "7d",
        "timeout": "2s",
        "strategy": "freshness"
      }
    }
  ]
{{< /highlight >}}

Notice the first api "scoreboard-api".  This api doesn't change very often, however is used heavily in the application.  So I set a 60 day invalidation and make it `performance` so that it pulls from the cache. 

However in the second api "contestants-api".  This call is updated frequently so I want it with `freshness` or network-first.  Notice I have a 2 second timeout, so if the api fails, the user will see the latest available cached data.

As you can see, the service worker is highly configurable and can be a powerful caching tool to improve your app's performance.

# Manifest Configuration

The `manifest.webmanifest` is our website manifest file.  If you haven't heard of a [webmanifest](https://developer.mozilla.org/en-US/docs/Web/Manifest) it is a metadata file for a webpage.  For our purposes, it's used by the browser for the application name, app icon, and other styling.  When the browser prompts you: "Would you like to add myApp to your homepage?".  The styling on this functionality will come from the manifest file

An example a manifest in an Angular project would be:

{{< highlight typescript >}}

{
   "short_name": "bachelor",
   "name": "Bachelor Tracker",
   "icons": [
       {
       "src": "/images/icons-192.png",
       "type": "image/png",
       "sizes": "192x192"
       },
       {
       "src": "/images/icons-512.png",
       "type": "image/png",
       "sizes": "512x512"
       }
   ],
   "start_url": "/",
   "background_color": "#3367D6",
   "display": "standalone",
   "scope": "/",
   "theme_color": "#3367D6"
}

{{< /highlight >}}

You'll want to leave the startUrl and scope alone if you are hosting at the origin of your url.  However for the other data, here are a few behavior gotchas:

* `short_name` - The name the appears under the image in your app selection screen.  The is also the name that will show in the prompt shown above.
* `name` - The name that will appear on the "splash screen" before the app loads
* `icon` - A handy trick!  Use  <a href="https://app-manifest.firebaseapp.com/" target="_blank">Manifest Generator</a> to generate your icons.  If you upload a 512px x 512px it will create the other device sizes you need
* `display` - keep as "standalone" for the home screen functionalit_y
* `background_color` - This is the splash screen background.  It should be the same color as your application background so the experience isn't jarring for the user
* `theme_color` - should be the same as the meta-theme tag color! 
  * A gotcha - make sure this color also works well with your image as they are overlayed on some devices

## Manifest Testing

To test your manifest, you can double check that your configuration with Chrome Dev tools (Application Section).  Also!  If you run an Audit in Chrome Dev tools you can verify that you have all the right info for the Add to Home Screen to work for your users (this would need to be on a production build however, as the service worker isn't registered in a dev build).

# Updates Available Button

The last thing I'll mention is updates available functionality.  Since service workers will cache your static website, you'll want to inform your users that a new version exists (Just like a native app!).  To do this, we'll utilize the ServiceWorkerModule that was injected in `app.module.ts`:

{{< highlight typescript >}}

import { SwUpdate } from '@angular/service-worker';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent {

  public hasUpdates: boolean = false;

  constructor(private swUpdate: SwUpdate) {
    this.subscribeToUpdates();
  }

  refreshPage() {
    window.location.reload();
  }

  private subscribeToUpdates() {

    // Check for available updates with swUpdate service
    this.swUpdate.available.subscribe(event => {
      console.log('current version is', event.current);
      console.log('available version is', event.available);

      this.hasUpdates = true;
    });

    // Check if updates have been applied
    this.swUpdate.activated.subscribe(event => {
      console.log('old version was', event.previous);
      console.log('new version is', event.current);

      this.hasUpdates = false;
    });
  }

{{< /highlight >}}

Now bind a refresh button on the homepage with:

{{< highlight html >}}

<button *ngIf="hasUpdates" (click)="refreshPage()" >Updates Available</button>

{{< /highlight >}}

All it's doing is forcing a page reload, however it lets your users know that you care and you want them to have the latest and greatest, yay!

# References

Thanks for reading everyone!  Here are a few helpful links if you want to learn more about Angular PWA's:

A recent PWA I built: https://github.com/Parmentrout/bachelor-pwa, plus it's hosted website if you want to see the behavior: https://bachelor-pwa.firebaseapp.com/

More PWA documentation from Angular and Google!
https://developers.google.com/web/progressive-web-apps/ 
https://angular.io/guide/service-worker-config 
