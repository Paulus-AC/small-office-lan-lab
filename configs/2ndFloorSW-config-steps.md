# 2ndFloorSW – Small Office LAN Configuration

This document shows the step-by-step configuration for **2ndFloorSW** (2nd floor access switch).

Responsibilities:
- LACP EtherChannel back to 1stFloorSW
- Access ports for HR and IT VLANs
- Sticky MAC port security on user ports

---

## Step 1 – Basic setup

```plaintext
enable
configure terminal

hostname 2ndFloorSW
no ip domain-lookup

spanning-tree mode pvst
spanning-tree extend system-id
```

---

## Step 2 – Create VLANs

Create the same VLANs as on 1stFloorSW.

```plaintext
vlan 10
 name ADMIN
vlan 20
 name FINANCE
vlan 30
 name HR
vlan 40
 name IT
vlan 99
 name MGMT
```

Verify:

```plaintext
do show vlan brief
```

---

## Step 3 – Configure LACP EtherChannel to 1stFloorSW

Use Ethernet0/1 and Ethernet0/2 as member links into Port-channel1.

```plaintext
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown

interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
```

Verify:

```plaintext
do show etherchannel summary
do show interfaces trunk
```

---

## Step 4 – Configure access ports for HR and IT

HR‑PC on VLAN 30, IT‑PC on VLAN 40, with sticky MAC port security.

```plaintext
interface Ethernet1/0
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security mac-address sticky
 no shutdown

interface Ethernet1/1
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security mac-address sticky
 no shutdown
```

Verify:

```plaintext
do show port-security interface e1/0
do show port-security interface e1/1
```

---

## Step 5 – Disable unused SVI

```plaintext
interface Vlan1
 shutdown
```

---

## Step 6 – Line configuration (optional for lab)

```plaintext
line con 0
 logging synchronous
 exec-timeout 0 0

line vty 0 4
 login
 transport input telnet ssh
```

---

## Step 7 – Verification summary

After the whole lab is up:

```plaintext
show vlan brief
show interfaces trunk
show etherchannel summary
show port-security interface e1/0
show port-security interface e1/1
```
