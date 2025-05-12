# How-To Setup Arista AVD with Containerlabs

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Step 1: Install Linux Host VM](#step-1-linux-vm)
4. [Step 2: Install Docker on Host VM](#step-2-install-docker)
5. [Step 3: Install ContainerLab on Host VM](#step-3-install-clab)
6. [Step 4: Install Arista AVD on Host VM](#step-4-install-avd)
7. [Step 5: Import Arista cEOS-Lab Image Into Docker](#step-5-import-image)
8. [Step 6: Copy AVD Examples to Working Directory](#step-6-copy-avd-exmpl)
9. [Step 7: Modify ContainerLab Topology with AVD Inventory](#step-7-topo-inv-file)
10. [Step 8: Launch ContainerLab Arista cEOS Switches](#step-8-lauch-clab)
11. [Step 9: Run AVD Build and Deploy](#step-9-build-run-deploy)
12. [Troubleshooting](#tshooting)
13. [Additional Resources](#add-resource)

### Introduction

This guide provides step-by-step instructions for setting up an Arista Validated Design (AVD) lab environment using ContainerLab with cEOS-Lab containers. The environment will allow you to:
- Simulate Arista network topologies
- Generate configuration templates
- Test network automation workflows
- Validate designs before production deployment

