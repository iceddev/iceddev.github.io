I hear a lot of talking points by people that use [browserify](http://browserify.org/) about
not wanting to bundle their JavaScript modules only in production, which is the common strategy
that [Require.js](http://requirejs.org/) promotes. It is possible, using beefy and r.js, to
emulate the bundle-during-development workflow. This post will take you through the steps needed
to set up that type of environment.

## What is beefy?

[beefy](https://github.com/chrisdickinson/beefy) is a static web server with built-in JavaScript bundling.

It was made to work with browserify by default, but if you use the `--bundler`
command line argument, you can specify r.js as your bundler of choice.

We are going to use this to run r.js each time our `main.js` file is requested.

Get beefy by using the command:

```bash
npm install -g beefy
```

## What is r.js?

[r.js](https://github.com/jrburke/r.js) is a command line utility designed for running [AMD](https://github.com/amdjs/amdjs-api/wiki)-based projects in node.js,
Rhino, and xpcshell. It also includes an optimizer for combining your AMD files
into a bundle.

By using the optimizer functionality of r.js, in combination with beefy, we can
bundle our JavaScript on each page refresh.

We need to install r.js into our project using the command:

```bash
npm install requirejs
```

If you have a `package.json` and want to stamp it with the dependency, add `--save-dev`
to the command:

```bash
npm install --save-dev requirejs
```

## Minimum Viable Config

There are six options we need to set in our r.js config file that will allow us to use
it with beefy.  They are `baseUrl`, `name`, `insertRequire`, `optimize`, `out` and `logLevel`
and each one is explained below.

These options will be set in a file named `config.js`, which will be passed to the beefy
command.

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

Note: This isn't needed if your `data-main` filename is the same as the module entrypoint name.
We will add it here because it doesn't hurt anything and will help in situations like naming
the files differently or using something like [almond](https://github.com/jrburke/almond).

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
option that can be used to disable logging.  Log level 3 is the level that logs only errors.

```javascript
// config.js
({
  baseUrl: './',
  name: 'main',
  insertRequire: ['main'],
  optimize: 'none'
  out: console.log,
  logLevel: 3
})
```

## `index.html`

The last thing we need is an `index.html` file that includes Require.js. If an `index.html`
file doesn't exist, beefy serves up a default page that just injects a script tag for your
entrypoint file. This won't work with the workflow outlined above because we assume the
require machinery will be available (see the Almond section for a different approach).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Beefy + Require.js</title>
</head>
<body>
  <script src="/node_modules/requirejs/require.js" data-main="main" type="text/javascript"></script>
</body>
</html>
```

## Run Beefy

The beefy command takes a filename as the first argument, or a input filename and output filename combination
in the form of `input-filename.js:output-filename.js`. If you don't specify an input filename, like `:output-filename.js`
beefy won't pass a filename to the bundler, but it will still make the result of the bundler command available as
`output-filename.js`.

r.js assumes it is supposed to run a file if it passed as the first argument to the command, and skips the optimization
tool. We will specify the first beefy argument as `:main.js`.

Next, we want to reference the r.js compiler as the bundler: `--bundler ./node_modules/.bin/r.js`

Finally, r.js expects the `-o config.js` argument to start the optimize tool with the `config.js` file. Anything
after the `--` argument to beefy is passed directly to the bundler command.

Putting it all together, the command looks like:

```bash
beefy :main.js --bundler ./node_modules/.bin/r.js -- -o config.js
```
