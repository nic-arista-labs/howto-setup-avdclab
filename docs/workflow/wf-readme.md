# Arista AVD Workflow

## Table of Contents
- [AVD Workflow Overview](workflow-overview)
- [Inventory Structure](#inventory-structure)
- [Role of group_vars and host_vars](#roles-of-vars)
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

```yaml

### inventory.yml

This YAML file defines the topology (fabric) and host relationships.

```yaml
all:
  children:
    FABRIC:
      children:
        DC1_SPINES:
          hosts:
            spine1:
            spine2:
        DC1_LEAFS:
          hosts:
            leaf1:
            leaf2:

```

### Role of group_vars and host_vars

### Build Playbook