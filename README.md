gexp.js
=======

[![Build Status](https://travis-ci.org/expansejs/gexp.js.svg)](https://travis-ci.org/expansejs/gexp.js)

Start and stop [gexp](https://github.com/ethereum/go-expanse) from Node.js.

Usage
-----

```
$ npm install gexp
```
To use gexp.js, simply require it:
```javascript
var gexp = require("gexp");
```

### Starting and stopping gexp

gexp's `start` method accepts a configuration object, which uses the same flags as the gexp command line client.  (Here, the flags are organized into an object.)  Flags that are not accompanied by a value on the command line (for example, `--mine`) should be passed in as `{ flag: null }`.
```javascript
var options = {
    networkid: "10102",
    port: 42786,
    rpcport: 9656,
    mine: null
};

gexp.start(options, function (err, proc) {
    if (err) return console.error(err);
    // get your gexp on!
});
```
The callback's parameter `proc` is the child process, which is also attached to the `gexp` object (`gexp.proc`) for your convenience.

When you and gexp have had enough lively times, `stop` kills the underlying gexp process:
```javascript
gexp.stop(function () {
    // no more lively times :( 
});
```

### Listeners

The callback for `start` fires after gexp has successfully started.  Specifically, it looks for `"IPC service started"` in gexp's stderr.  If you want to change the start callback trigger to something else, this can be done by replacing gexp's default listeners.  `gexp.start` accepts an optional second parameter which should specify the listeners to overwrite, for example:
```javascript
{
    stdout: function (data) {
        process.stdout.write("I got a message!! " + data.toString());
    },
    stderr: function (data) {
        if (data.toString().indexOf("Protocol Versions") > -1) {
            gexp.trigger(null, gexp.proc);
        }
    },
    close: function (code) {
        console.log("It's game over, man!  Game over!");
    }
}
```
In the above code, `gexp.trigger` is the callback passed to `gexp.start`.  (This callback is stored so that the startup trigger can be changed if needed.)  These three listeners (`stdout`, `stderr`, and `close`) are the only listeners which can be specified in this parameter, since only these three are set by default in `gexp.start`.

If you want to swap out or add other listeners (after the initial startup), you can use the `gexp.listen` convenience method:
```javascript
gexp.listen("stdout", "data", function (data) { process.stdout.write(data); });
```
This example (re-)sets the "data" listener on stdout by setting `gexp.proc.stdout._events.data = function (data) { process.stdout.write(data); }`.

Tests
-----

gexp.js's tests use [ethrpc](https://github.com/expansejs/ethrpc) to send some basic requests to gexp.
```
$ npm test
```
