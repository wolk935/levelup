LevelUP
=======

<img alt="LevelDB Logo" height="100" src="http://leveldb.org/img/logo.svg">

**Fast & simple storage - a Node.js-style LevelDB wrapper**

[![Build Status](https://secure.travis-ci.org/Level/levelup.png)](http://travis-ci.org/Level/levelup)

[![NPM](https://nodei.co/npm/levelup.png?stars&downloads&downloadRank)](https://nodei.co/npm/levelup/) [![NPM](https://nodei.co/npm-dl/levelup.png?months=6&height=3)](https://nodei.co/npm/levelup/)


  * <a href="#intro">Introduction</a>
  * <a href="#leveldown">Relationship to LevelDOWN</a>
  * <a href="#platforms">Tested &amp; supported platforms</a>
  * <a href="#basic">Basic usage</a>
  * <a href="#api">API</a>
  * <a href="#events">Events</a>
  * <a href="#json">JSON data</a>
  * <a href="#custom_encodings">Custom encodings</a>
  * <a href="#extending">Extending LevelUP</a>
  * <a href="#multiproc">Multi-process access</a>
  * <a href="#support">Getting support</a>
  * <a href="#contributing">Contributing</a>
  * <a href="#licence">Licence &amp; copyright</a>

<a name="intro"></a>
Introduction
------------

**[LevelDB](https://github.com/google/leveldb)** is a simple key/value data store built by Google, inspired by BigTable. It's used in Google Chrome and many other products. LevelDB supports arbitrary byte arrays as both keys and values, singular *get*, *put* and *delete* operations, *batched put and delete*, bi-directional iterators and simple compression using the very fast [Snappy](http://code.google.com/p/snappy/) algorithm.

**LevelUP** aims to expose the features of LevelDB in a **Node.js-friendly way**. All standard `Buffer` encoding types are supported, as is a special JSON encoding. LevelDB's iterators are exposed as a Node.js-style **readable stream**.

LevelDB stores entries **sorted lexicographically by keys**. This makes LevelUP's <a href="#createReadStream"><code>ReadStream</code></a> interface a very powerful query mechanism.

**LevelUP** is an **OPEN Open Source Project**, see the <a href="#contributing">Contributing</a> section to find out what this means.

<a name="leveldown"></a>
Relationship to LevelDOWN
-------------------------

LevelUP is designed to be backed by **[LevelDOWN](https://github.com/level/leveldown/)** which provides a pure C++ binding to LevelDB and can be used as a stand-alone package if required.

**As of version 0.9, LevelUP no longer requires LevelDOWN as a dependency so you must `npm install leveldown` when you install LevelUP.**

LevelDOWN is now optional because LevelUP can be used with alternative backends, such as **[level.js](https://github.com/maxogden/level.js)** in the browser or [MemDOWN](https://github.com/level/memdown) for a pure in-memory store.

LevelUP will look for LevelDOWN and throw an error if it can't find it in its Node `require()` path. It will also tell you if the installed version of LevelDOWN is incompatible.

**The [level](https://github.com/level/level) package is available as an alternative installation mechanism.** Install it instead to automatically get both LevelUP & LevelDOWN. It exposes LevelUP on its export (i.e. you can `var leveldb = require('level')`).


<a name="platforms"></a>
Tested & supported platforms
----------------------------

  * **Linux**: including ARM platforms such as Raspberry Pi *and Kindle!*
  * **Mac OS**
  * **Solaris**: including Joyent's SmartOS & Nodejitsu
  * **Windows**: Node 0.10 and above only. See installation instructions for *node-gyp's* dependencies [here](https://github.com/TooTallNate/node-gyp#installation), you'll need these (free) components from Microsoft to compile and run any native Node add-on in Windows.

<a name="basic"></a>
Basic usage
-----------

First you need to install LevelUP!

```sh
$ npm install levelup leveldown
```

Or

```sh
$ npm install level
```

*(this second option requires you to use LevelUP by calling `var levelup = require('level')`)*


All operations are asynchronous although they don't necessarily require a callback if you don't need to know when the operation was performed.

```js
var levelup = require('levelup')

// 1) Create our database, supply location and options.
//    This will create or open the underlying LevelDB store.
var db = levelup('./mydb')

// 2) put a key & value
db.put('name', 'LevelUP', function (err) {
  if (err) return console.log('Ooops!', err) // some kind of I/O error

  // 3) fetch by key
  db.get('name', function (err, value) {
    if (err) return console.log('Ooops!', err) // likely the key was not found

    // ta da!
    console.log('name=' + value)
  })
})
```

<a name="api"></a>
## API

  * <a href="#ctor"><code><b>levelup()</b></code></a>
  * <a href="#open"><code>db.<b>open()</b></code></a>
  * <a href="#close"><code>db.<b>close()</b></code></a>
  * <a href="#put"><code>db.<b>put()</b></code></a>
  * <a href="#get"><code>db.<b>get()</b></code></a>
  * <a href="#del"><code>db.<b>del()</b></code></a>
  * <a href="#batch"><code>db.<b>batch()</b></code> *(array form)*</a>
  * <a href="#batch_chained"><code>db.<b>batch()</b></code> *(chained form)*</a>
  * <a href="#isOpen"><code>db.<b>isOpen()</b></code></a>
  * <a href="#isClosed"><code>db.<b>isClosed()</b></code></a>
  * <a href="#createReadStream"><code>db.<b>createReadStream()</b></code></a>
  * <a href="#createKeyStream"><code>db.<b>createKeyStream()</b></code></a>
  * <a href="#createValueStream"><code>db.<b>createValueStream()</b></code></a>

### Special operations exposed by LevelDOWN

  * <a href="#approximateSize"><code>db.db.<b>approximateSize()</b></code></a>
  * <a href="#getProperty"><code>db.db.<b>getProperty()</b></code></a>
  * <a href="#destroy"><code><b>leveldown.destroy()</b></code></a>
  * <a href="#repair"><code><b>leveldown.repair()</b></code></a>

### Special Notes
  * <a href="#writeStreams">What happened to <code><b>db.createWriteStream()</b></code></a>


--------------------------------------------------------
<a name="ctor"></a>
### levelup(location[, options[, callback]])
### levelup(options[, callback ])
### levelup(db[, callback ])
<code>levelup()</code> is the main entry point for creating a new LevelUP instance and opening the underlying store with LevelDB.

This function returns a new instance of LevelUP and will also initiate an <a href="#open"><code>open()</code></a> operation. Opening the database is an asynchronous operation which will trigger your callback if you provide one. The callback should take the form: `function (err, db) {}` where the `db` is the LevelUP instance. If you don't provide a callback, any read & write operations are simply queued internally until the database is fully opened.

This leads to two alternative ways of managing a new LevelUP instance:

```js
levelup(location, options, function (err, db) {
  if (err) throw err
  db.get('foo', function (err, value) {
    if (err) return console.log('foo does not exist')
    console.log('got foo =', value)
  })
})

// vs the equivalent:

var db = levelup(location, options) // will throw if an error occurs
db.get('foo', function (err, value) {
  if (err) return console.log('foo does not exist')
  console.log('got foo =', value)
})
```

The `location` argument is available as a read-only property on the returned LevelUP instance.

The `levelup(options, callback)` form (with optional `callback`) is only available where you provide a valid `'db'` property on the options object (see below). Only for back-ends that don't require a `location` argument, such as [MemDOWN](https://github.com/level/memdown).

For example:

```js
var levelup = require('levelup')
var memdown = require('memdown')
var db = levelup({ db: memdown })
```

The `levelup(db, callback)` form (with optional `callback`) is only available where `db` is a factory function, as would be provided as a `'db'` property on an `options` object (see below). Only for back-ends that don't require a `location` argument, such as [MemDOWN](https://github.com/level/memdown).

For example:

```js
var levelup = require('levelup')
var memdown = require('memdown')
var db = levelup(memdown)
```

#### `options`

`levelup()` takes an optional options object as its second argument; the following properties are accepted:

* `'createIfMissing'` *(boolean, default: `true`)*: If `true`, will initialise an empty database at the specified location if one doesn't already exist. If `false` and a database doesn't exist you will receive an error in your `open()` callback and your database won't open.

* `'errorIfExists'` *(boolean, default: `false`)*: If `true`, you will receive an error in your `open()` callback if the database exists at the specified location.

* `'compression'` *(boolean, default: `true`)*: If `true`, all *compressible* data will be run through the Snappy compression algorithm before being stored. Snappy is very fast and shouldn't gain much speed by disabling so leave this on unless you have good reason to turn it off.

* `'cacheSize'` *(number, default: `8 * 1024 * 1024`)*: The size (in bytes) of the in-memory [LRU](http://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) cache with frequently used uncompressed block contents.

* `'keyEncoding'` and `'valueEncoding'` *(string, default: `'utf8'`)*: The encoding of the keys and values passed through Node.js' `Buffer` implementation (see [Buffer#toString()](http://nodejs.org/docs/latest/api/buffer.html#buffer_buf_tostring_encoding_start_end)).
  <p><code>'utf8'</code> is the default encoding for both keys and values so you can simply pass in strings and expect strings from your <code>get()</code> operations. You can also pass <code>Buffer</code> objects as keys and/or values and conversion will be performed.</p>
  <p>Supported encodings are: hex, utf8, ascii, binary, base64, ucs2, utf16le.</p>
  <p><code>'json'</code> encoding is also supported, see below.</p>

* `'db'` *(object, default: LevelDOWN)*: LevelUP is backed by [LevelDOWN](https://github.com/level/leveldown/) to provide an interface to LevelDB. You can completely replace the use of LevelDOWN by providing a "factory" function that will return a LevelDOWN API compatible object given a `location` argument. For further information, see [MemDOWN](https://github.com/level/memdown), a fully LevelDOWN API compatible replacement that uses a memory store rather than LevelDB. Also see [Abstract LevelDOWN](http://github.com/level/abstract-leveldown), a partial implementation of the LevelDOWN API that can be used as a base prototype for a LevelDOWN substitute.

Additionally, each of the main interface methods accept an optional options object that can be used to override `'keyEncoding'` and `'valueEncoding'`.

--------------------------------------------------------
<a name="open"></a>
### db.open([callback])
<code>open()</code> opens the underlying LevelDB store. In general **you should never need to call this method directly** as it's automatically called by <a href="#ctor"><code>levelup()</code></a>.

However, it is possible to *reopen* a database after it has been closed with <a href="#close"><code>close()</code></a>, although this is not generally advised.

--------------------------------------------------------
<a name="close"></a>
### db.close([callback])
<code>close()</code> closes the underlying LevelDB store. The callback will receive any error encountered during closing as the first argument.

You should always clean up your LevelUP instance by calling `close()` when you no longer need it to free up resources. A LevelDB store cannot be opened by multiple instances of LevelDB/LevelUP simultaneously.

--------------------------------------------------------
<a name="put"></a>
### db.put(key, value[, options][, callback])
<code>put()</code> is the primary method for inserting data into the store. Both the `key` and `value` can be arbitrary data objects.

The callback argument is optional but if you don't provide one and an error occurs then expect the error to be thrown.

#### `options`

Encoding of the `key` and `value` objects will adhere to `'keyEncoding'` and `'valueEncoding'` options provided to <a href="#ctor"><code>levelup()</code></a>, although you can provide alternative encoding settings in the options for `put()` (it's recommended that you stay consistent in your encoding of keys and values in a single store).

If you provide a `'sync'` value of `true` in your `options` object, LevelDB will perform a synchronous write of the data; although the operation will be asynchronous as far as Node is concerned. Normally, LevelDB passes the data to the operating system for writing and returns immediately, however a synchronous write will use `fsync()` or equivalent so your callback won't be triggered until the data is actually on disk. Synchronous filesystem writes are **significantly** slower than asynchronous writes but if you want to be absolutely sure that the data is flushed then you can use `'sync': true`.

--------------------------------------------------------
<a name="get"></a>
### db.get(key[, options][, callback])
<code>get()</code> is the primary method for fetching data from the store. The `key` can be an arbitrary data object. If it doesn't exist in the store then the callback will receive an error as its first argument. A not-found err object will be of type `'NotFoundError'` so you can `err.type == 'NotFoundError'` or you can perform a truthy test on the property `err.notFound`.

```js
db.get('foo', function (err, value) {
  if (err) {
    if (err.notFound) {
      // handle a 'NotFoundError' here
      return
    }
    // I/O or other error, pass it up the callback chain
    return callback(err)
  }

  // .. handle `value` here
})
```

#### `options`

Encoding of the `key` and `value` objects is the same as in <a href="#put"><code>put</code></a>. 

LevelDB will by default fill the in-memory LRU Cache with data from a call to get. Disabling this is done by setting `fillCache` to `false`.

--------------------------------------------------------
<a name="del"></a>
### db.del(key[, options][, callback])
<code>del()</code> is the primary method for removing data from the store.
```js
db.del('foo', function (err) {
  if (err)
    // handle I/O or other error
});
```

#### `options`

Encoding of the `key` object will adhere to the `'keyEncoding'` option provided to <a href="#ctor"><code>levelup()</code></a>, although you can provide alternative encoding settings in the options for `del()` (it's recommended that you stay consistent in your encoding of keys and values in a single store).

A `'sync'` option can also be passed, see <a href="#put"><code>put()</code></a> for details on how this works.

--------------------------------------------------------
<a name="batch"></a>
### db.batch(array[, options][, callback]) *(array form)*
<code>batch()</code> can be used for very fast bulk-write operations (both *put* and *delete*). The `array` argument should contain a list of operations to be executed sequentially, although as a whole they are performed as an atomic operation inside LevelDB.

Each operation is contained in an object having the following properties: `type`, `key`, `value`, where the *type* is either `'put'` or `'del'`. In the case of `'del'` the `'value'` property is ignored. Any entries with a `'key'` of `null` or `undefined` will cause an error to be returned on the `callback` and any `'type': 'put'` entry with a `'value'` of `null` or `undefined` will return an error.

If `key` and `value` are defined but `type` is not, it will default to `'put'`.

```js
var ops = [
    { type: 'del', key: 'father' }
  , { type: 'put', key: 'name', value: 'Yuri Irsenovich Kim' }
  , { type: 'put', key: 'dob', value: '16 February 1941' }
  , { type: 'put', key: 'spouse', value: 'Kim Young-sook' }
  , { type: 'put', key: 'occupation', value: 'Clown' }
]

db.batch(ops, function (err) {
  if (err) return console.log('Ooops!', err)
  console.log('Great success dear leader!')
})
```

#### `options`

See <a href="#put"><code>put()</code></a> for a discussion on the `options` object. You can overwrite default `'keyEncoding'` and `'valueEncoding'` and also specify the use of `sync` filesystem operations.

In addition to encoding options for the whole batch you can also overwrite the encoding per operation, like:

```js
var ops = [{
    type          : 'put'
  , key           : new Buffer([1, 2, 3])
  , value         : { some: 'json' }
  , keyEncoding   : 'binary'
  , valueEncoding : 'json'
}]
```

--------------------------------------------------------
<a name="batch_chained"></a>
### db.batch() *(chained form)*
<code>batch()</code>, when called with no arguments will return a `Batch` object which can be used to build, and eventually commit, an atomic LevelDB batch operation. Depending on how it's used, it is possible to obtain greater performance when using the chained form of `batch()` over the array form.

```js
db.batch()
  .del('father')
  .put('name', 'Yuri Irsenovich Kim')
  .put('dob', '16 February 1941')
  .put('spouse', 'Kim Young-sook')
  .put('occupation', 'Clown')
  .write(function () { console.log('Done!') })
```

<b><code>batch.put(key, value[, options])</code></b>

Queue a *put* operation on the current batch, not committed until a `write()` is called on the batch.

The optional `options` argument can be used to override the default `'keyEncoding'` and/or `'valueEncoding'`.

This method may `throw` a `WriteError` if there is a problem with your put (such as the `value` being `null` or `undefined`).

<b><code>batch.del(key[, options])</code></b>

Queue a *del* operation on the current batch, not committed until a `write()` is called on the batch.

The optional `options` argument can be used to override the default `'keyEncoding'`.

This method may `throw` a `WriteError` if there is a problem with your delete.

<b><code>batch.clear()</code></b>

Clear all queued operations on the current batch, any previous operations will be discarded.

<b><code>batch.write([callback])</code></b>

Commit the queued operations for this batch. All operations not *cleared* will be written to the database atomically, that is, they will either all succeed or fail with no partial commits. The optional `callback` will be called when the operation has completed with an *error* argument if an error has occurred; if no `callback` is supplied and an error occurs then this method will `throw` a `WriteError`.


--------------------------------------------------------
<a name="isOpen"></a>
### db.isOpen()

A LevelUP object can be in one of the following states:

  * *"new"*     - newly created, not opened or closed
  * *"opening"* - waiting for the database to be opened
  * *"open"*    - successfully opened the database, available for use
  * *"closing"* - waiting for the database to be closed
  * *"closed"*  - database has been successfully closed, should not be used

`isOpen()` will return `true` only when the state is "open".

--------------------------------------------------------
<a name="isClosed"></a>
### db.isClosed()

*See <a href="#put"><code>isOpen()</code></a>*

`isClosed()` will return `true` only when the state is "closing" *or* "closed", it can be useful for determining if read and write operations are permissible.

--------------------------------------------------------
<a name="createReadStream"></a>
### db.createReadStream([options])

You can obtain a **ReadStream** of the full database by calling the `createReadStream()` method. The resulting stream is a complete Node.js-style [Readable Stream](http://nodejs.org/docs/latest/api/stream.html#stream_readable_stream) where `'data'` events emit objects with `'key'` and `'value'` pairs. You can also use the `gt`, `lt` and `limit` options to control the range of keys that are streamed.

```js
db.createReadStream()
  .on('data', function (data) {
    console.log(data.key, '=', data.value)
  })
  .on('error', function (err) {
    console.log('Oh my!', err)
  })
  .on('close', function () {
    console.log('Stream closed')
  })
  .on('end', function () {
    console.log('Stream closed')
  })
```

The standard `pause()`, `resume()` and `destroy()` methods are implemented on the ReadStream, as is `pipe()` (see below). `'data'`, '`error'`, `'end'` and `'close'` events are emitted.

Additionally, you can supply an options object as the first parameter to `createReadStream()` with the following options:

* `'gt'` (greater than), `'gte'` (greater than or equal) define the lower bound of the range to be streamed. Only records where the key is greater than (or equal to) this option will be included in the range. When `reverse=true` the order will be reversed, but the records streamed will be the same.

* `'lt'` (less than), `'lte'` (less than or equal) define the higher bound of the range to be streamed. Only key/value pairs where the key is less than (or equal to) this option will be included in the range. When `reverse=true` the order will be reversed, but the records streamed will be the same.

* `'start', 'end'` legacy ranges - instead use `'gte', 'lte'`

* `'reverse'` *(boolean, default: `false`)*: a boolean, set true and the stream output will be reversed. Beware that due to the way LevelDB works, a reverse seek will be slower than a forward seek.

* `'keys'` *(boolean, default: `true`)*: whether the `'data'` event should contain keys. If set to `true` and `'values'` set to `false` then `'data'` events will simply be keys, rather than objects with a `'key'` property. Used internally by the `createKeyStream()` method.

* `'values'` *(boolean, default: `true`)*: whether the `'data'` event should contain values. If set to `true` and `'keys'` set to `false` then `'data'` events will simply be values, rather than objects with a `'value'` property. Used internally by the `createValueStream()` method.

* `'limit'` *(number, default: `-1`)*: limit the number of results collected by this stream. This number represents a *maximum* number of results and may not be reached if you get to the end of the data first. A value of `-1` means there is no limit. When `reverse=true` the highest keys will be returned instead of the lowest keys.

* `'fillCache'` *(boolean, default: `false`)*: wheather LevelDB's LRU-cache should be filled with data read.

* `'keyEncoding'` / `'valueEncoding'` *(string)*: the encoding applied to each read piece of data.

--------------------------------------------------------
<a name="createKeyStream"></a>
### db.createKeyStream([options])

A **KeyStream** is a **ReadStream** where the `'data'` events are simply the keys from the database so it can be used like a traditional stream rather than an object stream.

You can obtain a KeyStream either by calling the `createKeyStream()` method on a LevelUP object or by passing passing an options object to `createReadStream()` with `keys` set to `true` and `values` set to `false`.

```js
db.createKeyStream()
  .on('data', function (data) {
    console.log('key=', data)
  })

// same as:
db.createReadStream({ keys: true, values: false })
  .on('data', function (data) {
    console.log('key=', data)
  })
```

--------------------------------------------------------
<a name="createValueStream"></a>
### db.createValueStream([options])

A **ValueStream** is a **ReadStream** where the `'data'` events are simply the values from the database so it can be used like a traditional stream rather than an object stream.

You can obtain a ValueStream either by calling the `createValueStream()` method on a LevelUP object or by passing passing an options object to `createReadStream()` with `values` set to `true` and `keys` set to `false`.

```js
db.createValueStream()
  .on('data', function (data) {
    console.log('value=', data)
  })

// same as:
db.createReadStream({ keys: false, values: true })
  .on('data', function (data) {
    console.log('value=', data)
  })
```

--------------------------------------------------------
<a name="writeStreams"></a>
#### What happened to `db.createWriteStream`?

`db.createWriteStream()` has been removed in order to provide a smaller and more maintainable core. It primarily existed to create symmetry with `db.createReadStream()` but through much [discussion](https://github.com/level/levelup/issues/199), removing it was the best cause of action.

The main driver for this was performance. While `db.createReadStream()` performs well under most use cases, `db.createWriteStream()` was highly dependent on the application keys and values. Thus we can't provide a standard implementation and encourage more `write-stream` implementations to be created to solve the broad spectrum of use cases.

Check out the implementations that the community has already produced [here](https://github.com/level/levelup/wiki/Modules#write-streams).

--------------------------------------------------------
<a name='approximateSize'></a>
### db.db.approximateSize(start, end, callback)
<code>approximateSize()</code> can used to get the approximate number of bytes of file system space used by the range `[start..end)`. The result may not include recently written data.

```js
var db = require('level')('./huge.db')

db.db.approximateSize('a', 'c', function (err, size) {
  if (err) return console.error('Ooops!', err)
  console.log('Approximate size of range is %d', size)
})
```

**Note:** `approximateSize()` is available via [LevelDOWN](https://github.com/level/leveldown/), which by default is accessible as the `db` property of your LevelUP instance. This is a specific LevelDB operation and is not likely to be available where you replace LevelDOWN with an alternative back-end via the `'db'` option.


--------------------------------------------------------
<a name='getProperty'></a>
### db.db.getProperty(property)
<code>getProperty</code> can be used to get internal details from LevelDB. When issued with a valid property string, a readable string will be returned (this method is synchronous).

Currently, the only valid properties are:

* <b><code>'leveldb.num-files-at-levelN'</code></b>: returns the number of files at level *N*, where N is an integer representing a valid level (e.g. "0").

* <b><code>'leveldb.stats'</code></b>: returns a multi-line string describing statistics about LevelDB's internal operation.

* <b><code>'leveldb.sstables'</code></b>: returns a multi-line string describing all of the *sstables* that make up contents of the current database.


```js
var db = require('level')('./huge.db')
console.log(db.db.getProperty('leveldb.num-files-at-level3'))
// → '243'
```

**Note:** `getProperty()` is available via [LevelDOWN](https://github.com/level/leveldown/), which by default is accessible as the `db` property of your LevelUP instance. This is a specific LevelDB operation and is not likely to be available where you replace LevelDOWN with an alternative back-end via the `'db'` option.


--------------------------------------------------------
<a name="destroy"></a>
### leveldown.destroy(location, callback)
<code>destroy()</code> is used to completely remove an existing LevelDB database directory. You can use this function in place of a full directory *rm* if you want to be sure to only remove LevelDB-related files. If the directory only contains LevelDB files, the directory itself will be removed as well. If there are additional, non-LevelDB files in the directory, those files, and the directory, will be left alone.

The callback will be called when the destroy operation is complete, with a possible `error` argument.

**Note:** `destroy()` is available via [LevelDOWN](https://github.com/level/leveldown/) which you will have to install seperately, e.g.:

```js
require('leveldown').destroy('./huge.db', function (err) { console.log('done!') })
```

--------------------------------------------------------
<a name="repair"></a>
### leveldown.repair(location, callback)
<code>repair()</code> can be used to attempt a restoration of a damaged LevelDB store. From the LevelDB documentation:

> If a DB cannot be opened, you may attempt to call this method to resurrect as much of the contents of the database as possible. Some data may be lost, so be careful when calling this function on a database that contains important information.

You will find information on the *repair* operation in the *LOG* file inside the store directory.

A `repair()` can also be used to perform a compaction of the LevelDB log into table files.

The callback will be called when the repair operation is complete, with a possible `error` argument.

**Note:** `repair()` is available via [LevelDOWN](https://github.com/level/leveldown/) which you will have to install seperately, e.g.:

```js
require('leveldown').repair('./huge.db', function (err) { console.log('done!') })
```

--------------------------------------------------------

<a name="events"></a>
Events
------

LevelUP emits events when the callbacks to the corresponding methods are called.

* `db.emit('put', key, value)` emitted when a new value is `'put'`
* `db.emit('del', key)` emitted when a value is deleted
* `db.emit('batch', ary)` emitted when a batch operation has executed
* `db.emit('ready')` emitted when the database has opened (`'open'` is synonym)
* `db.emit('closed')` emitted when the database has closed
* `db.emit('opening')` emitted when the database is opening
* `db.emit('closing')` emitted when the database is closing

If you do not pass a callback to an async function, and there is an error, LevelUP will `emit('error', err)` instead.

<a name="json"></a>
JSON data
---------

You specify `'json'` encoding for both keys and/or values, you can then supply JavaScript objects to LevelUP and receive them from all fetch operations, including ReadStreams. LevelUP will automatically *stringify* your objects and store them as *utf8* and parse the strings back into objects before passing them back to you.

<a name="custom_encodings"></a>
Custom encodings
----------------

A custom encoding may be provided by passing in an object as an value for `keyEncoding` or `valueEncoding` (wherever accepted), it must have the following properties:

```js
{
    encode : function (val) { ... }
  , decode : function (val) { ... }
  , buffer : boolean // encode returns a buffer and decode accepts a buffer
  , type   : String  // name of this encoding type.
}
```

<a name="extending"></a>
Extending LevelUP
-----------------

A list of <a href="https://github.com/level/levelup/wiki/Modules"><b>Node.js LevelDB modules and projects</b></a> can be found in the wiki.

When attempting to extend the functionality of LevelUP, it is recommended that you consider using [level-hooks](https://github.com/dominictarr/level-hooks) and/or [level-sublevel](https://github.com/dominictarr/level-sublevel). **level-sublevel** is particularly helpful for keeping additional, extension-specific, data in a LevelDB store. It allows you to partition a LevelUP instance into multiple sub-instances that each correspond to discrete namespaced key ranges.

<a name="multiproc"></a>
Multi-process access
--------------------

LevelDB is thread-safe but is **not** suitable for accessing with multiple processes. You should only ever have a LevelDB database open from a single Node.js process. Node.js clusters are made up of multiple processes so a LevelUP instance cannot be shared between them either.

See the <a href="https://github.com/level/levelup/wiki/Modules"><b>wiki</b></a> for some LevelUP extensions, including [multilevel](https://github.com/juliangruber/multilevel), that may help if you require a single data store to be shared across processes.

<a name="support"></a>
Getting support
---------------

There are multiple ways you can find help in using LevelDB in Node.js:

 * **IRC:** you'll find an active group of LevelUP users in the **##leveldb** channel on Freenode, including most of the contributors to this project.
 * **Mailing list:** there is an active [Node.js LevelDB](https://groups.google.com/forum/#!forum/node-levelup) Google Group.
 * **GitHub:** you're welcome to open an issue here on this GitHub repository if you have a question.

<a name="contributing"></a>
Contributing
------------

LevelUP is an **OPEN Open Source Project**. This means that:

> Individuals making significant and valuable contributions are given commit-access to the project to contribute as they see fit. This project is more like an open wiki than a standard guarded open source project.

See the [contribution guide](https://github.com/Level/community/blob/master/CONTRIBUTING.md) for more details.

### Windows

A large portion of the Windows support comes from code by [Krzysztof Kowalczyk](http://blog.kowalczyk.info/) [@kjk](https://twitter.com/kjk), see his Windows LevelDB port [here](http://code.google.com/r/kkowalczyk-leveldb/). If you're using LevelUP on Windows, you should give him your thanks!


<a name="license"></a>
License &amp; copyright
-------------------

Copyright &copy; 2012-2015 **LevelUP** [contributors](https://github.com/level/community#contributors).

**LevelUP** is licensed under the MIT license. All rights not explicitly granted in the MIT license are reserved. See the included `LICENSE.md` file for more details.

=======
*LevelUP builds on the excellent work of the LevelDB and Snappy teams from Google and additional contributors. LevelDB and Snappy are both issued under the [New BSD Licence](http://opensource.org/licenses/BSD-3-Clause).*
