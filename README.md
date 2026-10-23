# Arista EVPN-VXLAN Fabric Automation with Ansible

## Project Overview

This project demonstrates how to automate a five-switch
Arista EVPN-VXLAN fabric using Ansible, Python, YAML,
Jinja2, and Arista EOS eAPI.

The lab consists of:

- 2 Arista spine switches acting as BGP EVPN route reflectors
- 3 Arista leaf switches acting as VXLAN tunnel endpoints
- OSPF for underlay IP reachability
- MP-BGP EVPN for overlay control-plane signaling
- VXLAN for Layer 2 overlay encapsulation
- Ansible for automated device configuration

The objective is to demonstrate a reproducible network
automation workflow, from management connectivity and
inventory design through deployment and troubleshooting.

## Validation Status

| Component | Status |
|---|---|
| Device management | Verified |
| Ansible eAPI connectivity | Verified |
| Automated hostname configuration | Verified |
| Automated loopback configuration | Verified |
| Automated routed interfaces | Verified |
| OSPF underlay | Verified |
| MP-BGP EVPN sessions | Verified |
| EVPN Type 2 routes | Verified |
| EVPN Type 3 routes | Verified |
| VLAN-to-VNI mapping | Verified |
| VXLAN host-to-host forwarding | Not working |

**Important:** This project demonstrates a working
automation workflow and EVPN control plane. End-to-end
VXLAN forwarding remains unresolved.

## Fabric Architecture

| Device | Role | Management IP | Loopback0 |
|---|---|---|---|
| S-10 | Spine / EVPN RR | 192.168.159.210 | 10.10.10.10 |
| S-20 | Spine / EVPN RR | 192.168.159.220 | 20.20.20.20 |
| L-01 | Leaf / VTEP | 192.168.159.211 | 1.1.1.1 |
| L-02 | Leaf / VTEP | 192.168.159.212 | 2.2.2.2 |
| L-03 | Leaf / VTEP | 192.168.159.213 | 3.3.3.3 |

### Underlay

- Routing protocol: OSPF process 1
- OSPF area: 0.0.0.0
- Fabric links: Routed Ethernet interfaces using /31 addressing
- Loopback0: Advertised through OSPF

### Overlay

- BGP ASN: 65000
- BGP EVPN route reflectors: S-10 and S-20
- VXLAN UDP port: 4789
- VNI: 10102

| Leaf | Local VLAN | VNI |
|---|---|---|
| L-01 | 102 | 10102 |
| L-02 | 103 | 10102 |
| L-03 | 104 | 10102 |

Different local VLAN IDs map to the same Layer 2 VNI.

## Automation Technologies

- Ubuntu Linux
- Python virtual environment
- Ansible Core
- Arista EOS collection
- YAML inventories and playbooks
- Jinja2 variable substitution
- Arista eAPI over HTTPS (TCP 443)
- Git and GitHub for version control

# Arista EVPN-VXLAN Fabric Automation with Ansible

**A hands-on network automation portfolio project using GNS3, Arista EOS, Python, YAML, Jinja2, Ansible, OSPF, MP-BGP EVPN, and VXLAN.**

## 1. Project Overview

This project demonstrates how to build, automate, validate, and troubleshoot a five-switch Arista EVPN-VXLAN fabric.

The primary objective is to develop a repeatable network automation workflow that engineers can understand and execute without manually configuring every switch.

The lab uses:

- **Two Arista spine switches** operating as MP-BGP EVPN route reflectors.
- **Three Arista leaf switches** operating as VXLAN tunnel endpoints (VTEPs).
- **OSPF** for underlay routing and loopback reachability.
- **MP-BGP EVPN** for overlay control-plane signaling.
- **VXLAN** for Layer 2 overlay encapsulation.
- **Ansible** for automated configuration deployment.
- **YAML** for inventory and playbook definitions.
- **Jinja2 expressions** for dynamic device-specific configuration.
- **Arista eAPI** for programmatic switch management.
- **Git and GitHub** for version control and portfolio documentation.

The project covers the complete automation lifecycle:

1. Build the virtual network topology.
2. Establish management connectivity.
3. Configure Arista eAPI.
4. Install and prepare the Ansible environment.
5. Create an inventory and define device variables.
6. Validate inventory and playbook syntax.
7. Automate the network underlay.
8. Verify OSPF routing.
9. Automate the BGP EVPN overlay.
10. Configure VXLAN VTEPs and VLAN-to-VNI mappings.
11. Validate the control plane and test the data plane.
12. Troubleshoot network and automation failures.
13. Save configurations and publish the project to GitHub.

### Current Validation Status

| Component | Status |
|---|---|
| GNS3 five-switch topology | Verified |
| Management IP connectivity | Verified |
| Arista eAPI over HTTPS | Verified |
| Ansible inventory parsing | Verified |
| Automated hostname configuration | Verified |
| Automated Loopback0 configuration | Verified |
| Automated routed fabric interfaces | Verified |
| OSPF adjacencies | Verified |
| OSPF loopback reachability | Verified |
| BGP EVPN peer establishment | Verified |
| EVPN Type 2 MAC routes | Verified |
| EVPN Type 3 IMET routes | Verified |
| VLAN-to-VNI mappings | Verified |
| Local host MAC learning | Verified |
| Remote VXLAN MAC installation | Not verified; table observed empty |
| EVPN-driven VXLAN flood-list programming | Not working in observed tests |
| End-to-end host connectivity | Failed |

**Important:** This repository is an automation learning project with a validated underlay and EVPN control plane. End-to-end VXLAN data-plane forwarding is not yet operational. The outstanding behavior is documented in the troubleshooting section.

---

## 2. Network Architecture

### 2.1 Fabric Topology

```text
                       MP-BGP EVPN
                     Route Reflectors

                 +-------------------+
                 |       S-10        |
                 |    10.10.10.10    |
                 |     AS 65000      |
                 +-------------------+
                    /       |       \
                   /        |        \
                  /         |         \
             +------+   +------+   +------+
             | L-01 |   | L-02 |   | L-03 |
             |1.1.1.1|  |2.2.2.2|  |3.3.3.3|
             +------+   +------+   +------+
                  \         |         /
                   \        |        /
                    \       |       /
                 +-------------------+
                 |       S-20        |
                 |    20.20.20.20    |
                 |     AS 65000      |
                 +-------------------+

              OSPF Underlay: Area 0
              BGP EVPN Overlay: AS 65000
              VXLAN L2 VNI: 10102
```

Each leaf connects to both spines.

The spines provide underlay IP connectivity and act as BGP EVPN route reflectors. They do not operate as VTEPs in this lab.

The leaf switches terminate VXLAN tunnels and provide access to endpoint VLANs.

