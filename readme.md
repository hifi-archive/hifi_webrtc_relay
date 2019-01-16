﻿# HiFi WebRTC Relay - A WebRTC bridge between HiFi servers and a HiFi web client.

The HiFi WebRTC Relay bridges the connection between HiFi servers (including the domain server and assignment clients) and the HiFi web client. Packets received from the HiFi servers are sent to the web client and vice versa. The relay should be easy to build using Qt SDK tools. An included build script allows easily building for Linux.

The relay acts as a WebSocket signalling server used for making connections to the web clients. When a web client opens a WebSocket connection to the relay’s signalling server, it creates a new HifiConnection object, which connects to the web client via a WebRTC peer connection. A single data channel is established over this peer connection for sending/receiving packets to/from the web client. A simpler, lightweight implementation of the WebRTC DataChannels API is being used for this project: https://github.com/chadnickbok/librtcdcpp.

The HifiConnection follows a network protocol similar to that of the Hifi native client. The relay performs a domain ID lookup via the metaverse API (e.g. https://metaverse.highfidelity.com/api/v1/places/janusvr) given the domain name specified by the web client user. The relay then sends requests to the stun server (default: stun.highfidelity.io) to discover its public address/port combo and sends requests to the ice server (default: ice.highfidelity.com) to obtain the address/port info for the domain server that the user wants to connect to.

Afterwards, the relay sends DomainConnectRequest packets to the domain server in an attempt to connect to the domain server. Users can potentially log in via username signing via the metaverse API, though this needs implementing on the web client side. Once a DomainList packet is obtained from the domain server, the HifiConnection parses out the info for the different assignment clients and stores it in Node objects. We are then able to receive/send packets to each of the assignment clients by using a UDP socket. We decipher which Node to send packets to/from by encapsulating packets with the node type we are communicating with (done on both the relay and web client side). Upon receiving a packet from a Node via the UDP socket, it is forwarded via the data channel to the web client. Upon receiving a packet from the web client via the data channel, it is forwarded to the correct node via the UDP socket. If we receive no packets from either the Hifi servers or from the web client, the connection times out.

TODO: We have the relay compiling on OSX and Linux, but have yet to compile librtcdcpp (a library dependency for WebRTC DataChannels) for Windows.