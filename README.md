# Design and Deployment of a Multi-Client WAN with VRF Segmentation

<p align="center">
  <img src="https://www.gns3.com/assets/images/logo-colour.svg" alt="GNS3 Logo" width="250">
</p>
<p align="center">
  <strong>A GNS3 simulation of a secure, scalable, and multi-tenant MPLS-based WAN.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-GNS3-blue" alt="Platform: GNS3">
  <img src="https://img.shields.io/badge/Technology-MPLS-orange" alt="Technology: MPLS">
  <img src="https://img.shields.io/badge/Routing-MP--BGP%20%26%20IS--IS-green" alt="Routing: BGP & IS-IS">
  <img src="https://img.shields.io/badge/Security-VRF%20%26%20VLAN-red" alt="Security: VRF & VLAN">
</p>

## 📖 Project Overview

This project showcases the design and implementation of a sophisticated multi-branch Wide Area Network (WAN) built and simulated entirely within **GNS3**. The architecture is designed to serve multiple clients, ensuring complete traffic isolation through a combination of Layer 2 VLANs and Layer 3 VRFs, alongside secure, centralized management.

The core of the network leverages an **MPLS backbone** with **MP-BGP** to provide scalable Layer 3 VPN services. Virtual Routing and Forwarding (**VRF**) instances create logical separation between clients, management, and internet traffic. The topology is a complete model of a service provider network, including a core, multiple customer sites, dedicated internet gateways, and connections to simulated ISPs.

---

## ✨ Key Features & Technologies

* **Multi-Tenant Architecture**: Supports multiple clients on a shared infrastructure with complete logical isolation.
* **MPLS L3 VPNs**: Utilizes MPLS and MP-BGP to create scalable and secure VPNs for each client.
* **Advanced VRF Segmentation**:
    * [cite_start]**Client VRFs**: Isolate traffic between different clients across the WAN[cite: 1088, 1089, 1090].
    * [cite_start]**Management VRF**: A dedicated VRF named `Gestion` is configured for secure, out-of-band management access[cite: 1091].
    * [cite_start]**Internet/IXP VRFs**: Separate VRFs are deployed on border routers to manage external connectivity to multiple ISPs and an Internet Exchange Point[cite: 1092, 1093, 1094].
* **Robust Routing Protocols**:
    * [cite_start]**IS-IS**: Used as the Interior Gateway Protocol (IGP) for fast convergence and scalability within the MPLS backbone[cite: 1097, 1109].
    * [cite_start]**MP-BGP**: Manages the distribution of VPNv4 routes between PE routers, with BGP AS **65000** for the provider network[cite: 1110].
    * [cite_start]**Redundant Route Reflectors**: Two route reflectors are used to ensure routing consistency and high availability[cite: 1110].
* **End-to-End Segmentation**:
    * [cite_start]Client traffic is segregated into distinct **VLANs** (e.g., 101, 102, 103) at the customer premises[cite: 1102, 1103, 1104].
    * [cite_start]A dedicated **Management VLAN (500)** provides a secure management plane across all devices[cite: 1105].

---

## 🌐 Network Topology & Components

* **P (Provider) Routers**: Devices that form the high-speed core of the MPLS backbone. Their sole purpose is to perform label switching. [cite_start]They run IS-IS and MPLS but have no VRFs or BGP awareness[cite: 839, 906].
* [cite_start]**PE (Provider Edge) Routers**: Multi-service edge routers that manage customer VRFs, run MP-BGP, and connect to customer sites[cite: 422, 584, 612, 861].
* **Border Routers**: Specialized PE routers (`DCBGP1`, `DCBGP2`) that handle all external connectivity. [cite_start]They manage the ISP and IXP VRFs and peer with external networks[cite: 33, 771].
* **CE (Customer Edge) Routers**: Routers like `CE-CLIENTE1-HERRERA` that belong to the customer. [cite_start]They run a standard eBGP session with the provider's PE router to exchange routes[cite: 369, 409].
* [cite_start]**CPE (Customer Premises Equipment) Switches**: Layer 2 switches that act as the demarcation point between the provider's core switch and the customer's CE router[cite: 53, 354, 494].
* [cite_start]**ISP & IXP Routers**: Simulated external networks that peer with the provider's Border Routers to provide internet connectivity[cite: 484, 761, 819].

---

## 🔬 Routing Architecture in Detail

### Core Routing (IS-IS & MPLS)

The stability of the network is built on the core. [cite_start]IS-IS is enabled on all P, PE, and Border routers within the provider network[cite: 527, 839]. [cite_start]It is responsible for advertising the loopback addresses of the PE/Border routers, which are used to establish iBGP peerings[cite: 527, 1097]. [cite_start]MPLS is enabled on all core-facing interfaces, allowing P routers to switch packets based on labels[cite: 528, 529, 530].

