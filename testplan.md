# Testplan – Assignment 45 (Site-to-Site VPN)

Udført 2026-10-20 via seriel konsol på SRX-C1 (R1), SRX-C2 (R2) og SRX-C3 (R3). Enheder og adresser som i [topologien](topology.jpg) og [LLD](LLD.md).

| # | Test | Fra | Til / kommando | Forventet | Resultat | Status |
|---|---|---|---|---|---|---|
| 1 | Interfaces oppe | R1, R2, R3 | `show interfaces terse` | ge-0/0/x og st0.1 up/up med korrekte adresser | Alle up/up, adresser som LLD | Bestået |
| 2 | Underlay R1 → R2 | R1 | `ping 10.3.3.2` | Svar | 3/3, 0 % tab | Bestået |
| 3 | Underlay R3 → R2 | R3 | `ping 10.4.4.2` | Svar | 3/3, 0 % tab | Bestået |
| 4 | Underlay R1 → R3 (tunnel-endepunkter) | R1 | `ping 10.4.4.1` | Svar via default route | 3/3, 0 % tab | Bestået |
| 5 | IKE fase 1 oppe | R1, R3 | `show security ike security-associations` | State UP, Main mode | UP, Main, peer 10.4.4.1 / 10.3.3.1 | Bestået |
| 6 | IPsec fase 2 oppe | R1, R3 | `show security ipsec security-associations` | 1 aktiv tunnel, ESP aes-cbc-128/sha256 | 1 tunnel, ESP:aes-cbc-128/sha256 | Bestået |
| 7 | Route til fjernt LAN via tunnel | R1 / R3 | `show route 10.2.2.0/24` / `10.1.1.0/24` | Next-hop st0.1 | Static/5 via st0.1 på begge | Bestået |
| 8 | R2 kender ikke de private LAN | R2 | `show route 10.1.1.0/24`, `10.2.2.0/24` | Ingen route | Ingen route | Bestået |
| 9 | R2 kan ikke nå hostene direkte | R2 | `ping 10.1.1.2`, `ping 10.2.2.2` | Fejler | "No route to host", 100 % tab | Bestået |
| 10 | Network-A → Network-B gennem tunnel | R1 (src 10.1.1.1) | `ping 10.2.2.1` og `10.2.2.2` (Host-B) | Svar | 5/5 og 5/5, 0 % tab | Bestået |
| 11 | Network-B → Network-A gennem tunnel | R3 (src 10.2.2.1) | `ping 10.1.1.1` og `10.1.1.2` (Host-A) | Svar | 5/5 og 5/5, 0 % tab | Bestået |
| 12 | Trafikken krypteres | R1 | `show security ipsec statistics` før/efter test 10 | Tællere stiger | Encrypted/Decrypted 2777 → 2787 | Bestået |
| 13 | Host-A ↔ Host-B | Host-A / Host-B | `ping 10.2.2.2` / `ping 10.1.1.2` | Svar begge veje | Svar begge veje (IPsec-tællere fulgte med) | Bestået |
| 14 | DHCP til Host-Wireshark | Host-Wireshark | DHCP på hub-segmentet | Adresse i 10.3.3.10–20, GW 10.3.3.2 | Fik 10.3.3.10 (`show dhcp server binding` på R2) | Bestået |
| 15 | Kun krypteret trafik på hub-segmentet | Host-Wireshark | Wireshark på `enp…`, filter `ip.addr==10.3.3.1 && ip.addr==10.4.4.1 && (esp \|\| isakmp)` under test 13 | Kun ESP mellem 10.3.3.1 og 10.4.4.1, ingen 10.1.1.x/10.2.2.x synlig | ESP mellem tunnel-endepunkterne, indhold ulæseligt | Bestået |
| 16 | Tunnel genopbygges | R1 | `clear security ike security-associations` og derefter `clear security ipsec security-associations` | Ny IKE- og IPsec-SA uden manuel indgriben | Ny IKE-SA (cookie ba4ae4f3…) og IPsec-SA (SPI d060d0ae) inden for 15 s, ping til Host-B 5/5 | Bestået |

## Konklusion

Tunnelen mellem SRX-C1 og SRX-C3 er oppe og sender trafik mellem Network-A og Network-B begge veje (test 10–13). SRX-C2 har ingen route til de private net og kan ikke nå hostene, så forbindelsen kun findes gennem tunnelen (test 8–9). På hub-segmentet ses kun ESP mellem 10.3.3.1 og 10.4.4.1 (test 15), og tunnelen kommer selv op igen efter den er ryddet (test 16).

Bemærk til test 15: Wireshark skal lytte på det fysiske interface i promiscuous mode. På "any" fanger Linux kun PC'ens egen trafik.
