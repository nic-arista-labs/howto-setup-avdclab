# Arista AVD Workflow

## Table of Contents
- [AVD Workflow Overview](workflow-overview)
- [Inventory Structure](#inventory-structure)
- [Role of Variables](#role-of-variables)
- [Build Playbook](#build-playbook)

### AVD Workflow Overview


This document outlines the workflow for using Arista Ansible Validated Designs (AVD) to automate and deploy network configurations to EOS devices via CloudVision as-a-Service (CVaaS).

### Inventory Structure

Below is a simple basic Ansible file structure breakdown.

```bash
project_rooot/
├── inventory.yml              # Main inventory file
├── group_vars/                # Global Ansible groups directory
├── └── all.yml                # Glodal Ansible variables yaml file
├── └── <group>.yml            # Group variables yaml file
│   └── custom-head.html       # Custom header/footer
├── host_vars/                 # Global Ansible hosts directory
│   └── <device-hosname>.yml   # host specific variables
├── build.yml                  # playbook to render configuration
├── deploy.yml                 # playbook to push configuration to CVaaS/CVP

```
Here is an example of the Ansible inventory file that can use used to define your AVD topology.
This YAML file example defines the topology (fabric) and host relationships.

```yaml
---
### AVD Topology inventory.yml
all:
  children:
    FABRIC:
      children:
        SPINES:
          hosts:
            spine1:
            spine2:
        LEAFS:
          hosts:
            leaf1a:
            leaf2b:
    NETWORK_SERVIVES:
      children:
        LEAFS:
        SPINES:
    NETWORK_PORTS:
      children:
        LEAFS:
        SPINES:
```

### Role of Variables

<span style="background-color:rgb(180, 180, 180);padding: 0.2em 0.4em;font-weight: bold">group_vars/all.yml:</span> Global AVD variables shared by all devices.
This is a great place to store the required Arista eAPI and SSH connection parameters. Ansible can reference these parameters for all devices in the inventory file.

AVD uses these connection parameters to make direct connections to the devices to perforem some unique task such as update documentation.

Ansible can used to query the network to help gather information.

```yaml
ansible_user: admin
ansible_ssh_pass: "{{ vault_ansible_password }}"
ansible_network_os: eos
ansible_connection: network_cli
ansible_become: yes
ansible_become_method: enable
ansible_become_password: admin
```

<span style="background-color:rgb(180, 180, 180);padding: 0.2em 0.4em;font-weight: bold">group_vars/FABRIC.yml:</span> Global AVD Configuration variables applied to all device under the tree heiarchy.

These configurations will outline the which devices are group and have mLAG applied.
Also, the uplinks and interface designations to the spine are referenced from this .yml structure file.

```yaml
# CloudVision TerminAtter definitions
cvp_instance_ips:
  - apiserver.arista.io
terminattr_smashexcludes: "ale,flexCounter,hardware,kni,pulse,strata"
terminattr_ingestexclude: "/Sysdb/cell/1/agent,/Sysdb/cell/2/agent"
terminattr_disable_aaa: true
terminattr_cvaddr: "apiserver.arista.io:443"
terminattr_cvauth: "token-secure,/tmp/cv-onboarding-token"
terminattr_cvvrf: MGMT
terminattr_taillogs: true

# Spine Switches
l3spine:
  defaults:
    platform: cEOSLab
    spanning_tree_mode: mstp
    spanning_tree_priority: 4096
    loopback_ipv4_pool: 172.16.1.0/24
    mlag_peer_ipv4_pool: 192.168.0.0/24
    mlag_peer_l3_ipv4_pool: 10.1.1.0/24
    virtual_router_mac_address: 00:1c:73:00:dc:01
    mlag_interfaces: [Ethernet55/1, Ethernet56/1]
  node_groups:
    - group: SPINES
      nodes:
        - name: spine1
          id: 1
          mgmt_ip: "192.168.101.13/24"
        - name: spine2
          id: 2
          mgmt_ip: "192.168.101.14/24"

# IDF - Leaf Switches
l2leaf:
  defaults:
    platform: cEOSLab
    mlag_peer_ipv4_pool: 192.168.0.0/24
    spanning_tree_mode: mstp
    spanning_tree_priority: 16384
    inband_mgmt_subnet: 10.10.10.0/24
    inband_mgmt_vlan: 10
  node_groups:
    - group: IDF1
      mlag: true
      uplink_interfaces: [Ethernet51]
      mlag_interfaces: [Ethernet53, Ethernet54]
      filter:
        tags: [ "110", "120" ]
      nodes:
        - name: leaf1a
          id: 3
          mgmt_ip: "192.168.101.111/24"
          uplink_switches: [SPINE1]
          uplink_switch_interfaces: [Ethernet1]
        - name: leaf1b
          id: 4
          mgmt_ip: "192.168.101.112/24"
          uplink_switches: [SPINE2]
          uplink_switch_interfaces: [Ethernet1]
```

<span style="background-color:rgb(180, 180, 180);padding: 0.2em 0.4em;font-weight: bold">group_vars/SPINES.yml: & LEAFS.yml</span> Global AVD Configuration variables which specify or designate the type of category the switch hosts belong to. The yaml file destigantes the switch host to be either a spine or a leaf in the topology.

You should notice the type designation aligns with the parameters outlined in the FABRIC.yml variables file


```yaml
---
### group_vars/SPINES.yml

type: l3spine     # Must be either spine|l3spine
```

```yaml
---
### group_vars/LEAFS.yml

type: l2leaf     # Must be l2leaf
```


<span style="background-color:rgb(180, 180, 180);padding: 0.2em 0.4em;font-weight: bold">group_vars/NETWORK_SERVICES.yml:</span> Global AVD Configuration variables applying Switched Virtual Interfaces (SVI) to the Default routing instance.

For each SVI create the assocated VLAN configuration is applied.
The SVI parameter is "tagged" to be used in the FABRIC.yml for filtering trunk links between switches.

```yaml
---
### group_vars/NETWORK_SERVICES.yml

tenants:
  - name: FABRIC
    vrfs:
      - name: default
        svis:
          - id: 110
            name: 'IDF1-Data'
            tags: ["110"]
            enabled: true
            ip_virtual_router_addresses:
              - 10.1.10.1
            nodes:
              - node: SPINE1
                ip_address: 10.1.10.2/23
              - node: SPINE2
                ip_address: 10.1.10.3/23
          - id: 120
            name: 'IDF1-Voice'
            tags: ["120"]
            enabled: true
            ip_virtual_router_addresses:
              - 10.1.20.1
            nodes:
              - node: SPINE1
                ip_address: 10.1.20.2/23
              - node: SPINE2
                ip_address: 10.1.20.3/23
```

### Build Playbook