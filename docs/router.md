###Finger
Request routing provided by [finger](https://www.npmjs.org/package/finger) module

In ```fist``` application, route is a relation of request pattern and unit. All you need to know about router is that you can add route in application using ```app.route``` method and how to write request patterns in ```finger``` terminology.

Router patterns is just strings written according to some syntax rules.

####Syntax
```js
// just static pattern
app.route('/about/', 'about-page');
// route with parameter
app.route('/profile/<userId>/', 'user-profile-page');
// route with optional part
app.route('/news/(<postId>/)', 'newsPage');
```

It's all but there are some features.

####Request method specifying
Patterns may require request method (```GET``` by default if not specified)
```js
app.route('POST /upload/');
// and you can specify more than one method
app.route('POST,PUT /news/<postId>');
```

If ```GET``` method is specified, ```HEAD``` will be specified automatically too for this route.

####Flags
Router supports some ```Boolean``` parameters that you can specify while creating app.
```js
var app = fist({
    router: {
        ignoreCase: true,
        doNotMatchStart: true,
        doNotMatchEnd: true
    }
});
```

All of these parameters is ```false``` by default. Setting parameters in router's  constructor will affect to all patterns, but it is possible to set some of them directly in pattern.

Patterns supports a little parameters known as flags. Every flag is related to some parameter. 
* ```i``` is ```ignoreCase```
* ```s``` is ```doNotMatchStart```
* ```e``` is ```doNotMatchEnd```

```js
app.route('GET /YoUsTrAnGe/ i', 'my-controller');
```

And it is possible to specify more than one flag
```js
app.route('GET /<imageName>.png si');
```

Flag is a single character in upper or lower case. Lowercased character sets up the related parameter in ```true``` but uppercased sets it up in ```false```.

```js
var app = fist({
    router: {
        // ignore case for all requests by default
        ignoreCase: true
    }
});

// disable case insensitivity for this route
app.route('GET /lower/ I', 'some-controller');
```

Let's explain parameters meanings
#####```ignoreCase```
This option enables url characters case ignoring
```js
app.route('/YoUAreStranGe/ i', 'some-page');
```

For this route the all urls below are valid
```
/youarestrange/
/YOUARESTARNGE/
/YoUAreStrAngE/
```
Easy.
#####```doNotMatchStart```
Not so usable parameter but sometimes is needed. This option enables the behaviour when start of url not needed to match with start of pattern.
```js
app.route('GET .png s', 'png-share');
```
All the urls that ends with ```.png``` will be matched to this pattern.
```
/i/bus.png
/static/images/sea.png
/background.png
```

#####```doNotMatchEnd```
More usable option than previous. Like ```doNotMatchStart``` but vice versa.
```js
app.route('/admin/ e', 'checkAdminRights');
```

In case above all the urls starting with /admin/ will be matched on first route.

###Routing
Usually, the function associated with pattern is controller that must send data to client. But ```fist``` units MAY send data or allow router to try next pattern. Units may have dynamic behaviour.
An application expects to some controlling actions (return of special object) from unit, related to pattern. But if these will no, it will continue to match remaining patterns.

```js
app.route('/admin/ e', 'checkAdminRights'); // may send 403 or do nothing if the user is admin
app.route('/admin/pane/', 'someRestrictedStuff1');
app.route('/admin/explode-the-world/', 'someRestrictedStuff2');
```
