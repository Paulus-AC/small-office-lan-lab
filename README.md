# Small Office LAN (GNS3 Lab)

Hands-on GNS3 lab that simulates a small government office LAN with multiple floors, VLAN segmentation, inter-VLAN routing, DHCP, EtherChannel, and port security.

This repository is part of my networking portfolio as a CCNA-certified network engineer. It shows how I design, build, and validate Layer 2/Layer 3 campus networks using only virtual lab tools.

---

## 🏛 Scenario

City Hall has four departments spread across two floors:

- Admin (VLAN 10)
- Finance (VLAN 20)
- HR (VLAN 30)
- IT (VLAN 40)

Each department must:

- Be isolated in its own VLAN for security and broadcast containment.
- Reach other departments and shared services through a central router.
- Obtain addresses automatically via DHCP.

The physical layout is two access switches (1st floor and 2nd floor) uplinked to a router, with an EtherChannel between the switches for redundancy and more bandwidth.

---

## 🧱 Topology and addressing

**Devices**

- R1 – Router-on-a-stick (inter-VLAN routing + DHCP)
- 1stFloorSW – Access switch (Admin & Finance)
- 2ndFloorSW – Access switch (HR & IT)
- 4× PCs (VPCS or similar) – one per department

**VLANs and IP subnets**

- VLAN 10 – Admin – `192.168.10.0/24` – gateway `192.168.10.1`
- VLAN 20 – Finance – `192.168.20.0/24` – gateway `192.168.20.1`
- VLAN 30 – HR – `192.168.30.0/24` – gateway `192.168.30.1`
- VLAN 40 – IT – `192.168.40.0/24` – gateway `192.168.40.1`

**Layer 2 design**

- Trunk between R1 and 1stFloorSW carrying VLANs 10, 20, 30, 40
- LACP EtherChannel (Port-channel1) between 1stFloorSW and 2ndFloorSW, carrying VLANs 10–40
- Sticky MAC port security on all user access ports
- PVST spanning tree, with 1stFloorSW as root for user VLANs (optional enhancement)

**Diagram**

![Topology](gov-office-vlan-lab/topology/topology-diagram.png)

---

## 🔧 Technologies demonstrated

- VLANs and 802.1Q trunking
- Router-on-a-stick inter-VLAN routing
- DHCP server on the router with excluded ranges
- EtherChannel (LACP) between access switches
- Port security (sticky MAC addresses) on access ports
- Per-VLAN spanning tree (PVST)
- Basic failure and recovery testing (link failures, port-security violations)

This style of lab is commonly used in CCNA/CCNP home labs and network design portfolios.[cite:1]

---

## 📄 Files and folders

- `gov-office-vlan-lab/`
  - `project-files/` – GNS3 internal project data
  - `configs/` – router and switch startup configurations
  - `topology/topology-diagram.png` – network diagram
  - `docs/test-plan.md` – configuration checklist and test cases

---

## 🧑‍💻 About me

I am a CCNA-certified network engineer working in government IT.  
This lab is a virtual version of the kind of small office networks I design and support (VLAN segmentation, inter-VLAN routing, DHCP, and access-layer security) using only my laptop and GNS3.
