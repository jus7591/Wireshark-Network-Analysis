# NAT Analysis

## Objective

Analyze how Network Address Translation (NAT) allows a device on a private network to communicate with resources on the public internet.

The goal of this investigation is to identify the private IP address used by the local computer, examine the outbound connection before NAT occurs, and understand where the private-to-public address translation takes place.

---

## Lab Environment

* **Operating System:** Windows 11
* **Packet Capture Tool:** Wireshark
* **Network Type:** Home network
* **Protocol Analyzed:** TCP
* **Destination Service:** HTTPS (TCP/443)

> **Sanitization Note:** Private IP addresses, public IP addresses, MAC addresses, and other identifying network information have been generalized or omitted where appropriate.

---

## Tools Used

* Wireshark
* Windows PowerShell
* `curl.exe`

---

## Private Network Configuration

The Windows computer is assigned a private IPv4 address by the local router.

For this investigation, the private address is represented as:

```text
192.168.1.x
```

Private IPv4 addresses such as those in the `192.168.0.0/16` range are not directly routable across the public internet.

Instead, the local router performs NAT when the computer communicates with an external destination.

---

## Packet Analysis

A Wireshark capture was taken while the computer established an HTTPS connection to an external server.

The captured TCP connection showed traffic similar to:

```text
Source:      192.168.1.x:65393
Destination: <sanitized-public-IP>:443
Protocol:    TCP
```

The source port was dynamically assigned by Windows for the outbound connection.

The destination port was:

```text
443
```

TCP port 443 is commonly used for HTTPS traffic.

The first packet in the TCP conversation was a TCP SYN packet, which begins the TCP three-way handshake.

The basic sequence is:

```text
Client → Server: SYN
Server → Client: SYN/ACK
Client → Server: ACK
```

At this point in the packet capture, the computer is still using its private IP address.

---

## Where NAT Occurs

NAT does **not** occur on the Windows computer itself in this scenario.

The translation occurs on the network's router, between the private LAN and the public WAN connection.

The traffic flow can be represented as:

```text
Windows PC
192.168.1.x
     |
     | Private Network
     v
Home Router
     |
     | NAT Translation
     v
Internet
     |
     v
External Server
```

The router receives the outbound packet from the private host and modifies the packet before sending it toward the internet.

The source address is translated from the computer's private IP address to the public IP address assigned to the network's internet connection.

---

## NAT and Port Translation

NAT commonly works together with Port Address Translation (PAT).

For example, the original connection may look like:

```text
Private Source:
192.168.1.x:65393

        ↓ NAT/PAT

Public Source:
<public-IP>:<translated-port>
```

The router maintains a translation table so that returning traffic can be associated with the correct internal device and connection.

This allows multiple devices on the same private network to share a single public IPv4 address.

The source port is important because the router can use the combination of the translated address and port to keep individual connections separate.

---

## Private vs. Public Addressing

The computer's private IP address is used for communication within the local network.

The public IP address represents the network when communicating with systems on the internet.

Conceptually:

| Address Type | Example                 | Purpose                     |
| ------------ | ----------------------- | --------------------------- |
| Private IPv4 | `192.168.1.x`           | Local network communication |
| Public IPv4  | `<sanitized-public-IP>` | Internet communication      |

The private address is not directly exposed as the source address of the packet after it leaves the router.

Instead, the router performs the NAT translation.

---

## Verifying the Public IP

The public-facing IP address was independently checked from Windows using:

```powershell
curl.exe https://api.ipify.org
```

This returned the public IP address assigned to the internet connection.

The important observation was that the public IP reported by the external service was different from the private IPv4 address observed in Wireshark.

This demonstrates the difference between the computer's local address and the address used by the network when communicating with the internet.

The actual public IP has been intentionally omitted from this documentation.

---

## Key Findings

* The Windows computer uses a private IPv4 address on the local network.
* The outbound HTTPS connection begins with the computer's private IP address.
* TCP port `443` is used for the HTTPS connection.
* The local router performs NAT between the LAN and WAN.
* The router translates the private source IP into the network's public IP.
* Port Address Translation allows multiple internal devices to share the same public IPv4 address.
* The private IP address is not directly routable across the public internet.
* Wireshark captures traffic from the computer's perspective, so a capture taken on the computer does not show the packet after the router performs NAT.
* An external service such as `api.ipify.org` can be used to determine the public IP address seen by the internet.

---

## What I Learned

This investigation helped me understand NAT as an actual network process rather than just a theoretical networking concept.

I was able to follow an outbound TCP connection from my computer, identify the private source address, and determine that the NAT translation happens at the router rather than on the Windows computer.

I also learned why a device can have a private IPv4 address while websites and other internet services see a completely different public IPv4 address.

Another important takeaway is that Wireshark captures traffic from the point where the capture is taken. Because this capture was performed on the Windows computer, I could see the packet before it reached the router, but I could not directly observe the router modifying the packet.

Understanding this distinction is useful when troubleshooting connectivity issues because the same traffic can look different depending on where the packet is captured.

---

## Troubleshooting / Real-World Relevance

NAT is important when troubleshooting connectivity because problems can occur at multiple points in the path:

```text
Client
  ↓
Local Network
  ↓
Router / NAT
  ↓
ISP
  ↓
Internet
  ↓
Destination
```

When troubleshooting an outbound connection, I would want to determine:

1. Does the client have a valid private IP address?
2. Does the client have a default gateway?
3. Can the client reach the local router?
4. Can the client resolve DNS names?
5. Can the client establish a TCP connection to the destination?
6. Is the router performing NAT correctly?
7. Is the destination service reachable?
8. Is a firewall blocking the connection?

This investigation demonstrates why packet captures should always be interpreted in the context of **where the capture was taken**.

---

## Conclusion

NAT allows private devices on a local network to communicate with the public internet without requiring every device to have its own public IPv4 address.

In this investigation, the Windows computer initiated an HTTPS connection using a private IPv4 address. The home router then performs NAT/PAT, translating the private source information into a public address before forwarding the traffic to the internet.

The investigation provided hands-on experience with private addressing, public addressing, TCP connections, NAT, PAT, and packet analysis using Wireshark.
