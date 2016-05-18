golang socket.io
================

golang implementation of [socket.io](http://socket.io) library, client and server

You can check working chat server, based on this library, at (funstream)[http://funstream.tv]

Examples directory contains simple client and server.

### Installation

    go get github.com/graarh/golang-socketio

### Simple server usage

```go
	//create
	server := gosocketio.NewServer(transport.GetDefaultWebsocketTransport())

	//handle connected
	server.On(gosocketio.OnConnection, func(c *gosocketio.Channel, args interface{}) {
		log.Println("New client connected")
		//join them to room
		c.Join("chat")
	})

	type Message struct {
		Name string `json:"name"`
		Message string `json:"message"`
	}

	//handle custom event
	server.On("send", func(c *gosocketio.Channel, msg Message) string {
		//send event to all in room
		c.BroadcastTo("chat", "message", msg)
		return "OK"
	})

	//setup http server
	serveMux := http.NewServeMux()
	serveMux.Handle("/socket.io/", server)
	log.Panic(http.ListenAndServe(":80", serveMux))
```

### Javascript client for this server

```javascript
var socket = io('ws://yourdomain.com', {transports: ['websocket']});

    // listen for messages
    socket.on('message', function(message) {

        console.log('new message');
        console.log(message);
    });

    socket.on('connect', function () {

        console.log('socket connected');

        //send something
        socket.emit('send', {name: "my name", message: "hello"}, function(result) {

            console.log('sended successfully);
            console.log(result)
        });
    });
```

### Server, detailed usage

```go
    //create server instance, you can setup transport parameters or get the default one
    //look at websocket.go for parameters description
	server := gosocketio.NewServer(transport.GetDefaultWebsocketTransport())

	// --- this is default handlers

	//on connection handler, occurs once for each connected client
	server.On(gosocketio.OnConnection, func(c *gosocketio.Channel, args interface{}) {
	    //client id is unique
		log.Println("New client connected, client id is ", c.Id())

		//you can join clients to rooms
		c.Join("room name")

		//of course, you can list the clients in the room, or account them
		channels, _ := c.List(data.Channel)
		log.Println(len(channels), "clients in room")
	})
	//on disconnection handler, if client hangs connection unexpectedly, it will still occurs
	server.On(gosocketio.OnDisconnection, func(c *gosocketio.Channel, args interface{}) {
		//this is not necessary, client will be removed from rooms
		//automatically on disconnect
		//but you can remove client from room whenever you need to
		c.Leave("room name")

		log.Println("Disconnected")
	})
	//error catching handler
	server.On(gosocketio.OnError, func(c *gosocketio.Channel, args interface{}) {
		log.Println("Error occurs")
	})

	// --- this is custom handler

	//custom event handler
	server.On("handle something", func(c *gosocketio.Channel, channel Channel) string {
		log.Println("Something successfully handled")

		//you can return result of handler, in this case
		//handler will be converted from "emit" to "ack"
		return "result"
	})

    //you can get client connection by it's id
    channel, _ := server.GetChannel("client id here")
    //and send the event to the client
    type MyEventData struct {
        Data: string
    }
    channel.Emit("my event", MyEventData{"my data"})

    //or you can send ack to client and get result back
    result, err := channel.Ack("my custom ack", MyEventData{"ack data"}, time.Second * 5)

    //you can broadcast to all clients
    server.BroadcastToAll("my event", MyEventData{"broadcast"})

    //or for clients joined to room
    server.BroadcastTo("my room", "my event", MyEventData{"room broadcast"})

    //setup http server like this for handling connections
	serveMux := http.NewServeMux()
	serveMux.Handle("/socket.io/", server)
	log.Panic(http.ListenAndServe(":80", serveMux))
```

### Client

```go
    //connect to server, you can use your own transport settings
	c, err := gosocketio.Dial("localhost:80", transport.GetDefaultWebsocketTransport())

	//do something, handlers and functions are same as server ones

	//close connection
	c.Close()
```

### Roadmap

1. Tests
2. Travis CI
3. http longpoll transport
4. pure http (short-timed queries) transport
5. binary format