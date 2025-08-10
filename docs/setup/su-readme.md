# How-To Setup Arista AVD with Containerlabs

## Table of Contents
- [How-To Setup Arista AVD with Containerlabs](#how-to-setup-arista-avd-with-containerlabs)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step 1: Install Linux Host VM](#step-1-install-linux-host-vm)
    - [VM Install Using OrbStack (ARM Optimized)](#vm-install-using-orbstack-arm-optimized)
    - [Benefits vs Traditional VMs:](#benefits-vs-traditional-vms)
  - [Step 2: Install ContainerLab on Host VM](#step-2-install-containerlab-on-host-vm)
    - [Install VS Code](#install-vs-code)
    - [Attach to OrbStack via VSCode IDE](#attach-to-orbstack-via-vscode-ide)
    - [Install Containerlabs "All Componets" Scrript Using VSC](#install-containerlabs-all-componets-scrript-using-vsc)
  - [Step 3: Install Arista AVD on Host VM](#step-3-install-arista-avd-on-host-vm)
  - [Step 4: Import Arista cEOS-Lab Image Into Docker](#step-4-import-arista-ceos-lab-image-into-docker)
  - [Step 5: Copy AVD Examples to Working Directory](#step-5-copy-avd-examples-to-working-directory)
  - [Step 6: Modify ContainerLab Topology with AVD Inventory](#step-6-modify-containerlab-topology-with-avd-inventory)
  - [Step 7: Launch ContainerLab Arista cEOS Switches](#step-7-launch-containerlab-arista-ceos-switches)
  - [Step 8: Run AVD Build and Deploy](#step-8-run-avd-build-and-deploy)
  - [Troubleshooting](#troubleshooting)
  - [Additional Resources](#additional-resources)

## Introduction

This guide provides step-by-step instructions for setting up an Arista Validated Design (AVD) lab environment using ContainerLab with cEOS-Lab containers. The environment will allow you to:
- Simulate Arista network topologies
- Generate configuration templates
- Test network automation workflows
- Validate designs before production deployment

## Prerequisites

- [ ] VMware Workstation/Fusion or VirtualBox
- [ ] 8GB+ RAM allocated to host VM
- [ ] Ubuntu 20.04/22.04/24.04 LTS ISO
- [ ] Internet access from host VM
- [ ] Basic Linux command line knowledge

## Step 1: Install Linux Host VM

### VM Install Using OrbStack (ARM Optimized)

For faster performance on Apple Silicon (M1/M2/M3) Macs:

1. Install [OrbStack](https://orbstack.dev/download) (Docker + lightweight Linux VMs in one tool)
   I like to use [Homebrew](https://brew.sh/) to manage the packages installed on my macbook.

```bash
brew install orbstack
```

2.  Create a new image by hitting the '+' and fille out the following in the New Machine Menu
![OrbStack New Machine Menu](images/orbstack_newmachine_menu.png)

### Benefits vs Traditional VMs:
- 3x faster startup times
- Native ARM architecture support
- Seamless file sharing between Mac/VM
- Lower CPU/memory overhead

## Step 2: Install ContainerLab on Host VM

Before we can install Containerlabs on the host VM we need a way to connect to the host VM. To do that we first will install a development enviroment that will provide the client tools to connect to our new VM and other tool to interfcae with the programming langauages for AVD.

### Install VS Code 

First, We will install the Visual Studio Code (VSC) for our Integrated development environment (IDE) software use HomeBrew.

1. Open a terminal and apply the following HomeBre command to install VSCode.

```bash
brew install --cask visual-studio-code
```

2. Open VSCode

### Attach to OrbStack via VSCode IDE

Orbstack has a really cool integration for VSCode. When using the Remote Explorer extension VSC will establish a SSH connection to orb. The OrbStack linux machine will present itself.

1. Install VSC Code extension Remote Explorer (If you don't already have it)

![VSC Remote Explorer Install](images/vsc_extension_remoteExplorer.png)

2. Navigate to the Remote Explorer extension on the left nav bar and click the icon. You should see the host 'orb' in the Remotes Tunnels/SSH menu. Then client the "arrow" to launch a SSH session. 

![VSC Remote Explorer SSH Connection](images/vsc_remoteExplorer_sshConnect.png)

3. Next, we will open our project folder in VSC. OrbStack has another cool integration where it will auto-magically mount the macbook file system. We can navigate to our local host file system.

![VSC Remote Explorer Open Folder](images/vsc_explorer_openFolder.png)

### Install Containerlabs "All Componets" Scrript Using VSC

1. Open a new terminal Terminal > New Terminal

2. Run the install script from Containerlabs which will deploy all the required componets including docker

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

3. My preference to helping manage Docker images and container is to use the VSC extension for Docker. We will install on the docker extension for VSC. This feature is now built into VSC console.

![VSC Docker Extension Install](images/vsc_extension_dockerInstall.png)


## Step 3: Install Arista AVD on Orb Host VM

### Installation workflow
  - Install Python 3.10 or later
  - Install arista.avd collection including Python requirements.
  - Modify ansible.cfg file to support additional jinja2 extensions

[AVD Installation Guide](https://avd.arista.com/5.4/docs/installation/collection-installation.html)

1. Validate Python version on orb host

```bash
$ python3 --version
Python 3.12.3
```

2. Create the python virtual environment. The python AVD and Ansible libraries will be installed in this environment.

```bash
python3 -m venv myenv
```

3. Activate the python virtual environment. You should notice the courser change. 

```bash
 $ source myenv/bin/activate
(myenv) $ 
```

4. Install the PyAVD python libraries. The example code will force a specific version instead of installing the latest.

```bash
pip install "pyavd[ansible]==5.4.0"
ansible-galaxy collection install arista.avd:==5.4.0
```

## Install AVD Examples

1. Run the Examples installation Ansible playbook.

```bash
ansible-playbook arista.avd.install_examples
```

The process will pull down all the examples from the repository. This guide will focus the Campus example.

```bash
(myenv) $ ansible-playbook arista.avd.install_examples

PLAY [Install Examples] *****************************************************************************************************

TASK [Copy all examples to /Path/to/folde/] *****************************************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

2. You can list all the AVD Example folders

```bash
(myenv) $ ls -l
total 4
-rw-r--r-- 1 admin admin 445 Aug 10 11:36 ansible.cfg.old
drwxr-xr-x 1 admin admin 384 Aug 10 11:41 campus-fabric
drwxr-xr-x 1 admin admin  96 Aug 10 11:41 common
drwxr-xr-x 1 admin admin 448 Aug 10 11:41 cv-pathfinder
drwxr-xr-x 1 admin admin 384 Aug 10 11:41 dual-dc-l3ls
drwxr-xr-x 1 admin admin 384 Aug 10 11:41 isis-ldp-ipvpn
drwxr-xr-x 1 admin admin 384 Aug 10 11:41 l2ls-fabric
drwxr-xr-x 1 admin admin 224 Aug 10 11:16 myenv
drwxr-xr-x 1 admin admin 448 Aug 10 11:41 single-dc-l3ls
```

3. Navigate to the "campus-fabric" folder

```bash
(myenv) $ cd campus-fabric/
```

4. Run the AVD build playbook. The "build.yml" playbook will render device configuration from the group and host varibles provided into AVD design templates.

```bash
(myenv) $ ansible-playbook -i inventory.yml build.yml 
```

## Step 4: Import Arista cEOS-Lab Image Into Docker

## Step 5: Copy AVD Examples to Working Directory

## Step 6: Modify ContainerLab Topology with AVD Inventory

## Step 7: Launch ContainerLab Arista cEOS Switches

## Step 8: Run AVD Build and Deploy

## Troubleshooting

## Additional Resources
1. [Arista AVD Official Documentation](https://avd.arista.com/5.4/index.html)
2. [ContainerLab Documentation](https://containerlab.dev/)
3. [AVD Campus Example](https://avd.arista.com/5.4/ansible_collections/arista/avd/examples/campus-fabric/index.html)