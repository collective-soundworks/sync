# sync

This module synchronises the server and clients clocks on a common shared clock called “sync clock”.

All time calculations and exchanges should be expressed in the sync clock time, and all times are in seconds.

## Get started

On the client side, you have to launch an instance of `SyncClient` and call the method `start`. You must specify what functions to use to:

- get the local time,
- send a message to the server (WebSocket-like),
- receive a message from the server (WebSocket-like).

For instance, with the [`socket.io`](https://github.com/Automattic/socket.io) library:

```javascript
/* Client side */

var audioContext = new AudioContext() || new webkitAudioContext();
var socket = io();
var Sync = require('sync/client');

////// Define the helper functions
// function to get the local time
var getTimeFunction = () => { return audioContext.currentTime; };
// function to send a message to the server
var sendFunction = socket.emit;
 // function to receive a message from the server
var receiveFunction = socket.on;

// Initialize the sync module and start the synchronisation process
var sync = new SyncClient(getTimeFunction);
// CAVEAT: first start the audioContext (via click or touch event)
sync.start(sendFunction, receiveFunction);

// Listen for the events that indicate that the clock is in sync
sync.on('sync:status', (report) => {
  if(report.status === 'training' || 'sync') {
    //  whatever you need to do once the clock is in sync
    }
})
```

On the server side, you have to launch an instance of `SyncServer` and call the method `start`. Just like on the client side, you must specify what functions to use to:

- get the local time;
- send a message to the clients;
- receive a message from the clients.

For instance, with the [`socket.io`](https://github.com/Automattic/socket.io) library:

```javascript
/* Server side */

// Require libraries
var io = require('socket.io');
var Sync = require('sync/server');

 // function to get the local time
var getTimeFunction = () => {
  let time = process.hrtime();
  return time[0] + time[1] * 1e-9;
};
// Initialize sync module
var sync = new Sync(getTimeFunction);

// Set up a WebSocket communication channel with the client
// and start to listen for the messages from that client
io.on('connection', function (socket) {
  // function to send a WebSocket message to the client
  let sendFunction = (msg, ...args) => socket.emit(msg, ...args);
  // function to receive a WebSocket message from the client
  let receiveFunction = (msg, callback) => socket.on(msg, callback);

  sync.start(sendFunction, receiveFunction);
  
  ... // the rest of your code
});
```

## Caveats

The synchronisation process is continuous: after a call to the `start`
method, it runs in the background. It is important to avoid blocking
it, on the client side and on the server side.

The reading of the clock often takes place in the main [event loop], when
it is not accessible from a [Web Worker] or a [Node.js]
[child process]. Then, the synchronisation process also runs in the main
[event loop]. For [Node.js], you can use [node-blocked] to monitor the
blockage.

## Documentation

The API documentation is available at `./doc/sync/${sync_version}/index.html`
Use `npm run doc` to generate it.

You can an read an [article], presented during the [Web Audio Conference
2016],  that describes the synchronisation process in
details, with measurements.

## License

[BSD-3-Clause]. See the [LICENSE file].


[article]:  http://hdl.handle.net/1853/54598
[BSD-3-Clause]: https://opensource.org/licenses/BSD-3-Clause
[child process]: https://nodejs.org/api/child_process.html
[event loop]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop
[LICENSE file]: ./LICENSE.
[node-blocked]: https://www.npmjs.com/package/blocked
[Node.js]: https://nodejs.org
[Web Audio Conference 2016]: http://webaudio.gatech.edu/
[Web Worker]: https://developer.mozilla.org/en-US/docs/Web/API/Worker
