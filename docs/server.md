###API
####```Server fist([Object config[, Object members[, Object statics]]])```
Use ```fist()``` factory to create an application instance
```js
var app = fist();
```
####```Object app.params```
Shallow copy of object passed to ```fist``` factory
```js
var app = fist({
    res: {
        hideStackTraces: true
    }
});

assert.deepEqual(app.params.res, {
    hideStackTraces: true
});
```
####```Server app.route(String pattern, Object|String unit)```
Add route to application
```js
app.route('GET /', 'unit-name-as-route-name');
app.route('GET /', {
    name: 'route-name',
    unit: 'unit-name'
});
```
####```Function app.getHandler()```
Returns request handler(req, res) to use with custom server ([http.Server](http://nodejs.org/api/http.html#http_class_http_server) by default)
```js
var serv = new CustomServer(app.getHandler())
```
####```http.Server app.listen()```
Start listening any socket or port you want, see [http.Server.prototype.listen](http://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback)
```js
app.listen(1337);
```
####```Server app.include([String pattern[, String pattern...]])```
Require files by given pattern[s] and set it as plugins
```
./my-plugins/
    ./sleep.js
```

```js
//sleep.js
module.exports = function (done) {
    setTimeout(function () {
        done();
    }, 1000);
}
```
```js
app.include('./my-plugins/**/*.js');
```
####```Server app.plug([* plugin[, *plugin]])```
Set plugin[s]
```js
app.plug(function (done) {
    setTimeout(function () {
        done();
    }, 1000)
});
```
####```Server app.unit(members[, statics])```
Register new application unit. Pass members as unit own members and statics as static members.
Special members attributes:
#####```String path```
Unit's identifier
#####```String base```
Name of base unit to inherit from
#####```Function|Array<Function> mix```
Unit's mixins

All inheritance magic provided by [inherit](https://www.npmjs.org/package/inherit) module
```js
app.unit({
    base: 'pet',
    path: 'cat',
    mix: [Voice],
    voice: function () {
        return 'meow';
    }
});

```
####```Server app.alias(String base, String path)```
Just inherit from ```<base>``` unit and register it as ```<path>```
```js
app.unit({
    path: 'fist-contrib-so-crazy-plugin-name'
});
app.alias('fist-contrib-so-crazy-plugin-name', 'new-name');
```
####```Server app.alias(Object aliases)```
Same but another sugnature. Supports many aliases at one time.
```js
app.unit({
    path: 'fist-contrib-so-crazy-plugin-name-1'
});
app.unit({
    path: 'fist-contrib-so-crazy-plugin-name-2'
});
app.alias({
    'fist-contrib-so-crazy-plugin-name-1': 'new-name-1',
    'fist-contrib-so-crazy-plugin-name-2': 'new-name-2'
});
```
####```vow.Promise app.ready()```
Returns a promise that will be resolved when application ready
```js
app.ready().then(function () {
    console.log('I am ready!');
});
```
####```Unit app.getUnit(String path)```
Returns registered ```Unit``` object by ```path```. Note that units instances available after application ready.
```js
app.unit({
    path: 'foo',
    test: 42
});
app.ready().then(function() {
    assert.strictEqual(app.getUnit('foo').test, 42);
})
```
####```app.channel(String name)```
Creates the new event-channel if not exist or uses existing
```js
var sys = app.channel('sys');
sys.on('request', function (event) {
    log(event);
});
```
###Events
As mentioned in introdiction app is an ```EventEmitter``` extended with channels.
####```sys@```
```sys``` is an application server channel
#####```pending()```
Application initialization in progress
#####```ready()```
Application ready
#####```eready(* err)```
Application initialization failed
#####```request(Track track)```
Http request given
#####```match(Track track)```
Router has succesfully matched request to one of patterns
#####```ematch(Track track)```
There are no suitable patterns
#####```response(Track track)```
Application has responded to client

####```ctx@```
```ctx``` is unit's calling context-binded event channel. The events has the same interface
######```event.path```
Target unit
######```event.time```
Time passed from unit calling to event triggering
######```event.data```
Some binded data
######```event.trackId```
```Track``` id

#####```pending(Object event)```
Application begins unit resolution
#####```accept(Object event)```
Application resolved the unit with result
#####```reject(Object event)```
Application resolved the unit with error

