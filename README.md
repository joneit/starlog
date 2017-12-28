Flexible event logger

It is often useful for debugging purposes to log events.

### Synopsis

```js
var StarLog = require('starlog');
var options = {...};
var starlog = new StarLog(options);
starlog.start();
starlog.stop();
```

#### Sample output
If logging keyboard events for example, pressing a key between either of the above `logStart` and `logStop` pairs would log:
```
keydown
keypress
keyup
```

### Theory of operation

StarLog uses a simple dictionary of `starlogger` objects to add and remove event listeners for logging purposes.

These objects look like this (pseudocode type declaration):
```js
type starlogger = {
    listener: function,
    targets: object | object[]
}
```

A dictionary of such objects can be defined as follows:
* **Explicitly** by defining `options.loggers`; or
* **Implicitly** by defining an array in `options.events`, which is just a list of event strings; or
* **Automatically** by defining `options.pattern` which tells StarLog to look for matching event strings.

Armed with such a dictionary, StarLog's `start()` method turns logging on (by attaching all the
listeners to all their targets) and `stop()` turns them off (by removing all the listeners).

### Options

All options are optional as the name implies, with the exception that one of
`loggers`, `events`, or `pattern` (with `select`) _must_ be defined.

#### `options.loggers`

Specify an object whose keys represent a complete list of event strings:
```js
var options = {
    loggers: {
        keydown: starlogger,
        keyup: starlogger,
        keypress: starlogger
    }
});
```
The value of each key can be falsy or an object. The `listener` and `targets` properties are are subject to defaults
(see [`options.listenerDictionary`](#optionslistenerdictionary), [`options.listener`](#optionslistener),
[`options.targetsDictionary`](#optionstargetsdictionary), and [`options.targets`](#optionstargets)).

#### `options.events`

Specify an array containing a complete list of event strings:
```js
var options = {
    events: [
        'keydown',
        'keyup',
        'keypress'
    ]
};
```
This is transformed into a loggers object which is then subject to the same defaults.

#### `options.pattern` and `options.select`

Discover a list of event strings by looking through your code:
```js
var options = {
    pattern: /\.addEventListener\('[a-z-]+'\)/,
    select: myAPI // may also be an array of objects
};
```
> Note: Both options must be defined together.
> See also [`options.match.captureGroup`](https://github.com/joneit/code-match/blob/master/README.md#optionscapturegroup),
> useful for specifying which submatch to return.

This approach is limited to the _visible code_ in the getters, setters, and methods of the object(s) given in `options.select`.
See [`code-match`](https://github.com/joneit/code-match) for more information.

The resulting list of search hits is transformed into a loggers object which is then subject to the same defaults.

#### `options.listener`

To override the default logging listener (to prepend a timestamp, for example):
```js
options.listener = function(e) { exports.log((new Date).toISOString(), e.type);
```

Alternatively, reassign `Starlog.prototype.listener` directly, which would change the default for all subsequent instantiations.

#### `options.listenerDictionary`

To override the default logging listener for specific event strings:
```js
var options = {
    events: ['keydown', 'keyup', 'keypress'],
    listeners: {
        keypress: function(e) { exports.log('PRESSED!'); }
    }
};
// pressing a key would then log:
keydown
PRESSED!
keyup
```

#### `options.targets`

To specify (a) default event target(s):
```js
options.targets = document.querySelector('textarea'); // may also be an array of targets
```

Alternatively, reassign `Starlog.prototype.targets` directly, which would change the default for all subsequent instantiations.

#### `options.targetsDictionary`

To override the default event target(s) for specific event strings:
```js
var options = {
    events: ['keydown', 'keyup', 'keypress'],
    targets: {
        keypress: document.querySelector('textarea')
    }
};

#### `options.log`

Define this option to override the default logging function, `console.log.bind(console)`, which performs the actual output.

Alternatively, reassign `Starlog.prototype.log` directly, which would change the default for all subsequent instantiations.

#### `options.match`
This object is a rich set of options that controls how `code-match` looks through objects,
including whitelists and blacklists for member names and string matches.
See [`code-match` options](https://github.com/joneit/code-match/blob/master/README.md#options) for details.
