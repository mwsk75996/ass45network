# Inventory – Assignment 45 (Site-to-Site VPN)

Enhederne svarer til [topologien](topology.jpg). Adresser og VPN-parametre står i [LLD](LLD.md).

## Routere

- **R1 / SRX-C1 (SRX-A)** – Juniper SRX240H-POE, serienr. AH4113AK0093, Junos 12.1X46-D55.3. ge-0/0/2 10.1.1.1/24 (Network-A), ge-0/0/3 10.3.3.1/24 (mod hub), st0.1 tunnel. Config: [r1-config.json](r1-config.json).
- **R2 / SRX-C2 (internet)** – Juniper SRX240H, serienr. AG4809AA0482, Junos 12.1X46-D30.2. ge-0/0/1 10.3.3.2/24 (mod hub), ge-0/0/2 10.4.4.2/24 (mod R3). DHCP-server for Host-Wireshark. Config: [r2-config.json](r2-config.json).
- **R3 / SRX-C3 (SRX-B)** – Juniper SRX240H2, serienr. BU3813AK0357, Junos 12.1X46-D30.2. ge-0/0/3 10.4.4.1/24 (mod R2), ge-0/0/2 10.2.2.1/24 (Network-B), st0.1 tunnel. Config: [r3-config.json](r3-config.json).

## Netværksudstyr

- **Hub** mellem R1 ge-0/0/3, R2 ge-0/0/1 og Host-Wireshark. Skal være en hub og ikke en switch, så Host-Wireshark ser trafikken mellem R1 og R2.
- **6 Ethernet-kabler**: R1–Host-A, R1–hub, hub–R2, hub–Host-Wireshark, R2–R3, R3–Host-B.
- **3 konsolkabler**: USB-seriel (FTDI FT232R) til RJ45-konsol, 9600 baud, 8N1, ingen flow control.

## PC'er

- **Host-A** – statisk 10.1.1.2/24, gateway 10.1.1.1.
- **Host-B** – statisk 10.2.2.2/24, gateway 10.2.2.1.
- **Host-Wireshark** – Linux med Wireshark, DHCP-klient på hub-segmentet (fik 10.3.3.10/24, gateway 10.3.3.2). Capture på det fysiske interface (`enp0s31f6`) i promiscuous mode, ikke "any".
- **Konfigurations-PC** – Linux med `screen`, forbundet til de tre routere via konsolkablerne.

## VPN

- Route-baseret IPsec mellem R1 (10.3.3.1) og R3 (10.4.4.1) over st0.1.
- IKE: main mode, pre-shared key, DH group14, SHA-256, AES-128-CBC.
- IPsec: ESP, HMAC-SHA-256-128, AES-128-CBC, PFS group14.