### 2.2 Device Addressing

| Device | Role | Management IP | Loopback0 |
|---|---|---|---|
| S-10 | Spine / EVPN RR | 192.168.159.210/24 | 10.10.10.10/32 |
| S-20 | Spine / EVPN RR | 192.168.159.220/24 | 20.20.20.20/32 |
| L-01 | Leaf / VTEP | 192.168.159.211/24 | 1.1.1.1/32 |
| L-02 | Leaf / VTEP | 192.168.159.212/24 | 2.2.2.2/32 |
| L-03 | Leaf / VTEP | 192.168.159.213/24 | 3.3.3.3/32 |

All addresses are laboratory examples and must be adapted to the user's own environment.

### 2.3 Physical Fabric Connections

| Spine | Interface | Leaf | Interface | /31 Network |
|---|---|---|---|---|
| S-10 | Ethernet1 | L-01 | Ethernet1 | 10.0.0.0/31 |
| S-10 | Ethernet2 | L-02 | Ethernet2 | 10.0.0.2/31 |
| S-10 | Ethernet3 | L-03 | Ethernet3 | 10.0.0.4/31 |
| S-20 | Ethernet10 | L-01 | Ethernet10 | 10.0.0.6/31 |
| S-20 | Ethernet11 | L-02 | Ethernet11 | 10.0.0.8/31 |
| S-20 | Ethernet12 | L-03 | Ethernet12 | 10.0.0.10/31 |

The /31 links are point-to-point Layer 3 connections.

For example:

```text
S-10 Ethernet1: 10.0.0.0/31
L-01 Ethernet1: 10.0.0.1/31
```

A /31 subnet provides two usable endpoint addresses for a point-to-point link.

### 2.4 Overlay VLAN and VNI Mapping

| Leaf | Local VLAN | Layer 2 VNI | VTEP |
|---|---|---|---|
| L-01 | 102 | 10102 | 1.1.1.1 |
| L-02 | 103 | 10102 | 2.2.2.2 |
| L-03 | 104 | 10102 | 3.3.3.3 |

The local VLAN IDs differ, but all three map to the same VNI.

This illustrates a key VXLAN design principle:

**The VLAN ID is locally significant. The VNI identifies the Layer 2 segment across the overlay.**

### 2.5 Test Endpoints

Two VPCS hosts were connected for forwarding tests.

| Host | Connected Device | Interface | IP |
|---|---|---|---|
| PC1 | L-01 | Ethernet5 | 192.168.100.11/24 |
| PC2 | L-03 | Ethernet5 | 192.168.100.33/24 |

Both hosts belong to the same IP subnet.

No default gateway is required for same-subnet communication.

The host-to-host test did not succeed in the current lab state.

---

## 3. Prerequisites

### 3.1 Software

This project was developed using:

- GNS3 2.2.61
- GNS3 VM running in VMware
- Arista vEOS-lab 4.33.2F
- Ubuntu Linux
- Python 3
- Ansible Core 2.21.5
- Arista EOS Ansible collection
- Git

Versions may differ in other environments. Commands and supported features should be validated against the installed software.

### 3.2 Required Skills

Readers should understand basic:

- IPv4 addressing and subnetting
- Ethernet switching and VLANs
- OSPF routing
- BGP neighbor establishment
- Linux terminal commands

Previous Ansible experience is not required.

---

## 4. Build the GNS3 Topology

### Step 1 — Prepare GNS3

Install GNS3 and configure the GNS3 VM.

Import a compatible Arista vEOS appliance and its required disk images.

Verify that the virtual switches can boot successfully.

### Step 2 — Add Network Devices

Create a new GNS3 project and add:

- Two spine switches
- Three leaf switches
- Two VPCS hosts
- Management connectivity to the Ubuntu automation controller

Connect each leaf to both spines using the interfaces shown in Section 2.3.

Connect PC1 to L-01 Ethernet5 and PC2 to L-03 Ethernet5.

### Step 3 — Build the Management Network

Connect the management interface of each Arista switch to a management network accessible from the Ubuntu controller.

The original lab used:

```text
Management subnet: 192.168.159.0/24

Ubuntu controller: 192.168.159.129
GNS3 management-side connectivity: 192.168.159.128
```

**Note:** The exact VMware and GNS3 network adapters depend on the local installation. Do not assume that the GNS3 VM's own management address and the emulated switch management subnet are identical.

Verify that Ubuntu and the switches share a reachable management network.

---

## 5. Configure Arista Management Access

This is the initial manual configuration required before Ansible can manage the switches.

### Step 1 — Configure the Management Interface

Example for L-01:

```eos
enable
configure terminal

hostname L-01

interface Management1
   ip address 192.168.159.211/24
   no shutdown

end
write memory
```

Repeat for the remaining switches using their assigned management IP addresses.

If the automation controller is on a different subnet, configure appropriate management routing.

### Step 2 — Configure a Management User

For an isolated laboratory environment, configure a local automation account.

Example:

```eos
configure terminal

username admin privilege 15 secret <LAB_PASSWORD>

end
write memory
```

Replace `<LAB_PASSWORD>` with a locally chosen password.

Do not use a production password or publish the actual password in GitHub.

### Step 3 — Enable Arista eAPI

Ansible uses Arista's eAPI to send configuration commands over HTTPS.

On every switch:

```eos
configure terminal

management api http-commands
   protocol https
   no shutdown

end
write memory
```

### Step 4 — Verify eAPI

On the switch:

```eos
show management api http-commands
```

Confirm that the HTTPS API is enabled.

From Ubuntu:

```bash
ping -c 4 192.168.159.211
```

Test the HTTPS port:

```bash
nc -vz 192.168.159.211 443
```

Expected result:

```text
Connection to 192.168.159.211 443 port [tcp/https] succeeded!
```

Repeat for all five switches.

### Understanding the Management Architecture

```text
Ubuntu Ansible Controller
192.168.159.129
        |
        | HTTPS / TCP 443
        |
Management Network
        |
        +---- S-10
        +---- S-20
        +---- L-01
        +---- L-02
        +---- L-03
```

Ansible does not require SSH to execute the current EOS configuration playbooks.

The inventory uses the `httpapi` connection plugin, which communicates with eAPI over HTTPS.

---

## 6. Install Python and Ansible

### Step 1 — Update Ubuntu

```bash
sudo apt update
```

Install the required packages:

```bash
sudo apt install -y \
  python3 \
  python3-venv \
  python3-pip \
  git \
  curl \
  netcat-openbsd
```

### Step 2 — Create the Project Directory

```bash
mkdir -p ~/network-automation/evpn-vxlan
cd ~/network-automation/evpn-vxlan
```

### Step 3 — Create a Python Virtual Environment

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

The shell should now display:

