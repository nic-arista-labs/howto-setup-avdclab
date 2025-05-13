# Arista AVD Workflow

## Table of Contents
- [AVD Workflow Overview](workflow-overview)
- [Inventory Structure](#inventory-structure)
- [Role of group_vars and host_vars](#roles-of-vars)
- [Build Playbook](#build-playbook)

### AVD Workflow Overview


This document outlines the workflow for using Arista Ansible Validated Designs (AVD) to automate and deploy network configurations to EOS devices via CloudVision as-a-Service (CVaaS).

### Inventory Structure

# Create this folder structure

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