# abortable-promise-cache

[![NPM version](https://img.shields.io/npm/v/abortable-promise-cache.svg?style=flat-square)](https://npmjs.org/package/abortable-promise-cache)
[![Coverage Status](https://img.shields.io/codecov/c/github/rbuels/abortable-promise-cache/master.svg?style=flat-square)](https://codecov.io/gh/rbuels/abortable-promise-cache/branch/master)
[![Build Status](https://travis-ci.com/rbuels/abortable-promise-cache.svg?branch=master)](https://travis-ci.com/rbuels/abortable-promise-cache)

Adds AbortController/AbortSignal semantics to a cache of promises. Each `get` from the cache can optionally take an `AbortSignal` object that lets that request be aborted.

Cached fill requests will be aborted and evicted from the cache if all the incoming requests for it
are aborted before the promise settles.

If the fill request has already settled before all the requests for it have been aborted, it will stay
in the cache.

## Install

    $ npm install --save abortable-promise-cache

## Usage

```js
import AbortablePromiseCache from 'abortable-promise-cache'
import QuickLRU from 'quick-lru'

const cache = new AbortablePromiseCache({
    // QuickLRU is a good backing cache to use, but you can use any
    // cache as long as it supports `get`, `set`, `delete`, and `keys`.
    cache: new QuickLRU({ maxSize: 1000 }),

    // the `fill` callback will be called for a cache miss
    async fill(requestData, abortSignal) {
        // do some long-running thing
        return longRunningThing(requestData, abortSignal)
    }
})

// Make a cached request. The returned promise will abort with the given abort signal if
// there is not already a cached copy that has been resolved.
// Fill requests will be signaled to abort if all the requests for them
// so far have been aborted.
const aborter = new AbortController()
const result = await cache.get('some key', { ...anyStuff }, aborter.signal)

// deleting and clearing will abort any outstanding requests
cache.delete('some key')
cache.clear()
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [constructor](#constructor)
-   [get](#get)
-   [delete](#delete)
-   [clear](#clear)

### constructor

**Parameters**

-   `args` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** constructor args
    -   `args.fill` **[Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** fill callback, will be called with sig `fill(data, signal)`
    -   `args.cache` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** backing store to use, must implement `get(key)`, `set(key, val)`,
          `delete(key)`, and `keys() -> iterator`

### get

**Parameters**

-   `key` **any** cache key to use for this request
-   `data` **any** data passed as the first argument to the fill callback
-   `signal` **AbortSignal?** optional AbortSignal object that aborts the request

### delete

delete the given entry from the cache. if it exists and its fill request has
not yet settled, the fill will be signaled to abort.

**Parameters**

-   `key` **any** 

### clear

Clear all requests from the cache. Aborts any that have not settled.

Returns **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** count of entries deleted

## Academic Use

This package was written with funding from the [NHGRI](http://genome.gov) as part of the [JBrowse](http://jbrowse.org) project. If you use it in an academic project that you publish, please cite the most recent JBrowse paper, which will be linked from [jbrowse.org](http://jbrowse.org).

## License

MIT © [Robert Buels](https://github.com/rbuels)