```text
(.venv)
```

### Why Use a Virtual Environment?

A Python virtual environment isolates the project's Python packages from the operating system.

This helps prevent dependency conflicts and makes the automation environment easier to reproduce.

**Important lesson from this lab:** Running Ansible outside the virtual environment resulted in a different Ansible version being used.

Always confirm the active environment before executing playbooks.

### Step 4 — Install Ansible

```bash
python -m pip install --upgrade pip
python -m pip install ansible-core
```

Verify:

```bash
ansible --version
```

Check the executable location:

```bash
which ansible
```

Expected location:

```text
/home/<username>/network-automation/evpn-vxlan/.venv/bin/ansible
```

### Step 5 — Install Required Collections

```bash
ansible-galaxy collection install arista.eos
```

Install the network common collection if needed:

```bash
ansible-galaxy collection install ansible.netcommon
```

Verify:

```bash
ansible-galaxy collection list
```

Look for:

```text
arista.eos
ansible.netcommon
```

### Step 6 — Verify Ansible Commands

```bash
ansible-playbook --version
ansible-inventory --version
```

If the system Ansible version appears instead of the virtual environment version, reactivate the environment.

```bash
source .venv/bin/activate
```

---

## 7. Project File Structure

```text
evpn-vxlan/
|
|-- .gitignore
|-- README.md
|-- inventory.yml
|
|-- hostname.yml
|-- loopback.yml
|-- underlay_interfaces.yml
|-- ospf.yml
|
|-- bgp_spines.yml
|-- bgp_leaves.yml
|-- vxlan.yml
|
`-- .venv/                 # Local only; excluded from Git
```

### File Responsibilities

| File | Purpose |
|---|---|
| `inventory.yml` | Defines devices, groups, addresses and configuration variables |
| `hostname.yml` | Configures hostnames |
| `loopback.yml` | Configures Loopback0 |
| `underlay_interfaces.yml` | Configures routed Ethernet interfaces |
| `ospf.yml` | Configures OSPF and enables IPv4 routing |
| `bgp_spines.yml` | Configures EVPN route reflectors |
| `bgp_leaves.yml` | Configures leaf EVPN neighbors |
| `vxlan.yml` | Configures VLANs, VXLAN interfaces and EVPN VLAN services |
| `.gitignore` | Prevents local environments and sensitive files from being committed |
| `README.md` | Documents the project and workflow |

---

## 8. Understanding the Ansible Inventory

The inventory is the central source of device-specific information.

Instead of creating a separate configuration for every switch, Ansible reads variables from the inventory and uses them to generate configuration commands.

### 8.1 Inventory Structure

The project uses:

```yaml
all:
  vars:
    bgp_asn: 65000

  children:
    spines:
      hosts:
        S-10:
          ansible_host: 192.168.159.210
          loopback_ip: 10.10.10.10

        S-20:
          ansible_host: 192.168.159.220
          loopback_ip: 20.20.20.20

    leaves:
      hosts:
        L-01:
          ansible_host: 192.168.159.211
          loopback_ip: 1.1.1.1
          access_vlan: 102
          l2_vni: 10102

        L-02:
          ansible_host: 192.168.159.212
          loopback_ip: 2.2.2.2
          access_vlan: 103
          l2_vni: 10102

        L-03:
          ansible_host: 192.168.159.213
          loopback_ip: 3.3.3.3
          access_vlan: 104
          l2_vni: 10102
```

This is a simplified illustration. The actual project inventory also contains `fabric_interfaces` and Ansible connection variables.

### 8.2 Fabric Interface Variables

Example for L-01:

```yaml
fabric_interfaces:
  - name: Ethernet1
    description: TO_S-10_Eth1
    ip: 10.0.0.1/31

  - name: Ethernet10
    description: TO_S-20_Eth10
    ip: 10.0.0.7/31
```

The variable `fabric_interfaces` contains a list of interface definitions.

Each entry has three attributes:

- `name`: Interface identifier.
- `description`: Interface description.
- `ip`: IPv4 address and prefix length.

Ansible loops through these entries and configures each interface.

### 8.3 Ansible Connection Variables

The lab uses the following shared settings:

```yaml
ansible_user: admin
ansible_connection: ansible.netcommon.httpapi
ansible_network_os: arista.eos.eos
ansible_httpapi_use_ssl: true
ansible_httpapi_validate_certs: false
ansible_httpapi_port: 443
ansible_become: true
ansible_become_method: enable
```

These values tell Ansible:

- Which account to use.
- Which transport plugin to use.
- Which network operating system is being managed.
- Which TCP port to connect to.
- Whether HTTPS certificate validation is enabled.
- Whether privileged execution is required.

Certificate validation is disabled for this isolated lab because self-signed certificates are used. Production environments should use trusted certificates and appropriate credential management.

### 8.4 Validate the Inventory

Run:

```bash
ansible-inventory -i inventory.yml --list
```

Inspect one device:

```bash
ansible-inventory -i inventory.yml --host L-01
```

Verify a specific variable:

```bash
ansible-inventory -i inventory.yml --host L-01 | grep loopback_ip
```

Expected:

```text
"loopback_ip": "1.1.1.1"
```

Verify the BGP ASN:

```bash
ansible-inventory -i inventory.yml --host S-10 | grep bgp_asn
```

Expected:

```text
"bgp_asn": 65000
```

Verify the VNI:

```bash
ansible-inventory -i inventory.yml --host L-01 | grep l2_vni
```

Expected:

```text
"l2_vni": 10102
```

### Common Inventory Mistake: Duplicate YAML Keys

During development, the inventory contained duplicate `vars` keys.

Ansible warned:

```text
Found duplicate mapping key 'vars'
Using last defined value only
```

This can cause earlier variables to disappear.

**Correct approach:** Keep a single `vars` mapping at the intended YAML level.

Use:

```bash
ansible-inventory -i inventory.yml --host S-10
```

to confirm that variables are actually available to the device.

---

## 9. Understanding YAML, Jinja2 and Ansible

These technologies have different responsibilities.

### YAML

YAML provides a human-readable structure for data and automation instructions.

Example:

```yaml
loopback_ip: 1.1.1.1
```

### Jinja2

Jinja2 substitutes variables into configuration commands.

Example:

```yaml
- "ip address {{ loopback_ip }}/32"
```

For L-01, Ansible renders:

```eos
ip address 1.1.1.1/32
```

For L-03, the same playbook renders:

```eos
ip address 3.3.3.3/32
```

The project uses inline Jinja2 expressions in YAML playbooks. It does not currently require separate `.j2` template files.

### Ansible

Ansible executes the tasks against the target devices.

Example:

```yaml
- name: Configure Loopback0
  arista.eos.eos_config:
    parents: interface Loopback0
    lines:
      - "ip address {{ loopback_ip }}/32"