### Internal BGP (iBGP) for VPNs

The MPLS VPN service is powered by Multi-Protocol BGP. [cite_start]All PE and Border routers peer with two redundant Route Reflectors[cite: 1110, 341]. [cite_start]The `address-family vpnv4` is used to exchange customers' routes, which are prepended with a Route Distinguisher to ensure uniqueness and carry Route Target communities to control the import/export of routes into the correct VRFs[cite: 1111, 342].

### Customer Edge Routing (eBGP)

The connection to each customer is standardized and robust.
* [cite_start]Each customer is placed in a unique Autonomous System (e.g., Client 1 is in **AS 65010**, Client 2 in **AS 65020**, and Client 3 in **AS 65030**)[cite: 417, 562, 1132].
* The customer's CE router establishes a simple eBGP session with the provider's PE router from within its designated VRF and VLAN. [cite_start]For example, `CE-CLIENTE1-HERRERA` (in AS 65010) peers with the local PE at `10.20.10.9` (in AS 65000)[cite: 377].
* [cite_start]Each CE router advertises its internal networks, such as a `Loopback10` interface, into BGP for end-to-end reachability tests[cite: 372, 378].
* For added resiliency, some CE routers are configured to be **multi-homed**, peering with more than one PE router. [cite_start]For example, `CE-CLIENTE1-HERRERA` peers with its local PE (`10.20.10.9`) and a PE at a different site (`10.20.10.21`)[cite: 377].

### External & Internet Routing (eBGP)

Internet access is handled exclusively by the Border Routers. [cite_start]These routers peer with the external ISP and IXP routers from within dedicated VRFs (`ISP1`, `ISP2`, `IXP`)[cite: 34, 35, 772, 773]. [cite_start]For example, `DCBGP1` peers with `ISP1` (AS 52100) from within the `ISP1` VRF[cite: 784]. [cite_start]The external ISP routers advertise a default route into BGP, which can then be selectively redistributed into the client VRFs that require internet service[cite: 489, 766].

---

## 🛠️ Setup and Deployment

### Prerequisites

* **GNS3**: Version 2.2 or higher.
* **Cisco IOS Images**: The project uses Cisco IOU images for switches and various router images (e.g., `c7200`). You will need to import the appropriate images into your GNS3 environment. The provided configurations use Cisco IOS version 15.2.

### Instructions

1.  **Clone the Repository**:
    ```bash
    git clone [https://github.com/YOUR_USERNAME/YOUR_REPOSITORY_NAME.git](https://github.com/YOUR_USERNAME/YOUR_REPOSITORY_NAME.git)
    ```
2.  **Import the GNS3 Project**:
    * Open GNS3 and go to `File > Import Portable Project`.
    * Navigate to the cloned directory and select the `.gns3project` file.
    * GNS3 will prompt you to provide the necessary IOS images if they are not already installed.
3.  **Load Configurations**:
    * The startup configuration files (`.cfg`) for each device are included in this repository.
    * Ensure each device in the GNS3 topology is set to use its corresponding configuration file.
4.  **Start the Simulation**:
    * Once the project is loaded and all devices have their configurations, start all the nodes in the GNS3 topology.
    * Allow a few minutes for the network to converge (IS-IS adjacency, BGP peering, etc.).

---

## ✅ Verification and Testing

Once the network has converged, you can perform the following tests:

* **Client Reachability**: From a CE router (e.g., `CE-CLIENTE1-CALLE50`), ping the loopback address of another CE router belonging to the *same client* (e.g., `CE-CLIENTE1-VERAGUAS` at `10.107.0.4`). The ping should succeed.
* **Client Isolation**: Attempt to ping a loopback address of a CE router belonging to a *different client* (e.g., ping `10.108.0.9` from `CE-CLIENTE1-CALLE50`). The ping should fail.
* [cite_start]**Secure Management**: From the designated management station, use Telnet to access the various switches and routers via their management IP addresses in VLAN 500. Attempts from other IPs should be blocked by `access-class 20`[cite: 1120, 1121].
* **BGP Peering**:
    * On a PE router, inspect VPN routes with `show ip bgp vpnv4 all summary`.
    * On a CE router, verify its connection to the provider with `show ip bgp summary`.
* **Traceroute**: A `traceroute` between two CE devices of the same client will show the MPLS labels being used for transport across the provider core.

---

## 📁 Repository Contents

* **GNS3 Project File**: The main topology file for GNS3.
* **Device Configurations**: A directory containing the `startup-config.cfg` for every router and switch in the simulation.
* **README.md**: This file.
