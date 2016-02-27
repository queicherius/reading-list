# Serializing big objects

- **Requirement**: 
  - Serialize a big (100mb) javascript object into a string using node (for saving into redis)
  - Blocking the event loop while serializing is okay, while deserializing is NOT.
- **`JSON`**
  - :+1: Native, no module needed
  - :+1: Pretty fast (~1s)
  - :-1: Blocking the event loop
- **[`msgpack`](https://www.npmjs.com/package/msgpack)**
  - :-1: Slow (about 5 times slower than JSON)
  - :-1: Blocking the event loop
- **[`BSON`](https://www.npmjs.com/package/bson)**
  - :-1: Doesnt handle objects bigger than 17mb in the JS version
  - :-1: Blocking the event loop
- **[`Response.json()`](http://azimi.me/2015/07/30/non-blocking-async-json-parse.html?utm_source=javascriptweekly&utm_medium=email)**
  - :-1: Not available in node.js and polyfills use native JSON anyway
- **[`json-parse-stream`](https://www.npmjs.com/package/json-parse-stream)**
  - :-1: EXTREMELY slow (300+ times slower than JSON)
  - :-1: Overhead of building the object again
  - :-1: Blocking the event loop
- **[`jsonparse`](https://github.com/creationix/jsonparse)**
  - :-1: Slow (about 30 times slower than JSON)
  - :-1: Even tho its streaming, it blocks the event loop
- **WebWorkers / Threads**
  - :-1: For messaging in between threads (which workers essentially are), we need serialize / unserialize the object AGAIN - just increasing our workload
- **[`json-streams`](https://github.com/Floby/node-json-streams.git)**
  - :-1: Seems to be not supported on Node 5.x, tests fail
- **[`node-ffi`](https://github.com/node-ffi/node-ffi) with a C library**
  - *TODO*

--

```js
// Some random code to check if the event loop is blocked or not
// Should print dots continously into the console
;(function spinForever () {
  process.stdout.write(".");
  process.nextTick(spinForever);
})();
```