```

The `arista.eos.eos_config` module handles configuration changes on EOS.

### Why This Architecture Matters

Instead of maintaining five separate loopback configuration scripts, we maintain:

- One inventory containing device-specific values.
- One playbook describing the desired configuration.
- One automation engine applying that configuration.

This reduces duplication and makes changes easier to audit.

---

## 10. Deploy the Fabric

Before deploying, confirm that the virtual environment is active and all five management addresses are reachable.

### 10.1 Check Management Connectivity

```bash
for ip in \
  192.168.159.210 \
  192.168.159.220 \
  192.168.159.211 \
  192.168.159.212 \
  192.168.159.213
do
  echo "Testing $ip"
  ping -c 2 "$ip"
done
```

Check eAPI:

```bash
nc -vz 192.168.159.211 443
```

Repeat for the other devices.

### 10.2 Validate All Playbooks

Run:

```bash
for file in \
  hostname.yml \
  loopback.yml \
  underlay_interfaces.yml \
  ospf.yml \
  bgp_spines.yml \
  bgp_leaves.yml \
  vxlan.yml
do
  ansible-playbook -i inventory.yml "$file" --syntax-check
done
```

Expected output for each file:

```text
playbook: <filename>.yml
```

**Important:** Syntax validation does not confirm device reachability or whether EOS accepts the configuration commands.

### 10.3 Deploy Hostnames

```bash
ansible-playbook -i inventory.yml hostname.yml -k
```

The `-k` option prompts for the connection password.

The playbook configures each device hostname from `inventory_hostname`.

Verify on EOS:

```eos
show hostname
```

Or inspect the CLI prompt.

### 10.4 Deploy Loopback Interfaces

```bash
ansible-playbook -i inventory.yml loopback.yml -k
```

The playbook uses:

```yaml
parents: interface Loopback0
lines:
  - "ip address {{ loopback_ip }}/32"
```

Verify:

```eos
show ip interface brief
```

Expected on L-01:

```text
Loopback0    1.1.1.1/32    up    up
```

### 10.5 Deploy Routed Fabric Interfaces

```bash
ansible-playbook -i inventory.yml underlay_interfaces.yml -k
```

The playbook iterates through `fabric_interfaces`.

Its configuration logic includes:

```yaml
parents: "interface {{ item.name }}"
lines:
  - "description {{ item.description }}"
  - "no switchport"
  - "ip address {{ item.ip }}"
  - "no shutdown"
```

The `no switchport` command converts the Ethernet interface into a routed interface.

Verify:

```eos
show ip interface brief
show interfaces status
show lldp neighbors
```

Expected on L-01:

```text
Ethernet1     10.0.0.1/31    up    up
Ethernet10    10.0.0.7/31    up    up
```

Test directly connected neighbors:

```eos
ping 10.0.0.0
ping 10.0.0.6
```

### 10.6 Deploy OSPF

```bash
ansible-playbook -i inventory.yml ospf.yml -k
```

The OSPF playbook configures:

- OSPF process 1.
- Router ID from Loopback0.
- Passive interfaces by default.
- Explicitly non-passive fabric interfaces.
- OSPF area 0.
- Point-to-point OSPF network type.
- Loopback0 advertisement.
- Global IPv4 routing.

Representative configuration:

```eos
ip routing

router ospf 1
   router-id 1.1.1.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet10

interface Ethernet1
   ip ospf area 0.0.0.0
   ip ospf network point-to-point

interface Ethernet10
   ip ospf area 0.0.0.0
   ip ospf network point-to-point

interface Loopback0
   ip ospf area 0.0.0.0
```

The example above represents L-01.

### Verify OSPF

```eos
show ip ospf neighbor
```

Expected: Each leaf forms OSPF adjacencies with its connected spines.

Verify routing:

```eos
show ip route ospf
```

Verify remote loopback reachability:

```eos
ping 10.10.10.10 source 1.1.1.1
ping 20.20.20.20 source 1.1.1.1
```

**Do not deploy BGP EVPN until the OSPF underlay is functioning.**

### 10.7 Deploy BGP EVPN Route Reflectors

```bash
ansible-playbook -i inventory.yml bgp_spines.yml -k
```

This playbook targets:

```yaml
hosts: spines
```

It configures BGP AS 65000, the `EVPN-CLIENTS` peer group, and leaf loopback neighbors.

The route reflectors use the following leaf loopbacks:

```text
1.1.1.1
2.2.2.2
3.3.3.3
```

Representative spine configuration:

```eos
router bgp 65000
   router-id 10.10.10.10
   no bgp default ipv4-unicast

   neighbor EVPN-CLIENTS peer group
   neighbor EVPN-CLIENTS remote-as 65000
   neighbor EVPN-CLIENTS update-source Loopback0

   neighbor 1.1.1.1 peer group EVPN-CLIENTS
   neighbor 2.2.2.2 peer group EVPN-CLIENTS
   neighbor 3.3.3.3 peer group EVPN-CLIENTS

   address-family evpn
      neighbor EVPN-CLIENTS activate
      neighbor EVPN-CLIENTS route-reflector-client
```

S-20 uses its own router ID.

### Why Route Reflectors?

Without route reflectors, iBGP requires a full mesh of BGP peerings for route propagation.

The two spines act as EVPN route reflectors, allowing the leaf switches to exchange EVPN routes without forming a full mesh of direct leaf-to-leaf BGP sessions.

The route reflectors distribute EVPN reachability information; they do not encapsulate endpoint traffic into VXLAN in this design.

### 10.8 Deploy BGP EVPN Leaf Configuration

```bash
ansible-playbook -i inventory.yml bgp_leaves.yml -k
```

This playbook targets:

```yaml
hosts: leaves
```

It configures the `EVPN-RR` peer group and establishes BGP sessions to:

```text
10.10.10.10
20.20.20.20
```

Representative leaf configuration:

```eos
router bgp 65000
   router-id 1.1.1.1
   no bgp default ipv4-unicast

   neighbor EVPN-RR peer group
   neighbor EVPN-RR remote-as 65000
   neighbor EVPN-RR update-source Loopback0

   neighbor 10.10.10.10 peer group EVPN-RR
   neighbor 20.20.20.20 peer group EVPN-RR

   address-family evpn
      neighbor EVPN-RR activate
