# Examples

## turn-server
The `turn-server` directory contains 4 examples that show common Pion TURN usages. All of these examples take the following arguments.

* -users     : &lt;username&gt;=&lt;password&gt;[,&lt;username&gt;=&lt;password&gt;,...] pairs
* -realm     : Realm name (defaults to "pion.ly")
* -port      : Listening port (defaults to 3478)
* -public-ip : IP that your TURN server is reachable on, for local development then can just be your local IP, avoid using `127.0.0.1` as some browsers discard from that IP.

The four example servers are

#### add-software-attribute
This examples adds the SOFTWARE attribute with the value "CustomTURNServer" to every outbound STUN packet. This could be useful if you want to add debug info to your outbound packets.

You could also use this same pattern to filter/modify packets if needed.

#### log
This example logs all inbound/outbound STUN packets. This could be useful if you want to store all inbound/outbound traffic or generate rich logs.

You could also intercept these reads/writes if you want to filter traffic going to/from specific peers.

```sh
$ cd log
$ go mod download
$ go build
$ ./log -public-ip 127.0.0.1 -users username=password,foo=bar
```

#### simple
This example is the most minimal invocation of a Pion TURN instance possible. It has no custom behavior, and could be a good starting place for running your own TURN server.

```sh
$ cd simple
$ go mod download
$ go build
$ ./simple -public-ip 127.0.0.1 -users username=password,foo=bar
```

#### tcp
This example demonstrates listening on TCP. You could combine this example with `simple` and you will have a Pion TURN instance that is available via TCP and UDP.

```sh
$ cd tcp
$ go mod download
$ go build
$ ./tcp -public-ip 127.0.0.1 -users username=password,foo=bar
```

## turn-client
The `turn-client` directory contains 2 examples that show common Pion TURN usages. All of these examples take the following arguments.

* -host      : TURN server host
* -ping      : Run ping test
* -port      : Listening port (defaults to 3478)
* -realm     : Realm name (defaults to "pion.ly")
* -user      : &lt;username&gt;=&lt;password&gt; pair

#### tcp
Dials the requested TURN server via TCP

#### udp
Dials the requested TURN server via UDP

```sh
$ go udp
$ go build
$ ./udp -host <turn-server-name> -user=user=pass
```

By adding `-ping`, it will perform a ping test. (it internally creates a 'pinger' and send
a UDP packet every second, 10 times then exits.

```sh
$ go build
./turn-client -host <turn-server-name> -user=user=pass -ping
```

Following diagram shows what turn-client does:
```
          +----------------+
          |   TURN Server  |
          |                |
          +---o--------o---+
  TURN port  /^      / ^\
       3478_/ |     /  | \_relayConn (*1)
              |    /   |
              |  _/    |
  mappedAddr_ | /      | ___external IP:port
       (*2)  \|v       |/    for pingerConn
          +---o--------o---+     (*3)
          |   |   NAT  |   |
          +----------------+
              |        |
  TURN    ___ |        | __pingerConn
  listen     \|        |/     (sends `ping` to relayConn)
  port    +---o--------o---+
  (conn)  |   turn-client  |
          +----------------+
```

> (*1) The relayConn actually lives in the local turn-client, but it acts as if it is
> listening on the TURN server. In fact, relayConn.LocalAddr() returns a transport address
> on which the TURN server is listening.

> (*2) For relayConn to send/receive packet to/from (*3), you will need to give relayConn permission
> to send/receive packet to/from the IP address. In the example code, this is done by sending a
> packet, "Hello" (content does not matter), to the mappedAddr. (assuming the IP address of
> mappedAddr and the external IP:port (*3) are the same) This process is known as
> "UDP hole punching" and TURN server exhibits "Address-restricted" behavior. Once it is done,
> packets coming from (*3) will be received by relayConn.
