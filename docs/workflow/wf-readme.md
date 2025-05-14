# Arista AVD Workflow

## Table of Contents
- [AVD Workflow Overview](workflow-overview)
- [Inventory Structure](#inventory-structure)
- [Role of Variables](#role-of-variables)
- [Build Playbook](#build-playbook)
- [Deploy Playbook](#deploy-playbook)

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
          - id: 130
            name: 'IDF1-Guest'
            tags: ["130"]
            enabled: true
            ip_virtual_router_addresses:
              - 10.1.30.1
            nodes:
              - node: SPINE1
                ip_address: 10.1.30.2/23
              - node: SPINE2
                ip_address: 10.1.30.3/23
```
<span style="background-color:rgb(180, 180, 180);padding: 0.2em 0.4em;font-weight: bold">group_vars/NETWORK_PORTS.yml:</span> Global AVD Configuration variables applying switch port level configuration in the form of profiles.

The following NETWORK_PORT.yml variable parameter file consists of applying port level configuration such as Vlan assignment, 802.1x, POE, and other feature in the form of profile to be assigned to the interface.

```yaml
---
### group_vars/DC1_NETWORK_PORTS.yml

### Port Profile

port_profiles:
  - profile: PP-DOT1X
    mode: "trunk phone"
    spanning_tree_portfast: edge
    spanning_tree_bpduguard: enabled
    poe:
      priority: critical
      reboot:
        action: maintain
      link_down:
        action: maintain
      shutdown:
        action: power-off
      limit:
        class: 4
    dot1x:
      port_control: auto
      reauthentication: true
      pae:
        mode: authenticator
      host_mode:
        mode: multi-host
        multi_host_authenticated: true
      mac_based_authentication:
        enabled: true
      timeout:
        reauth_period: server
        tx_period: 3
      reauthorization_request_limit: 3

# ---------------- IDF1 ----------------

# Assign switch interfaces the port porfile above

  - switches:
      - LEAF1[AB] # regex match LEAF1A & LEAF1B
    switch_ports:
      - Ethernet1-48
    description: IDF1 Standard Port
    profile: PP-DOT1X # Assigned port porfile
    native_vlan: 110
    structured_config: # Direct injection of EOS CLI-equivalent configuration into the interface, used for edge cases or features not abstracted by AVD.
      switchport:
        phone:
          trunk: untagged
          vlan: 120
    dot1x:
      authentication_failure:
        action: allow
        allow_vlan: 130

```

The global variables are in place and ready for the next steps

### Build Playbook

```yaml
---
# build.yml

- name: Build Configs
  hosts: FABRIC
  gather_facts: false
  tasks:

    - name: Generate AVD Structured Configurations and Fabric Documentation
      ansible.builtin.import_role:
        name: arista.avd.eos_designs

    - name: Generate Device Configurations and Documentation
      ansible.builtin.import_role:
        name: arista.avd.eos_cli_config_gen
```

1. <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">arista.avd.eos_designs</span>
  
    <strong>Purpose:</strong> Generates structured configuration data models from your inventory (inventory.yml, group_vars, and host_vars). It also builds fabric-wide documentation.
    
    <strong>Outputs:</strong>
    - YAML data structures per device under structured_configs/
    - Markdown-based documentation in fabric/documentation/

    <strong>What it includes:</strong>
   - Interface assignments
   - BGP/EVPN settings 
   - VLANs/SVI definitions
   - Underlay/Overlay routing logic
<br>
2. <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">arista.avd.eos_cli_config_gen</span>
  
    <strong>Purpose:</strong> Takes the structured config output from eos_designs and renders CLI-ready EOS configurations using Jinja2 templates.
    
    <strong>Outputs:</strong>
    - Flat text configuration files per device in intended_configs/
    - Optionally, generated_configlets/ for CVP Studio

    <strong>What it includes:</strong>
   - Complete running-config per device
   - Platform-specific syntax (MLAG, port-channel, BGP, etc.)
   - Ready to push to EOS or CVaaS

#### How They Work Together
1. <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">eos_designs:</span>
   - Consumes your inventory and vars.
   - Computes all design logic (interface IPs, routing adjacencies, etc.).
   - Exports structured YAML data.
2. <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">eos_cli_config_gen:</span>
   - Reads the structured YAML.
   - Renders Jinja2 templates to EOS CLI syntax.
   - Produces the actual config files and optional configlets.

💡 <strong>Think of it this way:</strong>
- <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">eos_designs</span> = "What should this network do?"
- <span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">eos_cli_config_gen</span> = "What CLI do I need to make that happen?"

### Deploy Playbook

```yaml
---
- name: Deploy Configurations to Devices Using CloudVision Portal
  hosts: DC1_FABRIC
  gather_facts: false
  connection: local
  tasks:
    - name: Push Configuration to CVaaS Studio
      ansible.builtin.import_role:
        name: arista.avd.cv_deploy
      vars:
        cv_server: www.cv-prod-us-central1-c.arista.io
        cv_token: "{{ lookup('env', 'CVP_PASSWORD') }}"

```

#### What This Playbook Does
The <strong>deploy.yml</strong> playbook pushes the rendered EOS configurations to CloudVision as-a-Service (CVaaS) using the arista.avd.cv_deploy role. Specifically:

<span style="background-color:rgb(207, 207, 207);padding: 0.2em 0.4em;font-weight: bold">cv_deploy Role Workflow:</span>
1. Reads intended configs from the intended_configs/ directory.
2. Connects to CVaaS using the cv_server and cv_token.
3. Creates or updates configlets in CloudVision Studio.
4. Assigns configlets to the appropriate devices.
5. Optionally, initiates config proposals for user approval (Studio mode).
6. Verifies assignment and returns status.
