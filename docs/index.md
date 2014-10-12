How it works
---------

![how it works](i/scheme0.png)

Oh. My. GOD! What is the crazy image!?
Calm down. This is easy. Let me to explain this scheme.

First, ```fist``` application object (```app``` below) is an http server. ```app``` has an ```Observable``` behavior that is used to broadcast any application events. ```app``` can receive incoming requests and respond to client. ```app``` object is a ```FIST``` square on the scheme.

```js
var app = fist();

app.on('some-event', function (event) {
    console.log(event);
});
```

When an application starts, webserver immediately starts listening socket or port you specified (defaults are taken from ```http``` module), and async initialization procedure begins. 'acync' means that the```app``` will defer any incoming requests until ```app``` is ready, and then it will handle them.

Example of the application long start:

```js
app.plug(function (done) {
    doSomethingForALongTimeBeforeStart(function () {
        done();
    });
});
```

OK, our ```app``` is ready to work, and we've received our first request.

When ```app``` receives a request, it begins routing procedure. In our diagram router depicted as a sequence of the squares marked like ```route + A```. ```Route```, which is the combination of request pattern and related static data, by itself is a part of ```Router```. The ```Router``` iterates over its routes and checks received request path for matching. If ```Router``` doesn't have matched routes, application responds, that resourse ```Not Found```. But if matched route is found, ```app``` starts resolving procedure.

Routes' declaration example:
```js
app.route('/', 'index-page');
app.route('/forum', 'forum-page');

//Route with parameter
app.route('/avatars/<imageId>.png', 'avatar-image');

//  "e" flag means prefix matching
// like url.match(/^\/admin\//)
//  instead of url.match(/^\/admin\/$/)
app.route('/admin/ e', 'admin-check');
app.route('/admin/pane', 'admin-pane-page');
```

As mentioned above, route is a pattern related to some data. In our case it is a ```Unit``` reference. In our diagram units are depicted as circles. ```Unit``` is a application logical unit that implements one of it's 'atom' part's. It could be kind of controller, wich handles client request, or model, which sends request to backend, or some helper, for example, which performs server-side form validating.

Example of contoller unit:
```js
app.unit({
    path: 'index-page',
    data: function (track) {
        return track.send(200, '<h1>Index</h1>');
    }
});
```

How do ```app``` resolves units? Before ```app``` begins resolving procedure, it creates a ```Track``` object. ```Track``` is a current request context, which contains API for dealing with request and response objects. For example, getting request headers, setting response headers and much more. ``Track``` instance for particular request is available during the whole execution of the resolution procedure.

```Unit``` that directly associated with request pattern widely known as controller. :)

```js
app.unit({
    path: 'avater-image',
    data: function (track, context) {
        var image = /* @type {Promise} */ getImageAnyWhere(context.arg('imageId'));
        track.header('Content-Type', 'image/png');
        return track.send(image);
    }
})
```

```Units``` can declare dependencies. Firstly,  ```app``` resolves ```Unit's``` dependencies, then ```Unit''' itself. It's result can depend on other ```Units'``` results. It is an explicit dependencies feature. For each ```Unit``` call, ```app``` creates an object ```Context``` and passes it to ```Unit```. ```Context``` is an unique ```Unit```'s calling context. It contains '''Unit``` dependencies' resolution results, call parameters and other special API.

Example of ``Unit``` with dependency:
```js
app.unit({
    path: 'sessionid',
    data: function (track) {
        return getSomeStuff(track.cookie('Session_id'));
    }
});

app.unit({
    path: 'forum-page',
    deps: ['sessionid'],
    data: function (track, context) {
        if ( context.getRes('sessionid.userId') ) {
            return track.send('<h1>Forum</h1>');
        }
        return track.send(403, 'Login first!');
    }
})

```

When ```app``` gets result from the controller it checks resolution status. If result status is ```rejected``` then ```app``` immediately responds with error and contoller status within response body. But if status is ```accepted```, ```app``` will check if track returns something. If yes, ```app``` will immedialely respond with controller return value and status code. But if not, ```app``` will continue routing procedure until next matching controller returns something.

Example with sending error from dependency:
```js
app.unit({
    path: 'admin-check',
    deps: ['sessionid'],
    data: function (track, context) {
        if ( isAdmin(context.getRes('sessionid')) ) {
            //  continue matching remaining routes
            return;
        }
        return track.send(403, 'You are not admin!');
    }
});

app.unit({
    page: 'admin-pane-page',
    data: function (track, context) {
        // you don't need check rights because of under admin-check controller
        return track.send('<h1>Admin pane</h1>');
    }
});
```

Note that not only controller could return controlling objects which tells to application "Hey, you must respond to client right now!". But any unit. If some dependency returns control object, application will stop resolution procedure and will respond.

```js
var app = fist();

app.route('/', 'page');

app.unit({
    path: 'page',
    deps: ['rebel'],
    data: function (track, context) {
        return track.send(200, 'Hello, fist!');
    }
});

app.unit({
    path: 'rebel',
    data: function (track, context) {
        return track.send(500, 'Ooops!');
    }
});

```

This application will respond ```500 Ooops!``` and controller will never execute.

That's all! Easy!

API Reference
---------
* [Application](server.md)
* [Router](router.md)
* [Unit](unit.md)
* [Track](track.md)
* [Context](context.md)
* [Plugins](plugins.md)
