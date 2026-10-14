# Wireshark investigation

A short packet capture on the home Wi-Fi, done to see the difference between three ways of looking at a network: DNS logs (Pi-hole), the router's device list, and the raw packets themselves. About 45 seconds of normal browsing, roughly 1,300 packets.

## Setup

- Wireshark 4.6 on the Windows laptop, with the Npcap capture driver it bundles. USBPcap skipped, not needed.
- Interface: **Wi-Fi**. The list also shows several "Local Area Connection*" entries; those are inactive virtual adapters. The right one is the interface with a moving traffic sparkline next to it.
- Capture started, normal browsing for about 45 seconds, capture stopped.

## The three filters that made sense of it

A raw capture is a wall of lines. Display filters cut it down to one kind of traffic at a time.

| Filter | Shows | What it gave |
|---|---|---|
| `tls.handshake.type == 1` | TLS Client Hello packets, the first message of every HTTPS connection | The site name each connection was going to (see below) |
| `ssdp` | UPnP discovery broadcasts on the local network | Which device was looking for a router to open ports on |
| `arp` | "Who has this IP address?" lookups between devices | A full map of the devices on the network, built from traffic alone |

## What showed up

### HTTPS destinations, even though HTTPS is encrypted

The first packet of a TLS connection carries the name of the site in plain text, in a field called **SNI** (Server Name Indication). It has to: the server needs to know which certificate to answer with before encryption starts.

In Wireshark: pick a Client Hello, expand **Handshake Protocol → Extensions → server_name**. Two real destinations from the capture were the Steam content network and the Epic Games launcher, both running in the background. Everything after the handshake is unreadable, but the destination was not.

### QUIC

Handshake packets to Cloudflare and Google over **QUIC**. That is HTTP/3, which runs over UDP instead of TCP and sets up encryption in its own way. A filter written for classic TCP + TLS misses it entirely, so a modern capture needs both.

### SSDP: the NAS looking for a router

Repeated broadcasts to the UPnP multicast address. The guess was a smart TV. The source address's vendor prefix said otherwise: it was the **NAS**. The packet detail showed it searching for `urn:schemas-upnp-org:service:WANIPConnection:2`, which means "is there a router here that will let me open ports to the internet?". The router was separately sending its own `NOTIFY` announcements, the other half of the same conversation.

Worth knowing: UPnP is how devices open ports on a router without anyone asking. It is convenient and also a common thing to switch off on a router once you know what is using it.

### ARP: a device map with no admin page

Every device that wants to talk to another on the same network first asks "who has this IP?". Each ARP packet has the sender's MAC address, and the first half of a MAC address identifies the manufacturer. Wireshark prints that vendor name for you.

From ARP alone, every main device was identifiable:

| Device | Vendor shown by Wireshark |
|---|---|
| The laptop doing the capture | Intel |
| The Pi-hole Raspberry Pi | Raspberry Pi |
| The main router | TP-Link |
| Two mesh satellite radios, asking for the router on behalf of their own clients | TP-Link |
| The NAS | UGREEN |

All of it normal housekeeping: the laptop and the Pi re-confirming each other's addresses, the router checking the laptop, the mesh radios finding the gateway.

### One red line that wasn't a problem

Wireshark's colouring flagged a **TCP Spurious Retransmission** and a **Dup ACK**. Red and black lines look alarming. Here it was a packet that arrived out of order and got sent again, ordinary noise on Wi-Fi, not a fault. Lesson: the colours point you at things to look at, not at things that are broken.

## What each tool can and can't see

| Tool | Sees | Doesn't see |
|---|---|---|
| Pi-hole | Every name a device looked up, and what got blocked | Anything that skips DNS, any local broadcast, anything inside a connection |
| Router device list | Which devices are connected, their addresses | What they are doing |
| Wireshark | Individual connections, site names from SNI, local broadcasts, devices identified from their own traffic | The content of encrypted traffic |

Wireshark was the only one of the three that showed the site name of each HTTPS connection, the local broadcast chatter, and the devices identified purely from what they sent.

## Note for anyone repeating this

Capture only on a network you own or have permission to capture on. On Wi-Fi a normal laptop mostly sees its own traffic plus broadcasts, not other devices' private conversations, which is exactly what this exercise shows.
