---
title: 'Fun with RXJS'
description: 'RXJS Patterns in Angular 7'
author:
  name: 'Patrick Armentrout'
  desc: ' '
date: 2019-03-26
draft: false
categories:
  - tutorials
---

## About

[RXJS](angular.io/guide/rx-library) (if you are new to Angular) is an asynchronous library used within Angular that allows us to manipulate a data stream. In this post, I'll go through a few patterns I've consistently used, and how to apply them to incoming data.

We're going to take a look at the following operators:

- map
- tap
- filter
- takeUntil
- switchMap

### map

The `map` operator allows us to synchronously manipulate data in a pipe and return it as a different type. For instance if I have the following method:

    import { interval } from 'rxjs';

    const myObserver$ = interval(1000).subscribe(result => console.log(result));

The previous code will fire every second and produce the following output: `1, 2, 3, 4...`.

Say we need to use our interval to instead create message with the number of times counted. As it stands now we'd have to subscribe to the observable and put our string conversion method in the `.subscribe(result => [convert string here..])`.

However with a `map` operator we can to convert our interval from `Observable<number>` to `Observable<string>` at the pipe level. For instance:

    import { map } from 'rxjs/operators';

    const myObserver$ = interval(1000)
        .pipe(map(result => `Counter: ${result}`);

So any subscriber listening would get `'Counter: 1', 'Counter: 2', 'Counter: 3....'`

This is really useful in common conversions in observables, as well as any business logic you want to have at the service level. For instance:

<b>my-service.ts</b>

    getData$(): Observable<string> {
        return this.http.get('http://harry-potter-data.json')
            .pipe(map(result => result.json()));
    }

Instead of having to do a json() convert in every subscriber that uses our getData\$() method, we can include the logic in a map operator and only make it at the topmost layer! This prevents all subscribers from having to duplicate logic, and more easily allows the data to persist neatly in an [async pipe](https://angular.io/api/common/AsyncPipe).

###tap

Similar to map, however the tap operator won't manipulate the data in the stream. This is a handy operator for logging and general background processing.

I'll use this a lot for controlling page load logic:

    private _isLoading: boolean = true;

    ngOnInit() {
        this._myService.getData$().pipe(tap(() => {
            console.log('Data Fetch successful');
            this._isLoading = false;
        }))
    }

Both the map and the tap operator are especially important for when using an async pipe and persisting Observables all the way to the template.

###takeUntil

As powerful as Observers are they can also be a potential memory leak if not cleaned up carefully. takeUntil allows us to add an event listener to an Observable stream. When the event inside of takeUntil fires, the stream will close itself from any incoming messages. For instance take the component below:

    private _cancellationToken: Subject<void> = new Subject();

    ngOnInit() {
        this._myService.getData$().pipe(takeUntil(this._cancellationToken)).subscribe(() => { ...});

        this.myObserv$ = interval(2000).pipe(takeUntil(this._cancellationToken)).subscribe(() => { ...});
    }

    ngOnDestroy() {
        this._cancellationToken.next();
        this._cancellationToken.complete();
    }

We have potentially two open observables that need to be closed when the component is destroyed. Now, rather than having to create the observables as private methods and unsubscribing manually, we can use the takeUntil operator on each pipe. Then when the component destroys, the `_cancellationToken` Subject fires, an all of our listening pipes will close themselves.

This is a great pattern for all components with observable subscriptions

### switchmap
