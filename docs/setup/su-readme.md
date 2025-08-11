# How-To Setup Arista AVD with Containerlabs

## Table of Contents

- [How-To Setup Arista AVD with Containerlabs](#how-to-setup-arista-avd-with-containerlabs)

  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step 1: Host Setup](#step-1-host-setup)
    - [Install Microsoft Visual Studio Code on Host](#install-microsoft-visual-studio-code-on-host)
    - [Install OrbStack](#install-orbstack)
    - [Attach to OrbStack via VSCode IDE](#attach-to-orbstack-via-vscode-ide)
  - [Step 2: Arista AVD Setup on Orb Host VM](#step-2-arista-avd-setup-on-orb-host-vm)
    - [Python Virtual Environment Setup](#python-virtual-environment-setup)
    - [AVD Install](#avd-install)
    - [Download AVD Examples](#download-avd-examples)
  - [Step 3. AVD Campus Example Configuration Render](#step-3-avd-campus-example-configuration-render)
    - [Navigate to Campus Fabric Folder Location](#navigate-to-campus-fabric-folder-location)
    - [Run AVD Build Playbook](#run-avd-build-playbook)
  - [Step 4. Containerlabs Environment Setup](#step-4-containerlabs-environment-setup)
    - [Install Containerlabs "All Componets" Scrript Using VSC](#install-containerlabs-all-componets-scrript-using-vsc)
    - [Download and Install Arista Container EOS](#download-and-install-arista-container-eos)
    - [Create Clab Topology](#create-clab-topology)
  - [Step 5. Deploy AVD Configuration to Containerlab hosts](#step-5-deploy-avd-configuration-to-containerlab-hosts)
  - [Additional Resources](#additional-resources)

## Introduction

This guide provides step-by-step instructions for setting up an Arista Validated Design (AVD) lab environment using ContainerLab with cEOS-Lab containers. The environment will allow you to:

- Simulate Arista network topologies
- Generate configuration templates
- Test network automation workflows
- Validate designs before production deployment

## Prerequisites

Constainer lab installation instruction on MAC [Containerlab on macOS](https://containerlab.dev/macos/)

- [ ] [OrbStack](https://containerlab.dev/macos/#:~:text=and%20ARM%20Macs.-,OrbStack,-%2D%20a%20great%20UX)
- [ ] [Containerlabs](https://containerlab.dev/install/)
- [ ] [Docker](https://docs.docker.com/engine/install/)
- [ ] Install [Python](https://avd.arista.com/5.4/docs/installation/collection-installation.html#:~:text=Install-,Python,-3.10%20or%20later) **3.10** or later
- [ ] Install [arista.avd](https://avd.arista.com/5.4/docs/installation/collection-installation.html#:~:text=Install%20arista.avd%20collection%20including%20Python%20requirements.) collection including Python requirements.

## Step 1: Host Setup

### Install Microsoft Visual Studio Code on Host

OrbStack installs remote connections with VSC and allows for access to the host's filesystem without additional setup.

This makes Orbstack and easy tool to use for sandbax setups.

VSC can be install on via download of Homebrew

1. [Download VSC](https://code.visualstudio.com/download)

2. Homebrew installation commands

   ```bash
   https://code.visualstudio.com/download
   ```

### Install OrbStack

For faster performance on Apple Silicon (M1/M2/M3) Macs:

Install [OrbStack](https://orbstack.dev/download) (Docker + lightweight Linux VMs in one tool)

1. [Download OrbStack](https://orbstack.dev/download)

2. You can use [Homebrew](https://brew.sh/) to manage the packages installed on macOS.

   ```bash
   brew install orbstack
   ```

Create a new image by hitting the '+' and fille out the following in the New Machine Menu

![OrbStack New Machine Menu](images/orbstack_newmachine_menu.png)

Benefits vs Traditional VMs

- 3x faster startup times
- Native ARM architecture support
- Seamless file sharing between Mac/VM
- Lower CPU/memory overhead

### Attach to OrbStack via VSCode IDE

Orbstack has a really cool integration for VSCode. When using the Remote Explorer extension VSC will establish a SSH connection to orb. The OrbStack linux machine will present itself.

1. Install VSC Code extension Remote Explorer (If you don't already have it)

   ![VSC Remote Explorer Install](images/vsc_extension_remoteExplorer.png)

2. Navigate to the Remote Explorer extension on the left nav bar and click the icon. You should see the host 'orb' in the Remotes Tunnels/SSH menu. Then client the "arrow" to launch a SSH session.

   ![VSC Remote Explorer SSH Connection](images/vsc_remoteExplorer_sshConnect.png)

3. Next, we will open our project folder in VSC. OrbStack has another cool integration where it will auto-magically mount the macbook file system. We can navigate to our local host file system.

   ![VSC Remote Explorer Open Folder](images/vsc_explorer_openFolder.png)

## Step 2: Arista AVD Setup on Orb Host VM

Installation workflow

- Install Python 3.10 or later
- Install arista.avd collection including Python requirements.
- Modify ansible.cfg file to support additional jinja2 extensions

### Python Virtual Environment Setup

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

### AVD Install

1. Install the PyAVD and Ansible AVD python libraries. The example code will force a specific version instead of installing the latest.

   [AVD Installation Guide](https://avd.arista.com/5.4/docs/installation/collection-installation.html)

    ```bash
    pip install "pyavd[ansible]==5.4.0"
    ansible-galaxy collection install arista.avd:==5.4.0
    ```

### Download AVD Examples

Arista AVD comes packed with pre-configured example tologies that you can download and test with.

[Arista AVD Examples Documentation](https://avd.arista.com/5.4/ansible_collections/arista/avd/examples/single-dc-l3ls/index.html)


1. Run the Examples installation Ansible playbook to download the example packes.

```bash
ansible-playbook arista.avd.install_examples
```

The process will pull down all the examples from the repository. This guide will focus the Campus example.

```bash
## Sample output
(myenv) $ ansible-playbook arista.avd.install_examples

PLAY [Install Examples] *****************************************************************************************************

TASK [Copy all examples to /Path/to/folde/] *****************************************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

## Step 3. AVD Campus Example Configuration Render

For this guide we are going to the "Campus Frabric" AVD Example to build from

After the example ansible playbook is complete you should see a list of all the AVD Example folders

```bash
## Sample output
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

### Navigate to Campus Fabric Folder Location

Navigate to the "campus-fabric" folder

```bash
(myenv) $ cd campus-fabric/
```

### Run AVD Build Playbook

Run the AVD build playbook. The "build.yml" playbook will render device configuration from the group and host varibles provided into AVD design templates.

```bash
(myenv) $ ansible-playbook -i inventory.yml build.yml 
```

The playbook will render the campus example configuration

```bash
## Sample Output
(myenv) $ ansible-playbook -i inventory.yml build.yml 

PLAY [Build Configs] *****************************************************************************************************************************************************************************************************************

TASK [arista.avd.eos_designs : Verify Requirements] **********************************************************************************************************************************************************************************
AVD version 5.4.0
Use -v for details.

TASK [arista.avd.eos_designs : Generate device configuration in structured format] ***************************************************************************************************************************************************
ok: [SPINE1 -> localhost]
ok: [SPINE2 -> localhost]
ok: [LEAF1A -> localhost]
ok: [LEAF1B -> localhost]
ok: [LEAF3A -> localhost]
ok: [LEAF3B -> localhost]
ok: [LEAF2A -> localhost]
ok: [LEAF3C -> localhost]
ok: [LEAF3D -> localhost]
ok: [LEAF3E -> localhost]
```

**Build Successful!!**

Congradulations you have successfullly rendered your device configuration using Arista AVD.
Each time you change the variables in any of the group or host var yml file the "build.yml" needs to be ran to update the configuations.

## Step 4. Containerlabs Environment Setup

The next phase on the demo is deploy this newly rendered configration onto EOS devices.

This can be accomplished using Arista's container image for EOS and loading into Docker.

Once loaded into Docker we can use Containerlabs to orchestrate the toplogy links in a destroy and deploy build like model.e

### Install Containerlabs "All Componets" Scrript Using VSC

1. Open a new terminal Terminal > New Terminal

2. Run the install script from Containerlabs which will deploy all the required componets including docker

    ```bash
    curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
    ```

3. Validate the containerlabs installation

    ```bash
    (myenv) $ containerlab version
    ____ ___  _   _ _____  _    ___ _   _ _____ ____  _       _     
    / ___/ _ \| \ | |_   _|/ \  |_ _| \ | | ____|  _ \| | __ _| |__  
    | |  | | | |  \| | | | / _ \  | ||  \| |  _| | |_) | |/ _` | '_ \ 
    | |__| |_| | |\  | | |/ ___ \ | || |\  | |___|  _ <| | (_| | |_) |
    \____\___/|_| \_| |_/_/   \_\___|_| \_|_____|_| \_\_|\__,_|_.__/ 

        version: 0.68.0
        commit: a90b6684
        date: 2025-05-05T15:36:49Z
        source: https://github.com/srl-labs/containerlab
    rel. notes: https://containerlab.dev/rn/0.68/
    ```

### Download and Install Arista Container EOS

Download the Arista container EOS version from Arista's support website

1. [Download cEOS Image](https://www.arista.com/en/support/software-download) from Arista's Support Portal

2. Load image into docker in orbstack VM

    ```bash
    docker load < ceos-lab-X.X.X.tar
    ```

3. Verify that the image is available. This is a show command to view load images in the docker instance

    ```bash
    docker images
    ```

### Create Clab Topology

Now that conatinerlabs is staged. The framework is installed and Docker has the eos image for the network.

Next we will create a containerlab topology file using yaml that will constructed the required number of nodes and make the links per the Arista Campus Fabric example.

The goal is to match the containerlab topoloy for the configuration that will be deployed form the example.

Sample Containerlabs Topo File for Arista AVD Campus Fabric

```yaml
name: avd-camp-exp-clab-topo
mgmt:
  network: clabmgmt
  bridge: clabmgmt
  ipv4-subnet: 172.16.100.0/24
topology:
  nodes:
    # Spine nodes
    SPINE1:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.101
    SPINE2:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.102
    # IDF-1
    LEAF1A:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.103
    LEAF1B:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.104
    # IDF-2
    LEAF2A:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.105
    # IDF-3
    LEAF3A:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.106
    LEAF3B:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.107
    LEAF3C:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.108
    LEAF3D:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.109
    LEAF3E:
      kind: ceos
      image: ceosarm:4.34.1F
      mgmt-ipv4: 172.16.100.110
#Container switch links
  links:
    # Spine to Spine - MLAG
    - endpoints: ["SPINE1:eth55_1", "SPINE2:eth55_1"]
    - endpoints: ["SPINE1:eth56_1", "SPINE2:eth56_1"]
    # Spine to IDF-1
    - endpoints: ["SPINE1:eth1", "LEAF1A:eth51"]
    - endpoints: ["SPINE2:eth1", "LEAF1B:eth51"]
    # Spine to IDF-2
    - endpoints: ["SPINE1:eth49_1", "LEAF2A:eth1_1"]
    - endpoints: ["SPINE2:eth49_1", "LEAF2A:eth1_3"]
    # Spine to IDF-3
    - endpoints: ["SPINE1:eth50_1", "LEAF3A:eth97_1"]
    - endpoints: ["SPINE2:eth50_1", "LEAF3A:eth97_2"]
    - endpoints: ["SPINE1:eth51_1", "LEAF3B:eth97_1"]
    - endpoints: ["SPINE2:eth51_1", "LEAF3B:eth97_2"]
    # Leaf1a to Leaf1b - MLAG
    - endpoints: ["LEAF1A:eth53", "LEAF1B:eth53"]
    - endpoints: ["LEAF1A:eth54", "LEAF1B:eth54"]
    # Leaf3a to Leaf3b - MLAG
    - endpoints: ["LEAF3A:eth98_3", "LEAF3B:eth98_3"]
    - endpoints: ["LEAF3A:eth98_4", "LEAF3B:eth98_4"]
    # Leafc
    - endpoints: ["LEAF3C:eth97_1", "LEAF3A:eth97_3"]
    - endpoints: ["LEAF3C:eth97_2", "LEAF3B:eth97_3"]
    # Leafd
    - endpoints: ["LEAF3D:eth97_1", "LEAF3A:eth97_4"]
    - endpoints: ["LEAF3D:eth97_2", "LEAF3B:eth97_4"]
    # Leafe
    - endpoints: ["LEAF3E:eth97_1", "LEAF3A:eth98_1"]
    - endpoints: ["LEAF3E:eth97_2", "LEAF3B:eth98_1"]
```

### Deploy Containerlabs topology

Deploy the containerlabs topology using the sample script provided.
This will create a setp of conatiner switch running eos.

```bash
(myenv) $containerlab -t avd-camp-exp-clab-topo.yml deploy
```

Conatinerlab will start to deploy the switches in docker
When the dployment is complete there containerlabs will provide a chart on the switches and hostnames.

```bash
14:46:27 INFO Containerlab started version=0.68.0
14:46:27 INFO Parsing & checking topology file=avd-camp-exp-clab-topo.yml
14:46:27 INFO Creating docker network name=clabmgmt IPv4 subnet=172.16.100.0/24 IPv6 subnet="" MTU=0
14:46:27 INFO Creating lab directory path=/Users/nicholas.dambrosio/Documents/Code/avd-expl-demo-clab/campus-fabric/clab-avd-camp-exp-clab-topo
14:47:07 INFO Adding host entries path=/etc/hosts
14:47:07 INFO Adding SSH config for nodes path=/etc/ssh/ssh_config.d/clab-avd-camp-exp-clab-topo.conf
🎉 A newer containerlab version (0.69.3) is available!
Release notes: https://containerlab.dev/rn/0.69/#0693
Run 'sudo clab version upgrade' or see https://containerlab.dev/install/ for installation options.
╭────────────────────────────────────┬─────────────────┬─────────┬────────────────╮
│                Name                │    Kind/Image   │  State  │ IPv4/6 Address │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF1A │ ceos            │ running │ 172.16.100.103 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF1B │ ceos            │ running │ 172.16.100.104 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF2A │ ceos            │ running │ 172.16.100.105 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF3A │ ceos            │ running │ 172.16.100.106 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF3B │ ceos            │ running │ 172.16.100.107 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF3C │ ceos            │ running │ 172.16.100.108 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF3D │ ceos            │ running │ 172.16.100.109 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-LEAF3E │ ceos            │ running │ 172.16.100.110 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-SPINE1 │ ceos            │ running │ 172.16.100.101 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
├────────────────────────────────────┼─────────────────┼─────────┼────────────────┤
│ clab-avd-camp-exp-clab-topo-SPINE2 │ ceos            │ running │ 172.16.100.102 │
│                                    │ ceosarm:4.34.1F │         │ N/A            │
╰────────────────────────────────────┴─────────────────┴─────────┴────────────────╯
```

## Step 5. Deploy AVD Configuration to Containerlab hosts

Validate the conatiner switch are running in docker

```bash
(myenv) $ docker container list
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS     NAMES
dee098f261dc   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-SPINE2
24fa250dffe9   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF3D
c1837ab98788   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-SPINE1
a62a386fdef5   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF1B
0ba3a63e044a   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF3A
06dc095cef7b   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF3C
3b8a492a0723   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF3B
b7614153359b   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF3E
5d435887aaa0   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF2A
2ecc33a39473   ceosarm:4.34.1F   "bash -c '/mnt/flash…"   2 minutes ago   Up 2 minutes             clab-avd-camp-exp-clab-topo-LEAF1A
```

And the AVD "build.yml" plyabook ran successully we can excute the deoloyment
Excute the "deploy.yml" playbook

```bash
(myenv) $ ansible-playbook -i inventory.yml deploy.yml 
```

You should see something similar to the following output.

```bash
## output is trimmed
(myenv) $ ansible-playbook -i inventory.yml deploy.yml 

PLAY [Build and Deploy Configs] ********************************************************************************************************************************************************************************************************************

TASK [arista.avd.eos_designs : Verify Requirements] ************************************************************************************************************************************************************************************************
AVD version 5.4.0

TASK [arista.avd.eos_config_deploy_eapi : Replace configuration with intended configuration] *******************************************************************************************************************************************************
ok: [SPINE2]
changed: [SPINE1]
changed: [LEAF1A]
changed: [LEAF1B]
changed: [LEAF3A]
changed: [LEAF3B]
changed: [LEAF2A]
ok: [LEAF3C]
changed: [LEAF3D]
changed: [LEAF3E]
```

At this point you have complete the AVD workflow for configuration management.
This is a very simplistic demo to outline and understand the sequence of steps AVD uses to deploy a configuration to a device.

## Additional Resources

1. [Arista AVD Official Documentation](https://avd.arista.com/5.4/index.html)
2. [ContainerLab Documentation](https://containerlab.dev/)
3. [AVD Campus Example](https://avd.arista.com/5.4/ansible_collections/arista/avd/examples/campus-fabric/index.html)