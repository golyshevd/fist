fist [![NPM version](https://badge.fury.io/js/fist.svg)](http://badge.fury.io/js/fist) [![Build Status](https://travis-ci.org/fistlabs/fist.png)](https://travis-ci.org/fistlabs/fist) [![Dependency Status](https://david-dm.org/fistlabs/fist.svg)](https://david-dm.org/fistlabs/fist) [![devDependency Status](https://david-dm.org/fistlabs/fist/dev-status.svg)](https://david-dm.org/fistlabs/fist#info=devDependencies)
=========
```fist``` is a [node.js](http://nodejs.org/) framework for creating server applications. ```fist``` offers you an architecture, for which support and extension is easy both for simple and complex web-servers.

Features
---------
 * Powerful routing
 * Modular structure (inheritance, mixins, explicit dependencies)
 * Low architecture coupling
 * Transparent async initialization
 * High level abstraction
 * Plugin interface
 * Thin controllers

System requirements
---------
* node >= 0.10

Installation
---------
```
$ npm install fist
```

Usage
---------
```js
'use strict';

var app = require('fist')();

app.unit({
    pattern: '/',
    data: function (track) {
        return track.send('Hello fist!');
    }
});

app.listen(1337);
```
```
$ curl localhost:1337
Hello fist!
```
See the full [documentation](docs/index.md)

License
---------
[MIT](LICENSE)