```

### Verify BGP EVPN

```eos
show bgp evpn summary
```

Expected on a leaf:

```text
Neighbor       AS       State
10.10.10.10    65000    Estab
20.20.20.20    65000    Estab
```

Expected on a spine: Established EVPN sessions with all three leaves.

### 10.9 Deploy VXLAN

```bash
ansible-playbook -i inventory.yml vxlan.yml -k
```

The playbook performs three major configuration operations.

**Create the local VLAN:**

```yaml
- "vlan {{ access_vlan }}"
```

**Configure Vxlan1:**

```yaml
parents: interface Vxlan1
lines:
  - "vxlan source-interface Loopback0"
  - "vxlan udp-port 4789"
  - "vxlan vlan {{ access_vlan }} vni {{ l2_vni }}"
```

**Configure the BGP EVPN VLAN service:**

```yaml
parents:
  - "router bgp {{ bgp_asn }}"
  - "vlan {{ access_vlan }}"
lines:
  - "rd {{ loopback_ip }}:{{ access_vlan }}"
  - "route-target both {{ l2_vni }}:{{ l2_vni }}"
  - "redistribute learned"
```

### Understanding the VXLAN Configuration

**VXLAN source interface**

```eos
vxlan source-interface Loopback0
```

Uses Loopback0 as the local VTEP address.

**VLAN-to-VNI mapping**

```eos
vxlan vlan 102 vni 10102
```

Maps local VLAN 102 to overlay VNI 10102.

**Route distinguisher**

```eos
rd 1.1.1.1:102
```

Uniquely identifies the EVPN routing information associated with the local service.

**Route target**

```eos
route-target both 10102:10102
```

Controls EVPN route import and export membership.

The RDs are different between leaves, while the route targets match.

**MAC advertisement**

```eos
redistribute learned
```

Allows locally learned MAC addresses to be advertised through BGP EVPN.

### 10.10 Save Configurations

The playbooks include:

```yaml
- name: Save configuration
  arista.eos.eos_command:
    commands:
      - write memory
```

This saves the running configuration to startup configuration.

The configuration tasks use `save_when: never`, while a separate command handles persistence.

This separation makes the configuration and save operations explicit.

**Note:** A successful `write memory` confirms that the device accepted the save operation; persistent storage should still be tested by restarting a lab device when practical.

---

## 11. Network Verification

A successful Ansible run does not automatically mean that the network is functioning correctly.

Verification must be performed at multiple layers.

### 11.1 Physical and Interface Layer

```eos
show interfaces status
show ip interface brief
show lldp neighbors
```

Confirm:

- Expected interfaces are connected.
- Fabric interfaces are routed.
- IP addresses are correct.
- LLDP neighbors match the topology.

### 11.2 Underlay Routing

```eos
show ip ospf neighbor
show ip route ospf
show ip route
```

Confirm:

- OSPF adjacencies reach FULL.
- Remote loopback routes are installed.
- VTEP addresses are reachable through the underlay.

### 11.3 BGP EVPN

```eos
show bgp evpn summary
```

Confirm the expected BGP sessions are Established.

### 11.4 VXLAN Mapping

```eos
show vxlan vni
```

Example observed on L-02:

```text
VNI     VLAN    Source    Interface
10102   103     static    Vxlan1
```

This verifies the local VLAN-to-VNI mapping.

### 11.5 EVPN Type 3 Routes

```eos
show bgp evpn route-type imet
```

Type 3 routes advertise participation in an EVPN broadcast domain and provide information used for BUM replication.

In this lab, the Type 3 routes showed:

```text
VNI: 10102
PMSI Tunnel: Ingress Replication
```

This indicates ingress replication as the advertised BUM transport mechanism.

### 11.6 EVPN Type 2 Routes

```eos
show bgp evpn route-type mac-ip
```

Type 2 routes advertise endpoint MAC addresses and optionally IP bindings.

The lab observed:

```text
PC1 MAC: 0050.7966.6800
PC2 MAC: 0050.7966.6801
```

L-01 received PC2's MAC through the EVPN control plane.

### 11.7 Detailed EVPN Routes

```eos
show bgp evpn route-type mac-ip detail
show bgp evpn route-type imet detail
```

Inspect:

- Route distinguisher
- Route target
- VNI
- BGP next hop
- VXLAN encapsulation extended community
- Route reflector attributes
- PMSI tunnel information for Type 3 routes

### 11.8 VXLAN Forwarding State

```eos
show vxlan flood vtep
show vxlan address-table
show vxlan config-sanity
```

These commands help determine whether received EVPN information has been installed into the VXLAN forwarding state.

In the current lab, the flood VTEP and remote VXLAN MAC tables were empty.

This remains an unresolved data-plane issue.

---

## 12. Test Host Connectivity

The following manual access-port configuration was used for testing.

### L-01

```eos
configure terminal

vlan 102

interface Ethernet5
   switchport
   switchport mode access
   switchport access vlan 102
   no shutdown

end
write memory
```

### L-03

```eos
configure terminal

vlan 104

interface Ethernet5
   switchport
   switchport mode access
   switchport access vlan 104
   no shutdown

end
write memory
```

### Configure PC1

On the VPCS console:

```text
ip 192.168.100.11/24
```

### Configure PC2

```text
ip 192.168.100.33/24
```

### Test

From PC1:

```text
ping 192.168.100.33
```

Inspect ARP:

```text
show arp
```

### Observed Result

The ping failed and PC1's ARP table remained empty.

However, L-01 learned PC1's MAC locally:

```text
VLAN 102
MAC 0050.7966.6800
Port Ethernet5
Type DYNAMIC
```

BGP EVPN also received PC2's remote MAC route.

This demonstrates that local MAC learning and EVPN control-plane signaling were working, but end-to-end VXLAN forwarding was not.

---

## 13. Network Troubleshooting Guide

The most important troubleshooting principle is to identify the layer where the failure occurs.

### 13.1 Troubleshooting Workflow

```text
Host connectivity fails
         |
         v
Are local interfaces up?
         |
         +-- NO --> Check GNS3 links and interface state
         |
        YES
         |
         v
Is underlay routing working?
         |
         +-- NO --> Check IP addressing, OSPF and routing
         |
        YES
         |
         v
Are BGP EVPN sessions established?
         |
         +-- NO --> Check loopback reachability and BGP
         |
        YES
         |
         v
Are Type 2 and Type 3 routes received?
         |
         +-- NO --> Check EVPN AF, RTs and advertisements
         |
        YES
         |
         v
Are remote VTEPs and MACs installed?
         |
         +-- NO --> Investigate EVPN import and VXLAN state
         |
        YES
         |
         v
