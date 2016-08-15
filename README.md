# Notes

这个项目我是从 branch origin/5.x 开始看的, 这个branch是为了支持koa1.x版本，里边用了大量的GeneratorFunction
。导致代码逻辑比较难读。但是由于我是先看的这个branch，所以所有的标注都加在了这个branch里边。
master branch里边的代码相对来看就清晰多了, 由于是koa2.x的语法, 里边没有GeneratorFunction。这里边所有类的方法
基本上和 branch 5.x 的一致。

这个项目主要的类 & 比较重要的方法: 

    class Layer
        param()

    class Router
        routes()
        register()

这个项目还有一个比较有意思的地方就是应用了 jsdoc2md = require('jsdoc-to-markdown'); 这个库来生成项目api文档。
值得参考。



# koa-router

[![NPM version](https://img.shields.io/badge/npm-v7.0.1-blue.svg?style=flat)](https://npmjs.org/package/koa-router) [![NPM Downloads](https://img.shields.io/npm/dm/koa-router.svg?style=flat)](https://npmjs.org/package/koa-router) [![Node.js Version](https://img.shields.io/node/v/koa-router.svg?style=flat)](http://nodejs.org/download/) [![Build Status](http://img.shields.io/travis/alexmingoia/koa-router.svg?style=flat)](http://travis-ci.org/alexmingoia/koa-router) [![Tips](https://img.shields.io/gratipay/alexmingoia.svg?style=flat)](https://www.gratipay.com/alexmingoia/) [![Gitter Chat](https://img.shields.io/badge/gitter-join%20chat-1dce73.svg?style=flat)](https://gitter.im/alexmingoia/koa-router/)

> Router middleware for [koa](https://github.com/koajs/koa)

* Express-style routing using `app.get`, `app.put`, `app.post`, etc.
* Named URL parameters.
* Named routes with URL generation.
* Responds to `OPTIONS` requests with allowed methods.
* Support for `405 Method Not Allowed` and `501 Not Implemented`.
* Multiple route middleware.
* Multiple routers.
* Nestable routers.
* ES7 async/await support.

## Migrating to 7 / Koa 2

- The API has changed to match the new promise-based middleware
  signature of koa 2. See the
  [koa 2.x readme](https://github.com/koajs/koa/tree/2.0.0-alpha.3) for more
  information.
- Middleware is now always run in the order declared by `.use()` (or `.get()`,
  etc.), which matches Express 4 API.

## Installation

Until koa 2 is released, you must specify `^7.0.0` or use the `next` tag:

```sh
npm install koa-router@next
```

When koa 2 is released, 7.x will be tagged with `latest` and installed by
default by NPM.

## API Reference

* [koa-router](#module_koa-router)
  * [Router](#exp_module_koa-router--Router) ⏏
    * [new Router([opts])](#new_module_koa-router--Router_new)
    * _instance_
      * [.get|put|post|patch|delete](#module_koa-router--Router+get|put|post|patch|delete) ⇒ <code>Router</code>
      * [.routes](#module_koa-router--Router+routes) ⇒ <code>function</code>
      * [.use([path], middleware, [...])](#module_koa-router--Router+use) ⇒ <code>Router</code>
      * [.prefix(prefix)](#module_koa-router--Router+prefix) ⇒ <code>Router</code>
      * [.allowedMethods([options])](#module_koa-router--Router+allowedMethods) ⇒ <code>function</code>
      * [.redirect(source, destination, code)](#module_koa-router--Router+redirect) ⇒ <code>Router</code>
      * [.route(name)](#module_koa-router--Router+route) ⇒ <code>Layer</code> &#124; <code>false</code>
      * [.url(name, params)](#module_koa-router--Router+url) ⇒ <code>String</code> &#124; <code>Error</code>
      * [.param(param, middleware)](#module_koa-router--Router+param) ⇒ <code>Router</code>
    * _static_
      * [.url(path, params)](#module_koa-router--Router.url) ⇒ <code>String</code>

<a name="exp_module_koa-router--Router"></a>
### Router ⏏
**Kind**: Exported class
<a name="new_module_koa-router--Router_new"></a>
#### new Router([opts])
Create a new router.


| Param | Type | Description |
| --- | --- | --- |
| [opts] | <code>Object</code> |  |
| [opts.prefix] | <code>String</code> | prefix router paths |

**Example**
Basic usage:

```javascript
var app = require('koa')();
var router = require('koa-router')();

router.get('/', function (ctx, next) {...});

app
  .use(router.routes())
  .use(router.allowedMethods());
```
<a name="module_koa-router--Router+get|put|post|patch|delete"></a>
#### router.get|put|post|patch|delete ⇒ <code>Router</code>
Create `router.verb()` methods, where *verb* is one of the HTTP verbs such
as `router.get()` or `router.post()`.

Match URL patterns to callback functions or controller actions using `router.verb()`,
where **verb** is one of the HTTP verbs such as `router.get()` or `router.post()`.

```javascript
router
  .get('/', function (ctx, next) {
    ctx.body = 'Hello World!';
  })
  .post('/users', function (ctx, next) {
    // ...
  })
  .put('/users/:id', function (ctx, next) {
    // ...
  })
  .del('/users/:id', function (ctx, next) {
    // ...
  });
```

Route paths will be translated to regular expressions used to match requests.

Query strings will not be considered when matching requests.

#### Named routes

Routes can optionally have names. This allows generation of URLs and easy
renaming of URLs during development.

```javascript
router.get('user', '/users/:id', function (ctx, next) {
 // ...
});

router.url('user', 3);
// => "/users/3"
```

#### Multiple middleware

Multiple middleware may be given:

```javascript
router.get(
  '/users/:id',
  function (ctx, next) {
    return User.findOne(ctx.params.id).then(function(user) {
      ctx.user = user;
      return next();
    });
  },
  function (ctx) {
    console.log(ctx.user);
    // => { id: 17, name: "Alex" }
  }
);
```

### Nested routers

Nesting routers is supported:

```javascript
var forums = new Router();
var posts = new Router();

posts.get('/', function (ctx, next) {...});
posts.get('/:pid', function (ctx, next) {...});
forums.use('/forums/:fid/posts', posts.routes(), posts.allowedMethods());

// responds to "/forums/123/posts" and "/forums/123/posts/123"
app.use(forums.routes());
```

#### Router prefixes

Route paths can be prefixed at the router level:

```javascript
var router = new Router({
  prefix: '/users'
});

router.get('/', ...); // responds to "/users"
router.get('/:id', ...); // responds to "/users/:id"
```

#### URL parameters

Named route parameters are captured and added to `ctx.params`.

```javascript
router.get('/:category/:title', function (ctx, next) {
  console.log(ctx.params);
  // => { category: 'programming', title: 'how-to-node' }
});
```

The [path-to-regexp](https://github.com/pillarjs/path-to-regexp) module is used
to convert paths to regular expressions.

**Kind**: instance property of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type | Description |
| --- | --- | --- |
| path | <code>String</code> |  |
| [middleware] | <code>function</code> | route middleware(s) |
| callback | <code>function</code> | route callback |

<a name="module_koa-router--Router+routes"></a>
#### router.routes ⇒ <code>function</code>
Returns router middleware which dispatches a route matching the request.

**Kind**: instance property of <code>[Router](#exp_module_koa-router--Router)</code>
<a name="module_koa-router--Router+use"></a>
#### router.use([path], middleware, [...]) ⇒ <code>Router</code>
Use given middleware.

Middleware run in the order they are defined by `.use()`. They are invoked
sequentially, requests start at the first middleware and work their way
"down" the middleware stack.

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type |
| --- | --- |
| [path] | <code>String</code> |
| middleware | <code>function</code> |
| [...] | <code>function</code> |

**Example**
```javascript
// session middleware will run before authorize
router
  .use(session())
  .use(authorize());

// use middleware only with given path
router.use('/users', userAuth());

// or with an array of paths
router.use(['/users', '/admin'], userAuth());

app.use(router.routes());
```
<a name="module_koa-router--Router+prefix"></a>
#### router.prefix(prefix) ⇒ <code>Router</code>
Set the path prefix for a Router instance that was already initialized.

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type |
| --- | --- |
| prefix | <code>String</code> |

**Example**
```javascript
router.prefix('/things/:thing_id')
```
<a name="module_koa-router--Router+allowedMethods"></a>
#### router.allowedMethods([options]) ⇒ <code>function</code>
Returns separate middleware for responding to `OPTIONS` requests with
an `Allow` header containing the allowed methods, as well as responding
with `405 Method Not Allowed` and `501 Not Implemented` as appropriate.

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type | Description |
| --- | --- | --- |
| [options] | <code>Object</code> |  |
| [options.throw] | <code>Boolean</code> | throw error instead of setting status and header |
| [options.notImplemented] | <code>Function</code> | throw throw the returned value in place of the default NotImplemented error |
| [options.methodNotAllowed] | <code>Function</code> | throw the returned value in place of the default MethodNotAllowed error |

**Example**
```javascript
var app = koa();
var router = router();

app.use(router.routes());
app.use(router.allowedMethods());

```
**Example with [Boom](https://github.com/hapijs/boom)**
```javascript
var app = koa();
var router = router();
var Boom = require('boom');

app.use(router.routes());
app.use(router.allowedMethods({
  throw: true,
  notImplemented: () => new Boom.notImplemented(),
  methodNotAllowed: () => new Boom.methodNotAllowed()
}));
```
<a name="module_koa-router--Router+redirect"></a>
#### router.redirect(source, destination, code) ⇒ <code>Router</code>
Redirect `source` to `destination` URL with optional 30x status `code`.

Both `source` and `destination` can be route names.

```javascript
router.redirect('/login', 'sign-in');
```

This is equivalent to:

```javascript
router.all('/login', function (ctx) {
  ctx.redirect('/sign-in');
  ctx.status = 301;
});
```

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type | Description |
| --- | --- | --- |
| source | <code>String</code> | URL or route name. |
| destination | <code>String</code> | URL or route name. |
| code | <code>Number</code> | HTTP status code (default: 301). |

<a name="module_koa-router--Router+route"></a>
#### router.route(name) ⇒ <code>Layer</code> &#124; <code>false</code>
Lookup route with given `name`.

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type |
| --- | --- |
| name | <code>String</code> |

<a name="module_koa-router--Router+url"></a>
#### router.url(name, params) ⇒ <code>String</code> &#124; <code>Error</code>
Generate URL for route. Takes either map of named `params` or series of
arguments (for regular expression routes).

```javascript
router.get('user', '/users/:id', function (ctx, next) {
 // ...
});

router.url('user', 3);
// => "/users/3"

router.url('user', { id: 3 });
// => "/users/3"
```

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type | Description |
| --- | --- | --- |
| name | <code>String</code> | route name |
| params | <code>Object</code> | url parameters |

<a name="module_koa-router--Router+param"></a>
#### router.param(param, middleware) ⇒ <code>Router</code>
Run middleware for named route parameters. Useful for auto-loading or
validation.

**Kind**: instance method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type |
| --- | --- |
| param | <code>String</code> |
| middleware | <code>function</code> |

**Example**
```javascript
router
  .param('user', function (id, ctx, next) {
    ctx.user = users[id];
    if (!ctx.user) return ctx.status = 404;
    return next();
  })
  .get('/users/:user', function (ctx) {
    ctx.body = ctx.user;
  })
  .get('/users/:user/friends', function (ctx) {
    return ctx.user.getFriends().then(function(friends) {
      ctx.body = friends;
    });
  })
  // /users/3 => {"id": 3, "name": "Alex"}
  // /users/3/friends => [{"id": 4, "name": "TJ"}]
```
<a name="module_koa-router--Router.url"></a>
#### Router.url(path, params) ⇒ <code>String</code>
Generate URL from url pattern and given `params`.

**Kind**: static method of <code>[Router](#exp_module_koa-router--Router)</code>

| Param | Type | Description |
| --- | --- | --- |
| path | <code>String</code> | url pattern |
| params | <code>Object</code> | url parameters |

**Example**
```javascript
var url = Router.url('/users/:id', {id: 1});
// => "/users/1"
```
## Contributing

Please submit all issues and pull requests to the [alexmingoia/koa-router](http://github.com/alexmingoia/koa-router) repository!

## Tests

Run tests using `npm test`.

## Support

If you have any problem or suggestion please open an issue [here](https://github.com/alexmingoia/koa-router/issues).
