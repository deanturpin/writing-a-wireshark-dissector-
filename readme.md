Parked until I need it again :)

---

# TODO
- Choosing a port number
	https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers

# Get started

This guide takes advantage of the features available only to Linux/bash.

Watch this video to get a feel for where dissectors fit into the grand scheme of
things.

https://www.youtube.com/watch?v=sP8ljsM8c9s

# Lua
An example of Lua dissector being developed.

https://www.youtube.com/watch?v=I4nf23HywmI

# Creating a message format
I have two messages in my format:
- request time
- respond with time

```c
struct request {

	unsigned int stx;			// Start transmission 0xa5a5
	unsigned int mode;		// 0 request, 1 response
	unsigned int length;	// Length is zero for a request
	char[length] time;		// Length of response string
}
```

Let's start by sending a packet to ourselves (using UDP so we don't have to
think about establishing a connection right now).
```bash
xxd -r -p <<< 'a5a5 0001 0000' > /dev/udp/0.0.0.0/9999
```

And capture it in ```tcpdump```.
```bash
sudo tcpdump -i lo -X -n dst port 9999

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
08:59:34.588571 IP 127.0.0.1.41157 > 127.0.0.1.9999: UDP, length 6
	0x0000:  4500 0022 f78f 4000 4011 4539 7f00 0001  E.."..@.@.E9....
	0x0010:  7f00 0001 a0c5 270f 000e fe21 a5a5 0001  ......'....!....
	0x0020:  0000      
```

# How long did this take?
I've logged my time on this project using the "logwork" hashtag in the git log.
Includes documenting it.

```bash
git log | grep logwork
YouTube and tinkering with /dev/tcp to develop a test message format #logwork 1h
#logwork 2h Googling, watching YouTube, tinkering with tcpdump options
```

# Resouces

"Dissectors are usually written in C, it's also possible to write them in Lua
for fast prototyping."

## C
- http://www.sewio.net/open-sniffer/develop/how-to-write-wireshark-dissector/
- https://www.wireshark.org/docs/wsdg_html_chunked/ChDissectAdd.html#ChDissectSetup
- https://github.com/boundary/wireshark/blob/master/doc/packet-PROTOABBREV.c
- http://www.sewio.net/open-sniffer/develop/how-to-write-wireshark-dissector/

## Lua
- https://prontog.wordpress.com/2016/01/29/a-simpler-way-to-create-wireshark-dissectors-in-lua/
- https://wiki.wireshark.org/Lua/Dissectors
- https://ask.wireshark.org/questions/5726/newbie-trying-to-get-lua-scripts-to-execute

## tcpdump
- http://rationallyparanoid.com/articles/tcpdump.html - tcpdump usage

# Injecting messages
With xxd - inject just enough that it fits nicely on a line once all the other
layers are added.
```bash
while :; do xxd -r -p <<< '0108 5556 5758' > /dev/tcp/127.0.0.1/9999; sleep 1;
done
```

Create a server in another terminal
```
nc -kl 9999 | xxd -c 6
```

Capture the messages with tcpdump, but only those with out destination port.
```bash
sudo tcpdump -i lo -XX -n dst port 9999

21:59:26.540459 IP 127.0.0.1.38583 > 127.0.0.1.9999: UDP, length 6
0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
0x0010:  0022 8d70 4000 4011 af58 7f00 0001 7f00 .".p@.@..X......
0x0020:  0001 96b7 270f 000e fe21 0108 5556 5758 ....'....!..UVWX
```