Check access VLANs, ARP, encapsulation and packet forwarding
```

### 13.2 OSPF Neighbors Are Missing

**Symptoms**

```eos
show ip ospf neighbor
```

Returns no neighbors.

**Check interface addressing:**

```eos
show ip interface brief
```

**Check OSPF configuration:**

```eos
show running-config section router ospf
show running-config interfaces Ethernet1
```

**Check IPv4 routing:**

```eos
show running-config | include ip routing
```

During development, `ip routing` was not initially enabled.

This was corrected in the OSPF automation.

**Other possible causes**

- Incorrect /31 addressing
- Interface shutdown
- OSPF area mismatch
- Passive interface configuration
- MTU mismatch
- Missing physical connectivity

### 13.3 OSPF Area 0 Versus 0.0.0.0

These represent the same OSPF area.

```text
Area 0 = Area 0.0.0.0
```

EOS may display the area in dotted-decimal format.

This is normal.

### 13.4 BGP EVPN Sessions Are Not Established

Run:

```eos
show bgp evpn summary
```

If a session is not Established, check:

1. Underlay reachability to the neighbor loopback.
2. Correct BGP ASN.
3. Correct neighbor IP.
4. Loopback update source.
5. EVPN address-family activation.
6. Whether the neighbor is assigned to the correct peer group.

Example:

```eos
ping 10.10.10.10 source 1.1.1.1
```

If the ping fails, troubleshoot the underlay before BGP.

### 13.5 EVPN Routes Are Missing

Check:

```eos
show bgp evpn route-type mac-ip
show bgp evpn route-type imet
```

Verify:

- EVPN address family is enabled.
- BGP peers are Established.
- Local VLAN service exists.
- Route targets are correct.
- MAC redistribution is configured.
- VXLAN interface and VNI mapping exist.

### 13.6 Type 2 Routes Exist but VXLAN Forwarding Fails

This issue was observed in the lab.

L-01 learned the remote MAC through BGP EVPN:

```text
Remote MAC: 0050.7966.6801
Next hop: 3.3.3.3
VNI: 10102
```

However:

```eos
show vxlan address-table
```

Returned:

```text
Total Remote Mac Addresses: 0
```

This means receiving a BGP EVPN route did not result in an observable remote VXLAN MAC forwarding entry.

**Investigation steps**

1. Verify the remote route's BGP next hop.
2. Verify underlay reachability to the remote VTEP.
3. Inspect the route target and VXLAN encapsulation attributes.
4. Verify the local EVPN instance.
5. Verify the VLAN-to-VNI mapping.
6. Check VXLAN forwarding tables.
7. Check EOS logs and platform-specific forwarding limitations.
8. If needed, capture traffic to determine whether VXLAN packets are transmitted or received.

The precise root cause has not yet been established.

### 13.7 Type 3 Routes Exist but Flood List Is Empty

The lab received Type 3 IMET routes but reported:

```text
No flood list configured
No remote VTEP in VLAN 102
```

An IMET route signals participation in an EVPN broadcast domain.

It does not, by itself, prove successful forwarding-plane programming.

Check:

```eos
show bgp evpn route-type imet detail
show vxlan flood vtep
show vxlan config-sanity
```

Verify that the routes contain the expected VNI, route target and replication attributes.

Avoid immediately configuring static flood lists because doing so may hide the underlying EVPN control-plane integration issue.

### 13.8 ARP Fails Between VXLAN Hosts

When PC1 pings PC2, it first needs PC2's MAC address.

Without an existing ARP entry, PC1 sends an ARP broadcast.

In a functioning EVPN-VXLAN fabric, the broadcast can be replicated to remote VTEPs participating in the same Layer 2 VNI, unless an applicable suppression mechanism handles the request.

Check PC1:

```text
show arp
```

Check L-01:

```eos
show mac address-table vlan 102
```

Check EVPN:

```eos
show bgp evpn route-type mac-ip
show bgp evpn route-type imet
```

Check VXLAN:

```eos
show vxlan flood vtep
show vxlan address-table
```

**Important:** The switch's Layer 3 ARP table is not the same as the endpoint's ARP table.

A Layer 2 VTEP does not necessarily have an ARP entry for every bridged host.

---

## 14. Ansible Troubleshooting Guide

### 14.1 Inventory Parsing Failure

**Symptom**

```text
Unable to parse inventory.yml as an inventory source
No inventory was parsed
No hosts matched
```

**Troubleshooting**

```bash
ansible-inventory -i inventory.yml --list -vvv
```

Check:

- YAML indentation
- Duplicate keys
- Correct file path
- Correct host grouping
- Whether the intended virtual environment is active

Validate one host:

```bash
ansible-inventory -i inventory.yml --host L-01
```

### 14.2 Wrong Ansible Version

**Symptom**

The same inventory behaves differently between terminal sessions.

**Possible cause**

Ansible is running from the operating system instead of the project virtual environment.

**Verify**

```bash
which ansible
ansible --version
```

**Correct**

```bash
cd ~/network-automation/evpn-vxlan
source .venv/bin/activate
```

Then rerun the command.

### 14.3 No Route to Host

**Symptom**

```text
[Errno 113] No route to host
```

This is a network reachability problem, not a YAML syntax problem.

**Check**

```bash
ip addr
ip route
ping -c 4 192.168.159.212
```

Possible causes:

- GNS3 VM restarted.
- Management adapter changed.
- Switch management IP was lost.
- Switch or management network is unavailable.
- GNS3 links or node state changed.

**Corrective action**

Restore management connectivity before rerunning the playbook.

### 14.4 Connection Refused

**Symptom**

```text
[Errno 111] Connection refused
```

The device may be reachable at Layer 3 while the required TCP service is unavailable.

**Verify**

```bash
ping -c 4 192.168.159.212
nc -vz 192.168.159.212 443
```

On EOS:

```eos
show management api http-commands
```

**Possible correction**

```eos
configure terminal
management api http-commands
   protocol https
   no shutdown
