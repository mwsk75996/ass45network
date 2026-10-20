# LLD – Assignment 45 (Site-to-Site VPN)

## Interfaces og adresser

| Enhed | Interface | Netværk | Adresse | Zone |
|---|---|---|---|---|
| R1 / SRX-C1 | ge-0/0/2 | Network-A 10.1.1.0/24 | 10.1.1.1/24 | trust |
| R1 / SRX-C1 | ge-0/0/3 | Hub 10.3.3.0/24 | 10.3.3.1/24 | untrust |
| R1 / SRX-C1 | st0.1 | Tunnel | unnumbered | vpn |
| R2 / SRX-C2 | ge-0/0/1 | Hub 10.3.3.0/24 | 10.3.3.2/24 | internet |
| R2 / SRX-C2 | ge-0/0/2 | Transit 10.4.4.0/24 | 10.4.4.2/24 | internet |
| R3 / SRX-C3 | ge-0/0/3 | Transit 10.4.4.0/24 | 10.4.4.1/24 | untrust |
| R3 / SRX-C3 | ge-0/0/2 | Network-B 10.2.2.0/24 | 10.2.2.1/24 | trust |
| R3 / SRX-C3 | st0.1 | Tunnel | unnumbered | vpn |
| Host-A | NIC | Network-A | 10.1.1.2/24, GW 10.1.1.1 | – |
| Host-B | NIC | Network-B | 10.2.2.2/24, GW 10.2.2.1 | – |
| Host-Wireshark | enp… | Hub | DHCP 10.3.3.10–20, GW 10.3.3.2 | – |

## Routing

| Router | Destination | Next-hop |
|---|---|---|
| R1 | 0.0.0.0/0 | 10.3.3.2 |
| R1 | 10.2.2.0/24 | st0.1 |
| R3 | 0.0.0.0/0 | 10.4.4.2 |
| R3 | 10.1.1.0/24 | st0.1 |
| R2 | kun direkte forbundne netværk | – |

R2 kender ikke 10.1.1.0/24 og 10.2.2.0/24, så trafik mellem Host-A og Host-B kun kan gå gennem tunnelen.

## VPN

| Parameter | Fase 1 (IKE) | Fase 2 (IPsec) |
|---|---|---|
| Peers | R1 10.3.3.1 ↔ R3 10.4.4.1 | bind-interface st0.1 |
| Mode / protokol | Main mode | ESP |
| Autentificering | Pre-shared key, SHA-256 | HMAC-SHA-256-128 |
| Kryptering | AES-128-CBC | AES-128-CBC |
| DH / PFS | group14 | PFS group14 |
| Lifetime | 28800 s | 3600 s |
| Etablering | – | establish-tunnels immediately |

## Security policies (R1 og R3)

| Fra zone | Til zone | Handling |
|---|---|---|
| trust | vpn | permit |
| vpn | trust | permit |

Host-inbound-traffic: trust tillader ping og ssh, untrust tillader ike og ping. R2 har én permit-policy fra zone internet til zone internet og tillader DHCP på ge-0/0/1.

## Øvrigt

| Enhed | Hostname | Router-id |
|---|---|---|
| R1 | SRX-C1 | 1.1.1.1 |
| R2 | SRX-C2 | 2.2.2.2 |
| R3 | SRX-C3 | 3.3.3.3 |
