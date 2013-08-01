This past weekend (July 27, 2013) was [International Nodebots Day](http://nodebotsday.com) and I decided
to attend Portland's event.

The event was excellent and I would like to gives props to all the organizers,
[Tracy](https://twitter.com/hackygolucky), [Troy](https://twitter.com/thoward37),
[Adron](https://twitter.com/adron), [Carter](https://twitter.com/carterrabasa),
[Jason](https://twitter.com/_jden), [Adam](https://twitter.com/s5fs) & anyone else
I might have forgotten, who were able to bring people of all skill levels and
interests together to control hardware with JavaScript.

Some projects I had interest in working on involved using different input devices
to control robots with a small JS layer in between.

I got started a day early with [Troy](https://twitter.com/thoward37) and
[Nick](https://twitter.com/nickniemeir) while hacking at the
[New Relic](https://twitter.com/newrelic) offices.

## Leap + Bots

The first thing we did was read the input data from [LeapMotion's](https://www.leapmotion.com/)
WebSocket connection and use it to drive a [ShieldBot](http://www.seeedstudio.com/depot/shield-bot-p-1380.html)
around using [johnny-five](https://github.com/rwldrn/johnny-five),
[shieldbot.js](https://github.com/phated/shieldbot.js), and [leapjs](http://js.leapmotion.com/).

```javascript
var five = require('johnny-five');
var Shieldbot = require('shieldbot');
var leap = require('leapjs');
var board = new five.Board();

board.on('ready', function(){

  var shieldbot = new Shieldbot();
  var prevZ;

  leap.loop(function(frame){
    if(frame.hands.length){
      var z = frame.hands[0].palmPosition[2];
      var velZ = frame.hands[0].palmVelocity[2];

      if(prevZ == null){
        prevZ = z;
      }

      var direction;
      if(z < prevZ && Math.abs(velZ) > 50){
        direction = 'forward';
        shieldbot.forward(127, 30);
      } else if(z > prevZ && Math.abs(velZ) > 50){
        direction = 'backward';
        shieldbot.reverse(127, 30);
      } else {
        direction = 'still';
        shieldbot.stop();
      }

      prevZ = z;
    } else {
      shieldbot.stop();
    }
  });

});
```

We used the Leap's z-axis velocity tracking of a user's palm to drive the bot
forward and backwards.

<iframe class="vine-embed" src="https://vine.co/v/hAexmlaWBUU/embed/simple" width="600" height="600" frameborder="0"></iframe>
<script async src="//platform.vine.co/static/scripts/embed.js" charset="utf-8"></script>

During Nodebots Day, I was able to add left & right stearing using palm x-axis rotation.

```javascript
// outside leap.loop
var prevX;

// inside leap.loop
var velX = frame.hands[0].palmVelocity[0];
var x = frame.hands[0].palmNormal[0];

if(direction === 'still'){
  if(x > prevX && Math.abs(velX) > 50){
    shieldbot.turnRight(127, 30);
  } else if(x < prevX  && Math.abs(velX) > 50){
    shieldbot.turnLeft(127, 30);
  } else {
    shieldbot.stop();
  }
}

prevX = x;
```

Other plans included using the finders to stop the bot running into things or off tables, but
I was unable to implement these because I quickly moved on to my next input device, Google Glass.

## Glass + Bots

For his [JSConf](https://github.com/jsconf/jsconfus2013/issues/13) talk, Luis created a
sideloadable app, [Face](https://github.com/monteslu/Face), for Glass that broadcast
the sensor data over WebSockets.  With this, I was able to get pitch and roll, which
were both easily translated to the logic of the LeapMotion/ShieldBot combo.

```javascript
var five = require('johnny-five');
var Shieldbot = require('shieldbot');

var io = require('socket.io-client');
var socket = io.connect('http://localhost:11097');
socket.on('connect', function(){
  socket.emit('joinChannel', CHAN_ID_FROM_FACE);
});

var board = new five.Board();

board.on('ready', function(){

  var shieldbot = new Shieldbot();
  var prevZ;
  var prevX;

  socket.on('broadcast', function(message){
    if(message.sensors){
      var z = message.sensors.pitch;
      var x = message.sensors.roll;

      if(prevZ == null){
        prevZ = z;
      }

      var direction;
      if(z > prevZ && Math.abs(z - prevZ) > 1){
        direction = 'forward';
        shieldbot.forward(127, 30);
      } else if(z < prevZ && Math.abs(z - prevZ) > 1){
        direction = 'backward';
        shieldbot.reverse(127, 30);
      } else {
        direction = 'still';
        shieldbot.stop();
      }

      if(direction === 'still'){
        if(x > prevX && Math.abs(x - prevX) > 1){
          shieldbot.turnRight(127, 30);
        } else if(x < prevX  && Math.abs(x - prevX) > 1){
          shieldbot.turnLeft(127, 30);
        } else {
          shieldbot.stop();
        }
      }

      prevZ = z;
      prevX = x;
    } else {
      shieldbot.stop();
    }
  });

});
```

After replacing some variables and reworking my velocity check, I was able to drive the bot
forward and backward, matching the direction I tilted my head.  Rolling my head left
and right also turned the bot.

<iframe class="vine-embed" src="https://vine.co/v/hq620P0Qgjx/embed/simple" width="600" height="600" frameborder="0"></iframe>
<script async src="//platform.vine.co/static/scripts/embed.js" charset="utf-8"></script>

This demo wasn't ideal because I couldn't see where the bot was while controlling it.
I needed something that could be about eye height, so I could see its movement while
controlling it.  Luckily, [Carter](https://twitter.com/carterrabasa) had brought a [NodeCopter](http://nodecopter.com/)!!

<iframe class="vine-embed" src="https://vine.co/v/hAgvjhl0zLA/embed/simple" width="600" height="600" frameborder="0"></iframe>
<script async src="//platform.vine.co/static/scripts/embed.js" charset="utf-8"></script>

<p><iframe src="http://player.vimeo.com/video/71408246" width="600" height="338" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe></p>

## Glass + NodeCopter

If it was so easy to switch the LeapMotion logic to use the Glass data, I figured it
would be just as easy to swap the ShieldBot logic for the NodeCopter's.  After
discussing the idea with a few people, it was suggested I use the [Voxel Drone](https://github.com/shama/voxel-drone)
to test my logic before applying it to a physical drone.  After a very frustrating
experience trying to get Voxel Drone, which seems to have been abandoned, to actually
work, I was able to wire up my Glass/NodeCopter logic and added code for the drone to
take off with the A button and land with the B button displayed in Glass.

```javascript
var io = require('socket.io-client');
var socket = io.connect('http://localhost:11097');
socket.on('connect', function(){
  socket.emit('joinChannel', CHAN_ID_FROM_FACE);
});

var prevZ;
var prevX;

socket.on('broadcast', function(message){

  if(message.event){
    if(message.event === 'A'){
      drone.takeoff();
    }
    if(message.event === 'B'){
      drone.land();
    }
    return;
  }
  if(message.sensors){
    var z = message.sensors.pitch;
    var x = message.sensors.roll;

    if(prevZ == null){
      prevZ = z;
    }

    var direction;
    if(z > prevZ && Math.abs(z - prevZ) > 1){
      direction = 'forward';
      drone.front(1);
    } else if(z < prevZ && Math.abs(z - prevZ) > 1){
      direction = 'backward';
      drone.back(1);
    } else {
      direction = 'still';
      drone.stop();
    }

    if(direction === 'still'){
      if(x > prevX && Math.abs(x - prevX) > 1){
        drone.right(1);
      } else if(x < prevX  && Math.abs(x - prevX) > 1){
        drone.left(1);
      } else {
        drone.stop();
      }
    }

    prevZ = z;
    prevX = x;
  } else {
    drone.stop();
  }

});
```

I loaded up Voxel Drone and was able to fly a WebGL drone in my browser.

All I needed to do was swap the voxel-drone specific code for:

```javascript
var arDrone = require('ar-drone');
var drone  = arDrone.createClient({
  ip: '192.168.1.10'
});
```

## Demos

![Flying a Drone with Glass #1](https://pbs.twimg.com/media/BQN9iHbCYAAJOo1.jpg)

Photo by [Alice Goldfuss](https://twitter.com/alicegoldfuss/status/361270420172857344)

![Flying a Drone with Glass #2](https://pbs.twimg.com/media/BQOckcxCAAMZtLA.jpg)

Photo by [Chris Hansen](https://twitter.com/cxhansen/status/361304545126645760)

<p><iframe width="600" height="450" src="//www.youtube.com/embed/BSXWkidgA1Q?rel=0" frameborder="0" allowfullscreen></iframe></p>

<p><iframe src="http://player.vimeo.com/video/71405845" width="600" height="337" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe></p>

Turns out that I was driving the drone at full speed in each direction I tilted my head.
I should have had the speed at about 0.3 instead of 1.  Lesson learned.  I would have also
liked to add the ability to rotate the drone left and right based on the Glass' azimuth value,
but I guess that will have to be in the future.  After such an amazing Nodebots Day, I have all
sorts of ideas swimming around in my head for taking these types of interactions further!

-Blaine