end
write memory
```

### 14.5 Ansible Playbook Runs but Nothing Changes

Possible causes include:

- Inventory variables are missing.
- The playbook targets the wrong host group.
- Configuration already matches the desired state.
- Tasks are skipped.
- The wrong inventory file is used.

Run:

```bash
ansible-playbook -i inventory.yml hostname.yml -k -v
```

Check the playbook's `hosts` field.

Example:

```yaml
hosts: leaves
```

Only targets the leaf group.

### 14.6 Understanding `ok`, `changed`, and `failed`

Example:

```text
L-01 : ok=4 changed=3 unreachable=0 failed=0
```

Interpretation:

- `ok`: Tasks completed successfully.
- `changed`: Tasks reported configuration changes.
- `unreachable`: Ansible could not connect.
- `failed`: Tasks encountered errors.

A task returning `changed=0` may be expected when the configuration already exists.

### 14.7 Idempotency

Idempotency means repeatedly applying the same intended configuration should not create unnecessary changes.

Test a playbook twice:

```bash
ansible-playbook -i inventory.yml hostname.yml -k
```

Then repeat:

```bash
ansible-playbook -i inventory.yml hostname.yml -k
```

Ideally, the second run reports no configuration changes.

**Note:** Idempotency depends on the module, command formatting and EOS configuration representation. The current playbooks may still produce warnings or report changes that require refinement.

### 14.8 Configuration Diff Warnings

The lab produced warnings similar to:

```text
To ensure idempotency and correct diff
the input configuration lines should be similar
to how they appear in the running configuration
```

This warning does not automatically mean the task failed.

It indicates that Ansible may have difficulty comparing the supplied configuration lines with the format EOS uses in its running configuration.

Inspect the actual configuration:

```eos
show running-config
```

Use configuration formatting consistent with EOS output where possible.

### 14.9 Syntax Check Versus Execution

Syntax check:

```bash
ansible-playbook -i inventory.yml ospf.yml --syntax-check
```

This verifies the playbook's structure.

Execution:

```bash
ansible-playbook -i inventory.yml ospf.yml -k
```

This connects to devices and applies configuration.

A successful syntax check does not guarantee:

- Device reachability
- Valid credentials
- Supported EOS commands
- Correct network behavior

### 14.10 Safe Deployment Practices

Before deploying changes:

1. Validate inventory.
2. Run syntax checks.
3. Confirm management reachability.
4. Review the target device group.
5. Back up configurations.
6. Consider `--check` and `--diff` where supported.
7. Deploy to a limited subset when practical.
8. Verify device state.
9. Save configurations.
10. Document results.

Example limited deployment:

```bash
ansible-playbook -i inventory.yml hostname.yml -k --limit L-01
```

This targets only L-01.

---

## 15. Automation Verification Checklist

Use this checklist after a fresh deployment.

### Management

- [ ] All switches respond to ping.
- [ ] HTTPS TCP 443 is reachable.
- [ ] eAPI is enabled.
- [ ] Inventory resolves all devices.

### Underlay

- [ ] Hostnames are correct.
- [ ] Loopback0 addresses are correct.
- [ ] Fabric interfaces are routed and up.
- [ ] OSPF neighbors are FULL.
- [ ] Remote loopbacks are reachable.

### Overlay

- [ ] BGP EVPN sessions are Established.
- [ ] Route reflectors have the expected leaf neighbors.
- [ ] Leaf EVPN peerings are established.
- [ ] Local VLANs exist.
- [ ] VLAN-to-VNI mappings are correct.
- [ ] Type 3 IMET routes are present.
- [ ] Type 2 MAC routes are advertised and received.

### Data Plane

- [ ] Remote VTEPs are installed in the VXLAN forwarding state.
- [ ] Remote MAC addresses appear in the VXLAN MAC table.
- [ ] Host ARP resolution succeeds.
- [ ] Host-to-host ICMP succeeds.
- [ ] VXLAN encapsulated traffic is verified where packet capture is available.

**Current lab status:** The final five data-plane checks remain unresolved or unsuccessful.

---

## 16. Understanding EVPN Route Types

### Type 2 — MAC/IP Advertisement

Purpose: Advertise endpoint reachability.

Example:

```text
MAC: 0050.7966.6801
VTEP: 3.3.3.3
VNI: 10102
```

This tells remote VTEPs where a particular endpoint MAC can be reached.

### Type 3 — Inclusive Multicast Ethernet Tag (IMET)

Purpose: Advertise membership and replication information for an EVPN broadcast domain.

In this lab, Type 3 routes advertise ingress replication.

Type 3 does not disappear when Type 2 information becomes available.

The routes perform different functions:

- Type 2 supports endpoint reachability.
- Type 3 supports BUM replication membership.

### Type 5 — IP Prefix Route

Purpose: Advertise routed IP prefix reachability through EVPN.

Type 5 is relevant to Layer 3 EVPN services.

**This lab does not implement an L3 VNI or Type 5-based routed overlay.**

---

## 17. Security Considerations

This project is intended for isolated laboratory use.

Before adapting it to production:

- Replace shared lab credentials with dedicated automation accounts.
- Use Ansible Vault or another approved secrets-management system.
- Validate HTTPS certificates.
- Apply least-privilege access.
- Restrict management-plane connectivity.
- Back up configurations before changes.
- Review every playbook before deployment.
- Avoid committing credentials, private keys or environment secrets.

### Recommended `.gitignore`

```gitignore
# Python environments
.venv/
venv/

# Python cache
__pycache__/
*.pyc

# Ansible temporary files
*.retry
.ansible/

# Environment and secrets
.env
.env.*
!.env.example
vault.yml
vault.yaml
*secret*
*password*
*.pem
*.key

