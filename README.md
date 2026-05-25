# Vlan-trunking-lab 
Cisco Packet Tracer lab exploring 802.1Q trunking, access ports, and L2 network segmentation.
<img width="730" height="350" alt="image" src="https://github.com/user-attachments/assets/b06475fe-fe6a-42ad-86f0-c85d544e98a9" />

## 📌 Project Objective
The objective of this lab is to build and analyze a standard Layer 2 Virtual Local Area Network (VLAN) topology across multiple switches using Cisco Packet Tracer. I did this to better my understanding of VLANs. I am a visual and question learner,so this project and crafting this doc helped me understand network segmentation, access port mapping and trunking links to carry multi-VLAN traffic across a single physical medium.


## 🛠️ Topology & Hardware Setup
- **Switches:** 2x Cisco Catalyst 2960 (S1 and S2)
- **Hosts:** 4x End Devices (PCs)
- **Inter-Switch Link:** Connected via `FastEthernet 0/1` on both switches using a **copper cross-over cable**.

### 💻 Addressing & Allocation Table

| Device Name | Connected Port | VLAN Assignment | IP Address | Subnet Mask |
| :--- | :--- | :--- | :--- | :--- |
| **PC-A** | S1 / Fa0/10 | VLAN 10 (NBA) | 192.168.10.11 | 255.255.255.0 |
| **PC-B** | S1 / Fa0/20 | VLAN 20 (NFL) | 192.168.20.11 | 255.255.255.0 |
| **PC-C** | S2 / Fa0/10 | VLAN 10 (NBA) | 192.168.10.12 | 255.255.255.0 |
| **PC-D** | S2 / Fa0/20 | VLAN 20 (NFL) | 192.168.20.12 | 255.255.255.0 |

---

## 🧠 Core Technical Concepts & Lessons Learned
I asked myself questions while doing the lab, to understand

### 1. Medium Selection: Crossover vs. Straight-Through Cables
* **The Question:** Why use a copper cross-over cable instead of a standard straight-through cable between the switches?
* **The Answer:** Switches are identical Layer 2 devices. They share the same internal pin layout for transmission (Tx) and reception (Rx). A standard straight-through cable connects Tx-to-Tx and Rx-to-Rx, resulting in signal collisions. The crossover cable physically crosses the transmission and reception pairs inside the sheath, allowing the data streams to align properly. *

### 2. The Mechanics of a VLAN
* **Where the Magic Happens:** I had the mistaken idea that VLANs were wireless, but VLANs separate info by tagging, not at the PC, but at the ports; this is where the magic happens. The PC remains entirely unaware of VLAN tagging, so VLANS are are a physical wire thing. I seem to have had VLANS confused with wireless networks.

### 3. Why static?
* **Static IP vs. DHCP:** When assigning IP addresses via the device menu, I saw an option for static and one for DHCP? It made sense to me becuase the nature of DHCP is to automatically assign network details (including IP addresses) to network devices that query via broadcast message. IP addresses were assigned statically in this lab, as I did it manually.

### 3. The Failure of Access Ports for Inter-Switch Links
* **The question:** What happens if the link between S1 and S2 is left as a normal access port instead of a trunk? I thought that the work was done on the PC ports. I didn't see the need for there to be another thing between the switches. I thought that if the information is tagged already at the switch ports, it should be fine to have a normal link between the two switches.
* **The Reality:** I learnt that Access ports are incapable of transmitting frames with explicit  VLAN tags; they always strip the tags before forwarding traffic out of the link. If S1 and S2 are connected via an access port, only one VLAN can successfully pass through, while traffic from other VLANs is instantly dropped at the interface edge. In order to preserve both, a trunk has to be used to allow simultaneous tagged data transmission.

---

## 💻 Cisco IOS Configuration Steps
Here are the commands i entered 

### 1. Create the VLAN Databases (S1 & S2)
VLAN IDs must explicitly exist within each switch's database to accept and forward corresponding frames.
```text
S1> enable
S1# configure terminal
S1(config)# vlan 10
S1(config-vlan)# name NBA
S1(config-vlan)# exit
S1(config)# vlan 20
S1(config-vlan)# name PFL
S1(config-vlan)# exit
```
*(I am an avid sports fan so i used names for famous sports organisations. I visualised that i was in a video streaming network scenario and I wanted to separate the video input of two different sports to two different screens.)

*(Repeated identically on S2).*

### 2. Map Host-Facing Edge Interfaces to Access VLANs
```text
S1(config)# interface FastEthernet 0/10
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10
S1(config-if)# exit

S1(config)# interface FastEthernet 0/20
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 20
S1(config-if)# exit
```
*(Repeated on S2 for its corresponding ports).*

### 3. Establish theInter-Switch Trunk Link
```text
S1(config)# interface FastEthernet 0/1
S1(config-if)# switchport mode trunk
S1(config-if)# exit
```
*(Repeated on S2 to guarantee predictable negotiation).*

---

## 🔍 Verification & Connectivity Analysis

### Verification Commands Run
- `show vlan brief`: Confirmed `Fa0/10` and `Fa0/20` are active in their respective HR and IT VLAN blocks.
- `show interfaces trunk`: Confirmed `Fa0/1` is actively operating as an operational 802.1Q trunk.

### Results 
1. **Intra-VLAN Ping (`PC-A` to `PC-C`):** **SUCCESSFUL** ✅  
   *Because they are on the same network *
2. **Inter-VLAN Ping (`PC-A` to `PC-B`):** **FAILED** ❌  
   *Because they are on different networks, i didn't use a router so this was expected

## :chart_with_upwards_trend: Learning
- From this lab i learnt how helpful trunk ports can be when dealing with multiple streams of VLAN traffic, keeping  data seperate when one line is used to carry 
many different types of sensitive data to the same destination via the same line.
- I also learnt to avoid using access ports for interswitch links when dealing with VLANs, the access port will strip the packets of their vlan id.
