# CN Lab-02: Packet Sniffer and Packet Analyzer

## What this is

For this lab, a packet capture (`chandan1.pcapng`) was taken in Wireshark while
accessing two websites from the terminal using `curl`:

1. `http://neverssl.com/` — a plain HTTP site
2. `https://www.cornell.edu/` — an HTTPS site

The capture was analyzed to answer the lab manual's questions (protocols observed,
amount of data received, HTTP GET request details, and the TCP three-way handshake).
The full write-up is in `LAB_CN_02_Answers.docx`.

## How the analysis was done

1. Opened the capture file in Wireshark.
2. Looked at the DNS query/response for `neverssl.com` and `www.cornell.edu` to find
   the actual server IP address each one resolved to (you need the IP before you can
   filter on it — filtering by hostname directly doesn't work at the packet level).
3. Filtered on that IP address (`ip.addr == <IP>`) to isolate only the packets
   belonging to that site, and ignored everything else in the capture — there was a
   lot of unrelated background traffic (Google services, WhatsApp, NTP, QUIC) that
   had nothing to do with the curl commands.
4. Used "Follow TCP Stream" on the HTTP session to read the GET request and response
   headers directly.
5. Noted down the packet numbers and sequence numbers for the SYN / SYN-ACK / ACK
   packets that make up the TCP handshake.

## Things worth noting

- The capture has a lot of IPv6 and QUIC (UDP port 443) traffic that isn't related to
  the curl commands at all — it's background traffic from other apps on the laptop.
  This was excluded from the answers, just noted as background noise.
- `neverssl.com` only has an IPv4 address and doesn't support HTTPS (hence the name
  "NeverSSL") — every attempt to connect on port 443 got reset immediately.
- `www.cornell.edu` sits behind Azure Front Door (visible from the CNAME chain in the
  DNS response) and uses HTTPS/TLS over IPv6 (`2603:1061:14:154::1`).

## Files

| File | Description |
|---|---|
| `LAB_CN_02_Answers.docx` | Full detailed answers to all lab questions (Word format) |
| `README.md` | This file — summary of the analysis |

## Quick summary of answers

| Question | Answer (short) |
|---|---|
| Application layer protocols | HTTP, HTTPS/TLS, DNS (+ NTP, QUIC in background) |
| Transport layer protocols | TCP, UDP |
| Network layer protocols | IPv4, IPv6, ICMP/ICMPv6, ARP |
| Data received — neverssl.com | ~4.2 KB |
| Data received — cornell.edu | ~128 KB |
| GET requests to neverssl.com | 1 (`GET / HTTP/1.1`) |
| Server HTTP version | HTTP/1.1 (Apache/2.4.66) |
| TCP segments in response | 4 data segments (7 total incl. control packets) |
| HTTP response size | 4261 bytes (300 header + 3961 body) |
| Three-way handshake | SYN (pkt 52) → SYN-ACK (pkt 54) → ACK (pkt 55) |