# Editor and OS files
.vscode/
.idea/
.DS_Store
```

**Warning:** `.gitignore` does not remove secrets already committed to Git history. Review files before the first commit.

---

## 18. Git and GitHub Workflow

This section explains how to publish the project.

### Step 1 — Initialize Git

From the project directory:

```bash
cd ~/network-automation/evpn-vxlan
git init
```

### Step 2 — Set the Main Branch

```bash
git branch -M main
```

### Step 3 — Configure Git Identity

```bash
git config --global user.name "Your GitHub Name"
git config --global user.email "your-email@example.com"
```

Replace the example values with your own.

### Step 4 — Verify Ignored Files

```bash
git status --short
```

Check whether `.venv` is ignored:

```bash
git check-ignore .venv/
```

Expected:

```text
.venv/
```

### Step 5 — Review Files Before Committing

```bash
git status
```

Inspect the repository for credentials and other sensitive values.

Remember that management IP addresses and lab topology details are intentionally documented, but passwords, tokens and private keys should not be published.

### Step 6 — Stage Files

After reviewing the files:

```bash
git add README.md .gitignore *.yml
```

Review staged changes:

```bash
git diff --cached --stat
git diff --cached
```

### Step 7 — Commit

```bash
git commit -m "Initial Arista EVPN-VXLAN Ansible automation lab"
```

### Step 8 — Create a GitHub Repository

On GitHub:

1. Sign in to your account.
2. Select **New repository**.
3. Name it `arista-evpn-vxlan-ansible`.
4. Add a short project description.
5. Select Public if it is intended as a portfolio project.
6. Do not initialize the remote repository with a README, license or `.gitignore` if those files already exist locally.
7. Create the repository.

### Step 9 — Connect the Local Repository

Copy the repository URL from GitHub.

Example:

```bash
git remote add origin https://github.com/YOUR_USERNAME/arista-evpn-vxlan-ansible.git
```

Verify:

```bash
git remote -v
```

### Step 10 — Push to GitHub

```bash
git push -u origin main
```

GitHub requires an appropriate authentication method, such as an authenticated credential helper, GitHub CLI or personal access token for HTTPS operations. An account password alone is not accepted for Git operations over HTTPS.

### Step 11 — Verify Publication

Open the repository in your browser.

Confirm that:

- README renders correctly.
- All eight playbooks are present.
- Inventory is present.
- `.gitignore` is present.
- `.venv` was not uploaded.
- No credentials were committed.

### Future Updates

After modifying files:

```bash
git status
git add .
git commit -m "Describe the changes"
git push
```

Only use `git add .` after reviewing the repository and confirming that ignored and sensitive files are handled correctly.

---

## 19. Reproducing the Automation Workflow

Once the lab topology, management connectivity and eAPI access are established, a reader can reproduce the deployment using the following sequence.

### Activate the Environment

```bash
cd ~/network-automation/evpn-vxlan
source .venv/bin/activate
```

### Validate Inventory

```bash
ansible-inventory -i inventory.yml --list
```

### Deploy Hostnames

```bash
ansible-playbook -i inventory.yml hostname.yml -k
```

### Deploy Loopbacks

```bash
ansible-playbook -i inventory.yml loopback.yml -k
```

### Deploy Underlay Interfaces

```bash
ansible-playbook -i inventory.yml underlay_interfaces.yml -k
```

### Deploy OSPF

```bash
ansible-playbook -i inventory.yml ospf.yml -k
```

**Stop and verify OSPF neighbors and loopback reachability before proceeding.**

### Deploy Spine EVPN Configuration

```bash
ansible-playbook -i inventory.yml bgp_spines.yml -k
```

### Deploy Leaf EVPN Configuration

```bash
ansible-playbook -i inventory.yml bgp_leaves.yml -k
```

**Stop and verify BGP EVPN sessions before proceeding.**

### Deploy VXLAN

```bash
ansible-playbook -i inventory.yml vxlan.yml -k
```

### Verify the Overlay

On EOS:

```eos
show bgp evpn summary
show bgp evpn route-type imet
show bgp evpn route-type mac-ip
show vxlan vni
show vxlan flood vtep
show vxlan address-table
show vxlan config-sanity
```

### Test Host Connectivity

From PC1:

```text
ping 192.168.100.33
show arp
```

If forwarding fails, follow the troubleshooting workflow in Section 13.

---

## 20. Known Limitations

### VXLAN Data-Plane Forwarding

The lab currently demonstrates successful EVPN control-plane route exchange but unsuccessful host-to-host VXLAN forwarding.

Observed behavior:

- Type 2 routes are present.
- Type 3 routes are present.
- Route targets match.
- VNI mappings are correct.
- Local MAC addresses are learned.
- Remote VXLAN MAC entries are not installed as expected.
- VXLAN flood-list entries are absent.
- Host-to-host ping fails.

The exact root cause remains undetermined.

Possible areas for further investigation include:

- Remote EVPN route attributes and import behavior.
- VXLAN forwarding-state programming.
- Virtual EOS platform behavior.
- Data-plane feature support and limitations.
- VXLAN packet transmission and reception.
- Ingress replication state.

These are investigation areas, not confirmed causes.

### No Layer 3 VNI

The current lab implements a Layer 2 VNI only.

It does not configure:

- Tenant VRFs
- L3 VNIs
- Distributed anycast gateways
- Inter-subnet VXLAN routing
- EVPN Type 5-based prefix routing

### No CI/CD Pipeline

The repository currently contains executable Ansible playbooks, but no automated GitHub Actions validation pipeline.

### No Automated Rollback

The playbooks save configurations but do not implement automatic rollback following failed deployments.

### Limited Automated Testing

Most network verification is currently performed manually using EOS show commands.

---

## 21. Future Improvements

### Phase 1 — Automation Quality

- Improve playbook idempotency.
- Introduce dedicated `group_vars` and `host_vars`.
- Replace repeated configurations with reusable roles.
- Introduce Jinja2 templates where appropriate.
- Add automated configuration backups.
- Add `requirements.yml` for collection dependencies.
- Pin Python dependencies for reproducibility.

### Phase 2 — Automated Verification

- Collect OSPF neighbor state.
- Validate BGP EVPN sessions.
- Check route counts and expected neighbors.
- Validate VLAN-to-VNI mappings.
- Assert remote VTEP reachability.
- Generate automated pass/fail reports.

### Phase 3 — VXLAN Data Plane

- Resolve remote MAC and flood-list programming.
- Verify ingress replication.
- Capture VXLAN traffic.
- Test host-to-host forwarding.
- Add automated endpoint connectivity tests.

### Phase 4 — Advanced EVPN

- Introduce VRFs.
- Configure Layer 3 VNIs.
- Add distributed anycast gateways.
- Implement inter-subnet routing.
- Study EVPN Type 5 advertisements.

### Phase 5 — CI/CD and Portfolio Expansion

- Add GitHub Actions syntax validation.
- Add YAML linting.
- Add Ansible linting.
- Introduce pre-deployment checks.
- Add automated change validation.
- Implement deployment reports.
- Document rollback procedures.

---

## 22. Key Engineering Lessons

This project demonstrates several important principles.

**1. Network automation depends on network fundamentals.**

Ansible cannot compensate for broken management connectivity, incorrect addressing or missing routing.

**2. Inventory design matters.**

A well-structured inventory separates device-specific data from reusable automation logic.

**3. YAML, Jinja2 and Ansible have different responsibilities.**

YAML structures the information, Jinja2 renders variable values, and Ansible executes tasks.

**4. Syntax validation is not network validation.**

A syntactically correct playbook can still fail during deployment or produce an incorrect network configuration.

**5. BGP EVPN control-plane success does not guarantee VXLAN data-plane success.**

EVPN routes may be present while the expected forwarding entries are missing.

**6. Configuration persistence must be considered.**

Saving the running configuration is part of the deployment workflow.

**7. Troubleshooting should follow the architecture.**

Verify management connectivity, interfaces, underlay routing, overlay control plane and data plane in that order.

**8. Honest validation is part of engineering quality.**

Documenting known limitations is more valuable than presenting an unverified implementation as fully functional.

---

## 23. Conclusion

This repository demonstrates a practical network automation workflow for an Arista EVPN-VXLAN fabric.

The project progresses from manual management bootstrap to inventory-driven configuration deployment using Ansible.

It successfully demonstrates automated switch configuration, OSPF underlay establishment, MP-BGP EVPN peer establishment, EVPN Type 2 and Type 3 route exchange, and VLAN-to-VNI configuration.

End-to-end VXLAN forwarding remains an open troubleshooting item.

The long-term goal is to evolve this lab into a more complete, validated, and reusable network automation framework incorporating automated testing, configuration management, CI/CD, and advanced EVPN services.

**Project focus:** Network engineering, infrastructure automation, reproducibility, validation, and systematic troubleshooting.

