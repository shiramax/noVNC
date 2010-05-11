VNC HTML5 Client
================


Description
-----------

A VNC client implemented using HTML5, specifically Canvas and
WebSocket (supports 'wss://' encryption).

For browsers that do not have builtin WebSocket support, the project
includes web-socket-js, a WebSocket emulator using Adobe Flash
(http://github.com/gimite/web-socket-js).

In addition, as3crypto has been added to web-socket-js to implement
WebSocket SSL/TLS encryption, i.e. the "wss://" URI scheme.
(http://github.com/lyokato/as3crypto_patched).


Requirements
------------

Until there is VNC server support for WebSocket connections, you need
to use a WebSocket to TCP socket proxy. There is a python proxy
included ('wsproxy'). One advantage of using the proxy is that it has
builtin support for SSL/TLS encryption (i.e. "wss://").

There a few reasons why a proxy is required:

  1. WebSocket is not a pure socket protocol. There is an initial HTTP
     like handshake to allow easy hand-off by web servers and allow
     some origin policy exchange. Also, each WebSocket frame begins
     with 0 ('\x00') and ends with 255 ('\xff').

  2. Javascript itself does not have the ability to handle pure byte
     strings (Unicode encoding messes with it) even though you can
     read them with WebSocket. The python proxy encodes the data so
     that the Javascript client can base64 decode the data into an
     array. The client requests this encoding

  3. When using the web-socket-js as a fallback, WebSocket 'onmessage'
     events may arrive out of order. In order to compensate for this
     the client asks the proxy (using the initial query string) to add
     sequence numbers to each packet.

To encrypt the traffic using the WebSocket 'wss://' URI scheme you
need to generate a certificate for the proxy to load. You can generate
a self-signed certificate using openssl. The common name should be the
hostname of the server where the proxy will be running:

    `openssl req -new -x509 -days 365 -nodes -out self.pem -keyout self.pem`


Usage
-----

* run a VNC server.
 
    `vncserver :1`

* run the python proxy:

    `./wsproxy.py [listen_port] [vnc_host] [vnc_port]`

    `./wsproxy.py 8787 localhost 5901`


* run the mini python web server to serve the directory:

    `./web.py PORT`

    `./web.py 8080`

* Point your web browser at http://localhost:8080/vnc.html
 (or whatever port you used above to run the web server).

* Specify the host and port where the proxy is running and the
  password that the vnc server is using (if any). Hit the Connect
  button and enjoy!


Integration
-----------

The client is designed to be easily integrated with existing web
structure and style.

At a minimum you must include the script and call the RFB.load()
function which takes a parameter that is the ID of the DOM element to
fill. For example:

    <body>
        <div id='vnc'>Loading</div>
    </body>
    <script src='vnc.js'></script>
    <script> windows.onload = RFB.load('vnc'); </script>


The file include/plain.css has a list of stylable elements.