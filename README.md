# Assignment 45 – Site-to-Site VPN

Konfigurationer til de tre SRX-routere i JSON: `r1-config.json`, `r2-config.json` og `r3-config.json`.

## Routerne

- **R1 / SRX-C1 (SRX-A)** – har Network-A med Host-A. Bygger en route-baseret IPsec-tunnel (st0.1) til SRX-C3 og sender trafik til 10.2.2.0/24 ind i tunnelen.
- **R2 / SRX-C2 (internet)** – spiller internettet mellem de to sites. Kender ikke 10.1.1.0/24 eller 10.2.2.0/24 og deler en adresse ud til Host-Wireshark med DHCP (10.3.3.10–.20, gateway 10.3.3.2).
- **R3 / SRX-C3 (SRX-B)** – har Network-B med Host-B. Er den anden ende af tunnelen og sender trafik til 10.1.1.0/24 ind i st0.1.

IKE: main mode, pre-shared key, DH group14, SHA-256, AES-128. IPsec: ESP, HMAC-SHA-256, AES-128, PFS group14.

## Adresser (LLD)

| Router | Interface | Netværk | Adresse |
|---|---|---|---|
| SRX-C1 | ge-0/0/2 | Network-A 10.1.1.0/24 (trust) | 10.1.1.1/24 (Host-A = .2) |
| SRX-C1 | ge-0/0/3 | Hub 10.3.3.0/24 (untrust) | 10.3.3.1/24 |
| SRX-C1 | st0.1 | Tunnel (vpn) | unnumbered |
| SRX-C2 | ge-0/0/1 | Hub 10.3.3.0/24 | 10.3.3.2/24 (Host-Wireshark = DHCP .10–.20) |
| SRX-C2 | ge-0/0/2 | 10.4.4.0/24 | 10.4.4.2/24 |
| SRX-C3 | ge-0/0/3 | 10.4.4.0/24 (untrust) | 10.4.4.1/24 |
| SRX-C3 | ge-0/0/2 | Network-B 10.2.2.0/24 (trust) | 10.2.2.1/24 (Host-B = .2) |
| SRX-C3 | st0.1 | Tunnel (vpn) | unnumbered |
