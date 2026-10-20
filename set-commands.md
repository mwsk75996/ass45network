# Set-kommandoer – Assignment 45

Hentet fra de kørende routere med `show configuration | display set` (uden `system` og `version`). Den delte nøgle er krypteret af Junos som `$9$…`.

## SRX-C1 (R1 / SRX-A)

```
# Interfaces
set interfaces ge-0/0/2 description "Network-A - Host-A 10.1.1.2 - trust"
set interfaces ge-0/0/2 unit 0 family inet address 10.1.1.1/24
set interfaces ge-0/0/3 description "Hub - SRX-C2 internet - untrust"
set interfaces ge-0/0/3 unit 0 family inet address 10.3.3.1/24
set interfaces st0 description "Secure tunnel to SRX-B"
set interfaces st0 unit 1 family inet

# Routing
set routing-options static route 0.0.0.0/0 next-hop 10.3.3.2
set routing-options static route 10.2.2.0/24 next-hop st0.1
set routing-options router-id 1.1.1.1

# IKE (fase 1)
set security ike proposal ike-phase1-proposal authentication-method pre-shared-keys
set security ike proposal ike-phase1-proposal dh-group group14
set security ike proposal ike-phase1-proposal authentication-algorithm sha-256
set security ike proposal ike-phase1-proposal encryption-algorithm aes-128-cbc
set security ike proposal ike-phase1-proposal lifetime-seconds 28800
set security ike policy ike-phase1-policy mode main
set security ike policy ike-phase1-policy proposals ike-phase1-proposal
set security ike policy ike-phase1-policy pre-shared-key ascii-text "$9$v6.W8xdb2aGiY2z36Au0N-Vbga5TFCtuHqT36C1I"
set security ike gateway gw-to-peer ike-policy ike-phase1-policy
set security ike gateway gw-to-peer address 10.4.4.1
set security ike gateway gw-to-peer external-interface ge-0/0/3.0

# IPsec (fase 2)
set security ipsec proposal ipsec-phase2-proposal protocol esp
set security ipsec proposal ipsec-phase2-proposal authentication-algorithm hmac-sha-256-128
set security ipsec proposal ipsec-phase2-proposal encryption-algorithm aes-128-cbc
set security ipsec proposal ipsec-phase2-proposal lifetime-seconds 3600
set security ipsec policy ipsec-phase2-policy perfect-forward-secrecy keys group14
set security ipsec policy ipsec-phase2-policy proposals ipsec-phase2-proposal
set security ipsec vpn vpn-to-peer bind-interface st0.1
set security ipsec vpn vpn-to-peer ike gateway gw-to-peer
set security ipsec vpn vpn-to-peer ike ipsec-policy ipsec-phase2-policy
set security ipsec vpn vpn-to-peer establish-tunnels immediately

# Security policies
set security policies from-zone trust to-zone vpn policy trust-to-vpn match source-address any
set security policies from-zone trust to-zone vpn policy trust-to-vpn match destination-address any
set security policies from-zone trust to-zone vpn policy trust-to-vpn match application any
set security policies from-zone trust to-zone vpn policy trust-to-vpn then permit
set security policies from-zone vpn to-zone trust policy vpn-to-trust match source-address any
set security policies from-zone vpn to-zone trust policy vpn-to-trust match destination-address any
set security policies from-zone vpn to-zone trust policy vpn-to-trust match application any
set security policies from-zone vpn to-zone trust policy vpn-to-trust then permit

# Zones
set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust host-inbound-traffic system-services ssh
set security zones security-zone trust interfaces ge-0/0/2.0
set security zones security-zone untrust interfaces ge-0/0/3.0 host-inbound-traffic system-services ike
set security zones security-zone untrust interfaces ge-0/0/3.0 host-inbound-traffic system-services ping
set security zones security-zone vpn interfaces st0.1
```

## SRX-C3 (R3 / SRX-B)

```
# Interfaces
set interfaces ge-0/0/2 description "Network-B - Host-B 10.2.2.2 - trust"
set interfaces ge-0/0/2 unit 0 family inet address 10.2.2.1/24
set interfaces ge-0/0/3 description "SRX-C2 internet - untrust"
set interfaces ge-0/0/3 unit 0 family inet address 10.4.4.1/24
set interfaces st0 description "Secure tunnel to SRX-A"
set interfaces st0 unit 1 family inet

# Routing
set routing-options static route 0.0.0.0/0 next-hop 10.4.4.2
set routing-options static route 10.1.1.0/24 next-hop st0.1
set routing-options router-id 3.3.3.3

# IKE (fase 1)
set security ike proposal ike-phase1-proposal authentication-method pre-shared-keys
set security ike proposal ike-phase1-proposal dh-group group14
set security ike proposal ike-phase1-proposal authentication-algorithm sha-256
set security ike proposal ike-phase1-proposal encryption-algorithm aes-128-cbc
set security ike proposal ike-phase1-proposal lifetime-seconds 28800
set security ike policy ike-phase1-policy mode main
set security ike policy ike-phase1-policy proposals ike-phase1-proposal
set security ike policy ike-phase1-policy pre-shared-key ascii-text "$9$xYNN-b24ZjkPJZ9ApBREsYg4Gjn6CO1RfT6ApOSy"
set security ike gateway gw-to-peer ike-policy ike-phase1-policy
set security ike gateway gw-to-peer address 10.3.3.1
set security ike gateway gw-to-peer external-interface ge-0/0/3.0

# IPsec (fase 2)
set security ipsec proposal ipsec-phase2-proposal protocol esp
set security ipsec proposal ipsec-phase2-proposal authentication-algorithm hmac-sha-256-128
set security ipsec proposal ipsec-phase2-proposal encryption-algorithm aes-128-cbc
set security ipsec proposal ipsec-phase2-proposal lifetime-seconds 3600
set security ipsec policy ipsec-phase2-policy perfect-forward-secrecy keys group14
set security ipsec policy ipsec-phase2-policy proposals ipsec-phase2-proposal
set security ipsec vpn vpn-to-peer bind-interface st0.1
set security ipsec vpn vpn-to-peer ike gateway gw-to-peer
set security ipsec vpn vpn-to-peer ike ipsec-policy ipsec-phase2-policy
set security ipsec vpn vpn-to-peer establish-tunnels immediately

# Security policies
set security policies from-zone trust to-zone vpn policy trust-to-vpn match source-address any
set security policies from-zone trust to-zone vpn policy trust-to-vpn match destination-address any
set security policies from-zone trust to-zone vpn policy trust-to-vpn match application any
set security policies from-zone trust to-zone vpn policy trust-to-vpn then permit
set security policies from-zone vpn to-zone trust policy vpn-to-trust match source-address any
set security policies from-zone vpn to-zone trust policy vpn-to-trust match destination-address any
set security policies from-zone vpn to-zone trust policy vpn-to-trust match application any
set security policies from-zone vpn to-zone trust policy vpn-to-trust then permit

# Zones
set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust host-inbound-traffic system-services ssh
set security zones security-zone trust interfaces ge-0/0/2.0
set security zones security-zone untrust interfaces ge-0/0/3.0 host-inbound-traffic system-services ike
set security zones security-zone untrust interfaces ge-0/0/3.0 host-inbound-traffic system-services ping
set security zones security-zone vpn interfaces st0.1
```
