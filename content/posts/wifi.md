---
title: "What is Wi-Fi anyway?" 
date : "2023-11-09"
---

You might ask what you mean by 'What is Wi-Fi?':confused: - I connect my device to the WiFi network and boom I am accessing the internet. Well, I held this abstraction for quite some time and the time had come to delve into a lower abstraction, because why not, there is no harm in knowing some background details. For the sake of not losing my mind, I wouldn't dive into super lower-level details nor I have looked into them :smiley:. But this should at least give you an understanding of how 'WiFi' operates.

### Background
_Let's clear (or loosely define) some background terms_ :
1. **Network**: Group of computers communicating and sharing information/resources with each other.
2. **Wireless network**: Network without physical connections(wires, cables etc.).
3. **Physical Layer**(TCP/IP model Layer 1): The actual medium between the nodes of a network. Maybe an easier thing to imagine would be how bits are sent as signals between the computer nodes are handled by this layer.
4. **Data link layer**(TCP/IP model Layer 2): This layer handles framing, addressing, reliable connectivity, error detection/correction, collision detection etc. It makes sense that when data is being transmitted through the physical layer, some upper layer needs to handle these problems for effective communication.
5. **Frame**: Unit of transport at the data link layer. Would consist of some headers and the actual data (payload) being sent. Headers would help in establishing the connection(source/destination MAC addresses) and some other control data.

### General scenario
Consider what happens when you try to connect to a WiFi network for the first time through your phone. 
1. You see the name of the wifi network listed in your phone's wifi settings. The technical term for this is actually called SSID (service set identifier).
2. Some networks would require you to put in a password(most home wifi networks) and others would not(some mall/coffee shop networks).
3. You click connect and you are able to access the internet (in most cases!).

> So a simple question arises what exactly is called Wi-Fi here? 

- WiFi is a standard used to enable wireless networking. And it operates at the _physical layer_ and the _data link layer_. 
- At the physical layer, it enables communication between the nodes using radio waves. At the data link layer, it mostly does the things mentioned in the background section.

> How does my phone even know/discover that a wireless network exists and information about it?

- Routers periodically broadcast information about their network. It contains information such as SSID (the name!), security, signal strength etc. Technically called WiFi beacon signal. 
- And since this broadcast is via radio waves anyone within the vicinity can receive them.
- Your phone periodically checks for these signals and lists them. WiFi standards would have defined the structure of the signal in the first place. And since your phone supports the WiFi standard it can interpret it.

> OK I was in a coffee shop and connected to a network without a password then what happened?

- The client (mobile) node sends an association request to the router, the router acknowledges the request and welcomes the client onto the network. It would receive an IP address through DHCP servers.
- Once the IP address is assigned the client is part of the network. The communication required by the client will be handled and routed by the router (e.g. internet access).
- The concern here is that this was an open network so there is no encryption of any sort between the router and the client. Anyone can intercept the WiFi radio signal and read the unencrypted frame payload. You still might be secured if communicating over TLS (Transport layer security). 
- But communication happening lower than TLS or using different transport like UDP might still be affected (e.g. DNS queries over UDP).

> Well, what happens when I connect to a protected WiFi network requiring some password?

- In this case, the network would have some type of **Security type** standard configured. Some examples of it are WPA/WPA2/WPA3. Most probably, you would have seen these on your phone when connecting to a password-protected WiFi network. This information is also broadcast along with wifi beacon signals.
- This password detail is sent along with the association request to the router. With that, a 4-way handshake occurs between the client and the router. The end result would be that the client and router would have the same secret key with them, which will be used to encrypt/decrypt (both with the same key!) the frame payload. Thus, securing our connection.
- These WPA standards typically differ in how the handshakes occur and how the symmetric key is generated.

> Got it, let's say I am connected to a protected WiFi network how does the overall flow look like now when the client wants to communicate with some node out there on the internet?

1. The client generates the layer 2 frames and the frame payload itself has layer 3 details (IP address details of the node on the internet). The frame payload is encrypted using the symmetric key generated via 4-way handshake. 
2. The client after looking at the local routing table infers that to send the data onto the internet the network router is the default gateway. So the router's MAC address is filled in the layer 2 destination frame header. And the frame is sent over the physical layer (i.e. radio waves for wifi). 
3. The router intercepts the frames decrypts the payload and inspects that the IP address belongs to some node over the internet. 
4. It then repacks this IP packet into another frame with the destination being the ISP (of course there might be intermediaries).
5. And similar thing happens in reverse for the response received (for our client).
 
I know, I have omitted many details, but the point was to get to a level where we have a general idea of how the stuff works. Hope, this is useful to people looking for more details on the working of the WiFi.
