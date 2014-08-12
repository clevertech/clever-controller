Clever Controller
====================
###### Lightning-fast and flexible controller prototype
[![GitHub version](https://badge.fury.io/gh/cleverstack%2Fclever-controller.png)](http://badge.fury.io/gh/cleverstack%2Fclever-controller) [![Dependency Status](https://david-dm.org/CleverStack/clever-controller.png)](https://david-dm.org/CleverStack/clever-controller) [![devDependency Status](https://david-dm.org/CleverStack/clever-controller/dev-status.png)](https://david-dm.org/CleverStack/clever-controller#info=devDependencies) [![Code Climate](https://codeclimate.com/github/CleverStack/clever-controller.png)](https://codeclimate.com/github/CleverStack/clever-controller) 
[![Build Status](https://secure.travis-ci.org/CleverStack/clever-controller.png?branch=master)](https://travis-ci.org/CleverStack/clever-controller) 
[![Coverage](https://codeclimate.com/github/CleverStack/clever-controller/coverage.png)](https://codeclimate.com/github/CleverStack/clever-controller) 

![CleverStack Node Seed](http://cleverstack.github.io/assets/img/logos/node-seed-logo-clean.png "CleverStack Node Seed")

NodeJS Controller for the CleverStack ecosystem, designed to work with or without CleverStack.


## Important
###### If you are using the CleverStack framework you do not need to install or setup clever-controler, CleverStack comes out of the box with the Controller ready to use.
--


## Installation:
It is published in `npm` so a simple `npm install clever-controller` will suffice.
```
npm i clever-controller
```

## About

### Aim
The main aim of the controller is to help simplify the most common tasks that you need to do when setting up routes and functions/classes to handle them. 

### Description
Controllers are the thing that directly responds to each HTTP Request that comes into your application, as such each web request will result in (if routed) a new instance of a Controller (Class).

Avoid callback soup/hell and promise slowdown by using this.proxy() or this.callback() in the Static section of the Controller, this keeps code nesting to a minimum and brings code re-use up.
```
// Example, taken from the RoleController inside the clever-roles module
RoleService
    .list()
    .then( this.proxy( 'handleServiceMessage' ) )
    .catch( this.proxy( 'handleException' ) );

// You can also do
this.hello = 'YEAH!'; // Only here for demo
RoleService
    .list()
    .then( this.proxy( function( roles ) {
        console.log( this.hello ); // Will output 'YEAH!'
    }))
    .catch( this.proxy( 'handleException' ) );
```

## Features

### Performance 
By taking advantage of V8's Hidden Classes, Inline Caches and the Optimizing Compiler (which generates effecient machine code to use in-place of your javascript) your application is blisteringly fast AND easy to use.

### Highlights
* Lightning fast
    * V8's Hidden Classes & Inline Caches
    * V8's Optimizing Compiler which generates efficient machine code to run in place of your javascript
    * Clean, Efficient and Optimized code
* Super methods
    * Static inheritance
    * Prototypal inheritance
* Setup and initialization methods for both Static and Prototype
* Easy-to-use method proxys with currying ( available as Static.callback() and Prototype.proxy() )
* Internet Addressable functions, simply add 'Action' to the end of your controllers method() name and it will be Internet Addressable (as long as a route points to that controller).
* Private functions, any function without 'Action' on the end of its name is NOT Internet Addressable
* Easy accessors for Request and Response, available as this.req and this.res.

### Routing

1. Automatic Routing ( Static.autoRouting )
    * Ability to turn on or off autoRouting (true or array for ON, false for OFF)
    * Array notation allows (Connect or any other) Middleware to be attached to the routes that are autoRouted

here are some examples:
```
{
    // ...

    autoRouting: [
        // As a String (only available for methods on the controllers Static, either inherited or defined)
        'middlewareFromThisController',

        // Any function
        function definedFunction( req, res, next ) {
            // any function (from any file) that takes these three arguments can be attached
        },

        // CleverStack example
        PermissionController.requiresPermission({
            all: 'Example.$action', // optional: ability to define rules based on action (Example.post for POST, etc)
            postAction: false       // optional: ability to define overrides per action (Overrides must include 'Action')
        })
    ],

    middlewareFromThisController: function( req, res, next ) {
        // Attached inside of autoRouting array
        next();
    }

    // ...
}
```

2. Route name ( used with autoRouting )
    * Ability to either define your own route (name), or have it automatically generated based on the filename of your controller! (ExampleController would have a route name of /user and /users )

3. Manual Routing

You can define your own routes like this
```javascript
// Default route setup ~ '/example' or '/example/' or '/example/hello'
app.all('/example/?:action?', ExampleController.attach())

// Action + ID Routes setup ~ '/example/custom/12'
app.all('/example/:action/:id?', ExampleController.attach())

// A common use/place for middleware in a controller
app.use(ExampleController.someMiddleware);
```

**Note:** if you use both types of routes, be sure to place your routes in this order

We use Express' routing, so be sure to check it out at http://expressjs.com/api.html#app.VERB

## Making A Controller:

```javascript
module.exports = ExampleController = function() {
    return Controller.extend(
    {
        // example that returns JSON, available from route '/example/hello'
        helloAction: function() {
            this.send({
                message: 'Hi there'
            })
        },

        // example that renders a view, available from route '/example/view'
        viewAction: function() {
            this.render('view.ejs', {});
        }
    });
};
```

## Defining middleware
```javascript
module.exports = ExampleController = function() {
    return Controller.extend(
    {
        someMiddleware: function(req, res, next) {
            res.send({
                message: 'Hi from middleware!'
            });
        }
    },
    {
        // example that returns JSON, available from route '/example/hello'
        helloAction: function() {
            this.send({
                message: 'Hi there'
            })
        }
    });
};
```

## RESTful Actions

```javascript
module.exports = ExampleController = function() {
    return Controller.extend(
    {
        postAction: function() {
            this.send({
                status: 'Created record!' 
            });
        },

        listAction: function() {
            this.send({
                status: 'Sending you the list of examples.'
            });
        },

        getAction: function() {
            this.send({
                status: 'sending you record with id of ' + this.req.params.id
            });
        },

        putAction: function() {
            this.send({
                status: 'updated record with id ' + this.req.params.id
            });
        },

        deleteAction: function() {
            this.send({
                status: 'deleted record with id ' + this.req.params.id
            });
        }
    });
};
```

## Making Actions:

When doing a `GET /example` it will route to either `listAction` first OR `getAction` if listAction is not defined.

If neither are defined, express's `.next()` function will be called allowing it to fall through the controller and move onto any other middleware you may have configured.

If you want `/example/hello` as a route, you can simply implement `helloAction` in your controller and it will be automatically routed to it.

This is the default way to setup a controller to use actions. By default you can also visit `/example/12` and it will route to the `getAction` function in your controller (if it's defined) with `this.req.params.id` set for you to use (the same applies for all HTTP methods, eg PUT/DELETE/POST/et cetera)


## Performance (Tests folder 'performance-tests')
```
node test/performance/master.js
clever-controller: 1 server, 1 client processes: avg 2742 req/second (2726, 2758)
clever-controller: 2 server, 1 client processes: avg 2764 req/second (2784, 2745)
clever-controller: 3 server, 1 client processes: avg 2669 req/second (2671, 2668)
clever-controller: 4 server, 1 client processes: avg 2676 req/second (2688, 2664)
clever-controller: 5 server, 1 client processes: avg 2715 req/second (2773, 2661) *
raw express.js: 1 server, 1 client processes: avg 2766 req/second (2743, 2790)
raw express.js: 2 server, 1 client processes: avg 2793 req/second (2780, 2807)
raw express.js: 3 server, 1 client processes: avg 2730 req/second (2743, 2717)
raw express.js: 4 server, 1 client processes: avg 2710 req/second (2722, 2699)
raw express.js: 5 server, 1 client processes: avg 2635 req/second (2712, 2562) *
```

## Testing: 

```
npm test
```
