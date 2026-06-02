---
layout: lab
title: VLAN L2 Troubleshooting
description: "A Cisco Packet Tracer lab exploring 802.1Q trunking, access ports, and L2 network segmentation across multiple switches."
diagram: /Vlan-trunking-lab/vlantrunk.png

concepts:
  - Layer 2 Switching
  - 802.1Q Trunking
  - Access Ports
  - VLAN Segmentation
  - Network Troubleshooting

objective: >
  The objective of this lab is to build and analyze a standard Layer 2 VLAN topology across multiple switches using Cisco Packet Tracer.
  The goal was to better understand network segmentation, access port mapping, and trunking links to carry multi-VLAN traffic across a single physical medium.

hardware:
  - "2x Cisco Catalyst 2960 (S1 and S2)"
  - "4x End Devices (PCs)"
  - "Copper Cross-Over Cable (Inter-Switch Link)"

questions:
  - q: Why use a copper cross-over cable instead of a straight-through cable between the switches?
    a: >
      Switches are identical Layer 2 devices with the same internal pin layout for transmission (Tx) and reception (Rx).
      A straight-through cable connects Tx-to-Tx and Rx-to-Rx, causing signal collisions.
      The crossover cable physically crosses the pairs, allowing data streams to align properly.

  - q: Where does the VLAN tagging actually happen?
    a: >
      VLANs separate traffic by tagging at the switch ports, not at the PC. The PC remains entirely unaware of VLAN tagging.
      This means VLANs are a physical wire thing — not wireless. I had confused VLANs with wireless networks initially.

  - q: Why were IP addresses assigned statically instead of using DHCP?
    a: >
      DHCP automatically assigns network details to devices via broadcast. In this lab, IPs were assigned manually
      to keep things simple and controlled — no DHCP server was needed for a basic L2 topology.

  - q: What happens if the inter-switch link is left as an access port instead of a trunk?
    a: >
      Access ports strip VLAN tags before forwarding traffic. If S1 and S2 are connected via an access port,
      only one VLAN can pass through while traffic from other VLANs is dropped at the interface edge.
      A trunk port must be used to allow simultaneous tagged data transmission across both VLANs.

steps:
  - title: Create VLAN databases on S1 and S2
    desc: VLAN IDs must explicitly exist within each switch's database to accept and forward corresponding frames.
    code: |
      S1> enable
      S1# configure terminal
      S1(config)# vlan 10
      S1(config-vlan)# name NBA
      S1(config-vlan)# exit
      S1(config)# vlan 20
      S1(config-vlan)# name NFL
      S1(config-vlan)# exit

  - title: Map host-facing ports to access VLANs
    desc: Assign each PC-facing port to its respective VLAN on both switches.
    code: |
      S1(config)# interface FastEthernet 0/10
      S1(config-if)# switchport mode access
      S1(config-if)# switchport access vlan 10
      S1(config-if)# exit

      S1(config)# interface FastEthernet 0/20
      S1(config-if)# switchport mode access
      S1(config-if)# switchport access vlan 20
      S1(config-if)# exit

  - title: Establish the inter-switch trunk link
    desc: Configure the link between S1 and S2 as a trunk to carry traffic from both VLANs simultaneously.
    code: |
      S1(config)# interface FastEthernet 0/1
      S1(config-if)# switchport mode trunk
      S1(config-if)# exit

  - title: Verify configuration
    desc: Confirm VLANs and trunk are active using verification commands.
    code: |
      S1# show vlan brief
      S1# show interfaces trunk

learnings:
  - Trunk ports are essential when carrying multiple VLANs across a single inter-switch link.
  - Access ports strip VLAN tags — never use them for inter-switch links in a multi-VLAN setup.
  - VLAN tagging happens at the switch port, not the end device — PCs are unaware of VLANs.
  - Intra-VLAN pings succeed; inter-VLAN pings fail without a Layer 3 device (router or L3 switch).
---
