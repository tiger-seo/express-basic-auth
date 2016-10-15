# express-basic-auth

Simple plug & play HTTP basic auth middleware for Express.

## How to install

Just run

```shell
npm install express-basic-auth
```

add the `--save` option to add it to the `dependencies` in your `package.json` as well

## How to use

The module will export a function, that you can call with an options object to
get the middleware:

```js
var app = require('express')();
var basicAuth = require('express-basic-auth');

app.use(basicAuth({
    users: { 'admin': 'supersecret' }
}));
```

The middleware will now check incoming requests to match the credentials
`admin:supersecret`.

The middleware will check incoming requests for a basic auth (`Authorization`)
header, parse it and check if the credentials are legit.

**If a request is found to not be authorized**, it will respond with HTTP 401 and an empty body.

**If a request is successfully authorized**, an `auth` property will be added to the request,
containing an object with `user` and `password` properties, filled with the credentials.

### Static Users

If you simply want to check basic auth against one or multiple static credentials,
you can pass those credentials as in the example above:

```js
app.use(basicAuth({
    users: {
        'admin': 'supersecret',
        'adam': 'password1234',
        'eve': 'asdfghjkl'
    }
}));
```

The middleware will check incoming requests to have a basic auth header matching
one of the three passed credentials.

### Custom authorization

Alternatively, you can pass your own `authorizer` function, to check the credentials
however you want. It will be called with a username and password and is expected to
return `true` or `false` to indicate that the credentials were approved or not:

```js
app.use(basicAuth( { authorizer: myAuthorizer } ));

function myAuthorizer(username, password) {
    return username.startsWith('A') && password.startsWith('secret');
}
```

This will authorize all requests with credentials where the username begins with
`'A'` and the password begins with `'secret'`. In an actual application you would
likely look up some data instead ;-)

### Custom Async Authorization

Note that the `authorizer` function above is expected to be synchronous. This is
the default behavior, you can pass `authorizeAsync: true` in the options object to indicate
that your authorizer is asynchronous. In this case it will be passed a callback
as the third parameter, which is expected to be called by standard node convention
with an error and a boolean to indicate if the credentials have been approved or not.
Let's look at the same authorizer again, but this time asynchronous:

```js
app.use(basicAuth({
    authorizer: myAsyncAuthorizer,
    authorizeAsync: true
}));

function myAsyncAuthorizer(username, password, cb) {
    if(username.startsWith('A') && password.startsWith('secret'))
        return cb(null, true);
    else
        return cb(null, false)
}
```

### Challenge

Per default the middleware will not add a `WWW-Authenticate` challenge header to
responses of unauthorized requests. You can enable that by adding `challenge: true`
to the options object. This will cause most browsers to show a popup to enter credentials
on unauthorized responses:

```js
app.use(basicAuth({
    users: { 'someuser': 'somepassword' },
    challenge: true
}));
```

## Try it

The repository contains an `example.js` that you can run to play around and try
the middleware. To use it just put it somewhere (or leave it where it is), run

```shell
npm install express express-basic-auth
node example.js
```

This will start a small express server listening at port 8080. Just look at the file,
try out the requests and play around with the options.

## To Do

- Allow customization of unauthorized response body
- Allow to set a realm for the challenge
- Some kind of automated testing with the example server
- Maybe add some optional callback to be called for unauthorized requests (for security logging)
- Decide what should be included in `1.0.0`
