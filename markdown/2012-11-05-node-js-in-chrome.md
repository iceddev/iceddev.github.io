This:

```
<script type="text/javascript" src="node.js"></script>
```

Allows you to do this in a Chrome App:

```javascript
var http = require('http');
http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
}).listen(1337, '127.0.0.1');
console.log('Server running at http://127.0.0.1:1337/');
```

## This is actually Node.js code running the http listener on Chrome's Javascript VM!

While we like the [Browserver project](http://browserver.org/), it uses a reverse proxy and pushes requests down to the client via a websocket.  Node-chromify took a different approach, and has no external server dependency.

You can get the source to the demo app, and a build of node.js (the script, not the whole node project) here:  [https://github.com/iceddev/node-chromify](https://github.com/iceddev/node-chromify)

## So, how did we do this?

[node-chromify](https://github.com/iceddev/node-chromify) is built on top of [browserify](https://github.com/substack/node-browserify), which allows node modules to be converted into a format that can be run in a browser.
Browsers can't normally created TCP listeners, but Chrome Apps can in Canary with the experimental extension APIs flag enabled.

Google's Paul Kinlan created a net-browserify module which wraps the Chrome API calls to the node.js "net" package.  This gets us quite a bit further. We've [patched](https://github.com/iceddev/net-chromify) a few things on this to help us out.

We've created an [http-chromify](https://github.com/iceddev/http-chromify) module which is based on Node.js' "http" module.  The Node.js version uses native C++ code to parse the HTTP messages between client and server.  We replaced that with the pure JS [http-parser-js](https://github.com/creationix/http-parser-js) module.

There were also couple of other pure JS modules that weren't browserified yet, namely [freelist](https://github.com/iceddev/freelist-chromify) and [string_decoder](https://github.com/iceddev/string_decoder-chromify).  We created repos for those so that browserify could pull them in.

Finally we used the excellent [Grunt.js](http://gruntjs.com/) to build and package this all up into [node-chromify](https://github.com/iceddev/node-chromify).

## How do you run it?

You'll need Chrome Canary to run the demo.  Canary has recently allowed developers the ability to open TCP socket listeners in apps and extensions.

Also, this requires enabling "Experimental Extension APIs" from chrome://flags

Then, in chrome://extensions  use the "Load unpacked extension..." to point to the examples/plugin folder in the [node-chromify](https://github.com/iceddev/node-chromify) source.

We're accepting pull request to [node-chromify](https://github.com/iceddev/node-chromify), please help us fix this :)

## Why did we do this?

"any application that can be written in JavaScript, will eventually be written in JavaScript."

It's the law.  Obey the law.