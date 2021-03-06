# Meteor Paginated Subscription

This package is an experiment that adds pagination to Meteor's standard subscriptions. It's a byproduct of the [Telescope project](http://telesc.pe).

## Installation

Install via  [Meteorite](https://github.com/oortcloud/meteorite/):


``` sh
$ mrt add paginated-subscription
```

## Usage

This package makes available a single function `Meteor.subscribeWithPagination`. Like the built in `Meteor.subscribe`, it returns a handle, which should be used to keep track of the state of the subscription:

```js
var handle = Meteor.subscribeWithPagination('posts', 10);
```

The arguments are as usual to `Meteor.subscribe`, with two exceptions:

1. The last argument must be a number, indicating the number of documents per page.
2. There's no support for a `onReady` callback, see TODO below.

The paginated subscription expects you to have a publication setup, as normal, which expects as a final argument the *current* number of documents to display (which will be incremented, in a infinite scroll fashion):

```js
Meteor.publish('posts', function(limit) {
  return Posts.find({}, {limit: limit});
});
```

The important part of all this is the `handle`, which has the following API:

 - `handle.loaded()` - how many documents are currently loaded via the sub
 - `handle.limit()` - how many have we asked for
 - `handle.ready()` - are we waiting on the server right now?
 - `handle.loadNextPage()` - fetch the next page of results

The first three functions are reactive and thus can be used to correctly display an 'infinite-scroll' like list of results. Please see the [telescope](https://github.com/SachaG/Telescope/blob/master/client/views/posts/posts_list.js) project for an example of real-world usage.

## Caveats / Limitations

The most important caveat is that the subscription shouldn't be used from within an `autorun` block. The reason for this is that although the three handle functions are reactive, there's nothing making the handle itself reactive and so your templates will be relying on reactive functions that no longer exist if the handle changes.

If your subscriptions arguments are going to reactively change, simply pass in functions, and the paginated sub will evaluate them within it's own internal `autorun` block.

## TODO

Contributions are heavily encouraged. The obvious things to fix are:

1. Figure out a good way to imitate what `subscribe` does so we can avoid the "pass in functions" hack.
2. Figure out a way to get a `onReady` callback (the obvious way is to solve 1.)
3. Do actual "pagination" rather than "infinite scroll" -- i.e. have an option to pass around an offset as well as limit.

Please contact me if you want to have a go at these and I'll be happy to help in what ways I can.