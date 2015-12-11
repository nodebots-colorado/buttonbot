# Button Bot

## Getting Set Up

There is only one real prerequisite to getting set up and that's to have [Node.js][] installed on your computer. You can [download Node.js from the official website][Node.js].

[Node.js]: http://nodejs.org/

After you install Node.js, download or clone this project, navigate to the directory in your terminal, and install the dependencies using [npm][], which comes with Node.js.

```shell
npm install
```

[npm]: http://npmjs.org

## A Tour of the Project

This project contains two main files:

- `server.js`: a little Node.js application that will connect with the Arduino and start up a web server.
- `client/script.js`: the code that will run in the browser.

## Responding to a Button Press

First, let's get wire up a button and get Johnny-Five to recognize a button press.

![Button Diagram](https://github.com/rwaldron/johnny-five/blob/master/docs/breadboard/button.png)

```js
board.on('ready', function () {

  button = new five.Button(2);

  button.on('down', function() {
    console.log('down');
  });

  button.on('hold', function() {
    console.log('hold');
  });

  // 'up' the button is released
  button.on('up', function() {
    console.log('up');
  });
});
```

## Pushing Data to the Browser with WebSockets

### Establishing a Connection

The next thing we'll need to do is figure out how to send stuff out to the browser from the server. Let's establish a connection between the client and the server.

In `server.js`:

```js
io.on('connection', function (socket) {
  console.log('Someone has connected.');
});
```

In `client/script.js`:

```js
var socket = io();
```

### Sending Messages Out to the Client

We've established a connection. Next, let's push a message from Node to the browser. We'll send a message over the `message` channel.

In `server.js`:

```js
io.on('connection', function (socket) {
  console.log('Someone has connected.');
  io.sockets.emit('message', 'Hello from Node!');
});
```

In `client/script.js`:

```js
socket.on('message', function (message) {
  console.log('Something came along on the "message" channel:', message);
});
```

For fun, we can go ahead and send a message every few seconds. In `server.js`:

```js
io.on('connection', function (socket) {
  console.log('Someone has connected.');

  setInterval(function () {
    var currentTime = (new Date()).toString();
    io.sockets.emit('message', 'Hello from Node! ' + currentTime);
  }, 2000);
});
```

That's a terrible idea for a number of reasons, so let's change it back to the previous example.

### Bringing It All Together

So, we can listen for button presses on in Node, we can listen for Socket messages in the browser, and we can send socket messages from Node to the browser. The final step will be sending a socket message when a button press occurs.

In `server.js`:

```js
board.on('ready', function () {

  button = new five.Button(2);

  button.on('down', function() {
    console.log('down');
    io.sockets.emit('button', 'down');
  });

  button.on('hold', function() {
    console.log('hold');
    io.sockets.emit('button', 'hold');
  });

  // 'up' the button is released
  button.on('up', function() {
    console.log('up');
    io.sockets.emit('button', 'up');
  });
});
```

Finally, we need to listen for this in the browser in `client/script.js`:

```js
socket.on('button', function (position) {
  console.log('Button', position);
  document.getElementById('status').innerText = position;
});
```
