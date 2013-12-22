## What is Beefy?

Beefy is a static web server with built-in JavaScript bundling.

It was made to work with Browserify by default, but if you use the `--bundler`
command line argument, you can specify r.js as your bundler of choice.

We are going to use this to run r.js each time our `main.js` file is requested.

## `baseUrl`

As with every Require.js project, you are going to want to set your `baseUrl`.

This could be any place you want to begin resolving. Typically, it will be
your vendor or application directory. Since we are doing a simple example,
let's just use our current directory and we will put our JS files at the root
of our project.

```javascript
// config.js
({
  baseUrl: './'
})
```

## `name`

To specify the module (and all of its dependencies) we want to optimize,
we use the `name` option. This option indicates an entry point for r.js
to begin from.

We are going to name our entry module `main.js`, an AMD package convention,
and it will be in the root of our project.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main' // drop the .js, as AMD moduleId resolution appends it
})
```

### `main.js`

Create a file in the root of your project named `main.js`.

This file will be the entry point for your applicaton, and will be wrapped
in a `define` function call.

```javascript
// main.js
define([
  './another-module'
], function(another){

  console.log('inside main.js');

  console.log('another-module dependency:', another);

});
```

### `another-module.js`

Create another file in the root of your project, but name this one `another-module.js`.

This file is a dependency of your `main.js` module and is just used to demonstrate that
we are, in fact, bundling all the dependencies together.

```javascript
// another-module.js
define(function(){

  console.log('inside another-module.js');

  return {
    something: 'yup, another module';
  };

});
```

## `insertRequire`

When bundling files that only contain `define` function calls, loaded by `data-main`,
you need to bootstrap your application with a `require` function call.  We can specify
the `insertRequire` option, to insert a `require` function call at the bottom of our bundle.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main',
  insertRequire: ['main'] // will add `require(['main'])` to the end of your bundle
})
```

## `optimize`

During development, you probably want to disable optimization/uglification, so you will
be able to debug your bundle and builds will happen quicker. This is done by setting
the `optimize` option to `'none'`.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main',
  insertRequire: ['main'],
  optimize: 'none'
})
```

## `out`

The typical way r.js is used is to output a file, determined by providing
a filename string as the `out` option.

Beefy doesn't operate on files, and instead expects to receive data on
`process.stdout`.

The `out` option can also take a function that will receive the output
text as its only parameter.

We are going to leverage the `out` function to redirect the r.js output
to `process.stdout`.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main',
  insertRequire: ['main'],
  optimize: 'none'
  out: console.log // console.log outputs to process.stdout and is tightly bound in node
})
```

## `logLevel`

By default, r.js logs info about the build process.  This gets intercepted
by beefy on stdout and is added to the output served.  r.js provides a `logLevel`
option that can be used to disable logging.  Log level 4 is the level that disables
logging.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main',
  insertRequire: ['main'],
  optimize: 'none'
  out: console.log,
  logLevel: 4
})
```

## Run Beefy

```bash
beefy :main.js --bundler ./node_modules/.bin/r.js -- -o config.js
```
