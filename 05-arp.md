# ARP Investigation

## Objective

Capture and analyze Address Resolution Protocol (ARP) traffic using Wireshark to understand how IPv4 addresses are resolved to MAC addresses on a local Ethernet network.

---

## What is ARP?

Address Resolution Protocol (ARP) is used on IPv4 networks to determine the MAC address associated with a known IPv4 address.

IPv4 addresses operate at Layer 3, while MAC addresses operate at Layer 2. When a device needs to communicate with another device on the local network, it may need to determine the destination device's MAC address before sending an Ethernet frame.

---

## Frame 119 — ARP Request

The first packet was an ARP request.

The request was sent from the router to the PC.

### Ethernet Information

- Source: Router
- Destination: PC
- Type: ARP (`0x0806`)

### ARP Information

- Opcode: Request (`1`)
- Sender IP: `192.168.1.1` *(sanitized)*
- Sender MAC: `11:22:33:44:55:66` *(sanitized)*
- Target IP: `192.168.1.100` *(sanitized)*
- Target MAC: `00:00:00:00:00:00`

The important part of this request is that the router knows the target's IPv4 address but does not yet know its MAC address.

The request is essentially asking:

> "Who has 192.168.1.100? Tell me your MAC address."

Because the target MAC address is unknown, it is represented as `00:00:00:00:00:00`.

### ARP Request Screenshot

![ARP Request](../screenshots/arp/arp-request.png)

---

## Frame 120 — ARP Reply

The second packet was the ARP reply.

The PC responded directly to the router.

### Ethernet Information

- Source: PC
- Destination: Router
- Type: ARP (`0x0806`)

### ARP Information

- Opcode: Reply (`2`)
- Sender IP: `192.168.1.100` *(sanitized)*
- Sender MAC: `AA:BB:CC:DD:EE:FF` *(sanitized)*
- Target IP: `192.168.1.1` *(sanitized)*
- Target MAC: `11:22:33:44:55:66` *(sanitized)*

The PC is identifying itself as the device using `192.168.1.100` and providing its MAC address.

The router can then associate the two addresses:

```text
192.168.1.100 → AA:BB:CC:DD:EE:FF

### ARP Reply Screenshot

![ARP Reply](../screenshots/arp/arp-reply.png)