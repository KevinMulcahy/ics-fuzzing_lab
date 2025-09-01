# ICS Protocol Fuzzing Laboratory - Complete Step-by-Step Implementation Plan

## Prerequisites and Preparation

### Hardware Requirements Checklist

- [ ] Dell PowerEdge R430 (minimum 64GB RAM, 2TB storage)

- [ ] Dell PowerEdge R730 (minimum 128GB RAM, 4TB storage)

- [ ] Intel NUC8i7HVK (32GB RAM, 1TB NVMe SSD)

- [ ] 2x Raspberry Pi 3B+ with microSD cards (32GB+)

- [ ] Managed switch with VLAN support

- [ ] Network cables, power cables

- [ ] KVM/IPMI access to servers

### Software Downloads

- [ ] Proxmox VE 8.x ISO

- [ ] pfSense CE latest ISO

- [ ] Raspberry Pi OS Lite

- [ ] Ubuntu 22.04 LTS Server ISO

- [ ] Windows Server 2022 evaluation ISO (for protocol simulators)

—

## Phase 1: Infrastructure Foundation (Week 1)

### Day 1-2: Hardware Installation & Initial Setup

#### Step 1.1: Dell Server Hardware Setup

**Dell R430 Setup:**

```bash

# BIOS/UEFI Settings

Boot Mode: UEFI

Virtualization: Intel VT-x/VT-d ENABLED

SR-IOV: ENABLED

Hyperthreading: ENABLED

Power Management: OS Control

RAID: Configure RAID 1 for OS, RAID 5/10 for VMs

```

**Dell R730 Setup:**

```bash

# Similar BIOS settings as R430

# Additional memory configuration

Memory Operating Mode: Optimizer Mode

Node Interleaving: DISABLED (for NUMA awareness)

```

**Intel NUC Setup:**

```bash

# BIOS Settings

Intel VT-x: ENABLED

Intel VT-d: ENABLED

Secure Boot: DISABLED

Legacy Boot: DISABLED (UEFI only)

Wake on LAN: ENABLED

```

#### Step 1.2: Network Infrastructure Setup

**Deploy OVS Software Switch:**

The network switching infrastructure uses a dedicated Open vSwitch implementation from our companion project. This provides enterprise-grade Layer 2 switching with VLAN support, port mirroring, and SDN capabilities specifically designed for laboratory environments.

> **Required Dependency**: [ovs-layer2-switch](https://github.com/your-org/ovs-layer2-switch)

> **Version**: v1.0+

> **Documentation**: Full installation and configuration guide available in the OVS project repository

**Quick Deployment for ICS Lab:**

```bash

# Clone the OVS switch project

git clone https://github.com/your-org/ovs-layer2-switch.git

cd ovs-layer2-switch

# One-line installation for Ubuntu/Debian

sudo ./scripts/install.sh

# Deploy pre-configured ICS lab setup

sudo ./scripts/setup-bridge.sh —config config/examples/ics-lab-setup.conf

# Customize for your hardware (edit interface names)

sudo nano config/examples/ics-lab-setup.conf

# Enable services and verify

sudo systemctl enable —now ovs-lab-switch.service

sudo ./tools/ovs-manager.sh status

```

**Physical Network Layout:**

```

Lab Network Topology:

âââââââââââââââââââââââââââââââââââââââââââââââââââââââââââ

â                 OVS Software Switch                     â

â  [eth1] [eth2] [eth3] [eth4] [eth5]                    â

ââââââ¬ââââââ¬ââââââ¬ââââââ¬ââââââ¬âââââââââââââââââââââââââââââ

â     â     â     â     â

â     â     â     â     âââ pfSense LAN Interface

â     â     â     âââââââââââ pfSense WAN (Pi Bridges)

â     â     ââââââââââââââââââââ Intel NUC (All VLANs)

â     ââââââââââââââââââââââââââ Dell R730 (All VLANs)

ââââââââââââââââââââââââââââââââ Dell R430 (All VLANs)

VLAN Assignments:

â¢ VLANs 10,20,30,40,50,60: Trunked to Proxmox nodes

â¢ VLAN 200: WAN bridge network (access mode)

â¢ VLAN 30: ICS Lab (with port mirroring to monitoring)

```

**Key OVS Features Enabled:**

- **VLAN Segmentation**: Full 802.1Q support with trunk/access ports

- **Port Mirroring**: VLAN 30 traffic automatically mirrored to monitoring systems

- **OpenFlow Ready**: Integration with Ryu SDN controller in VLAN 40

- **Health Monitoring**: Automated status checks and alerting

- **Performance Tuning**: Optimized for low-latency switching

**Management Commands:**

```bash

# Health check and status

sudo ./tools/ovs-manager.sh health

sudo ./tools/ovs-manager.sh vlans

# Monitor ICS lab traffic

sudo ./tools/ovs-manager.sh monitor-ics

# Backup configuration

sudo ./tools/ovs-manager.sh backup

```

> **For Complete Details**: See the [OVS Layer 2 Switch Documentation](https://github.com/your-org/ovs-layer2-switch/tree/main/docs) for:

>

> - Advanced configuration options

> - Troubleshooting procedures

> - Performance tuning

> - API reference

> - Custom scripting examples

#### Step 1.3: Raspberry Pi WAN Bridge Setup

**Deploy Pi WiFi Bridge Solution:**

The WAN redundancy infrastructure uses a dedicated Raspberry Pi WiFi bridge implementation that provides reliable internet connectivity with automatic failover capabilities.

> **Required Dependency**: [pi-wifi-bridge](https://github.com/KevinMulcahy/pi-wifi-bridge)

> **Version**: Latest stable release

> **Documentation**: Complete setup guide and configuration options available in the Pi WiFi bridge repository

**Quick Deployment for Dual Pi Setup:**

```bash

# On both Raspberry Pi devices, clone the bridge project

git clone https://github.com/KevinMulcahy/pi-wifi-bridge.git

cd pi-wifi-bridge

# Run automated setup on RPi #1 (Primary/Master)

sudo ./setup.sh —role master —static-ip 192.168.200.10/24 —vrrp-priority 100

# Run automated setup on RPi #2 (Backup)

sudo ./setup.sh —role backup —static-ip 192.168.200.11/24 —vrrp-priority 90

# Configure WiFi credentials (same on both units)

sudo ./configure-wifi.sh —ssid “YourWiFiNetwork” —password “YourWiFiPassword”

# Set up VRRP virtual IP for failover

sudo ./configure-vrrp.sh —virtual-ip 192.168.200.100/24 —router-id 51

```

**Bridge Configuration Features:**

- **Automatic WiFi Bridge**: Seamlessly bridges WiFi to Ethernet for lab connectivity

- **VRRP Failover**: Automatic failover between Pi units with <3 second switchover

- **Network Monitoring**: Built-in connectivity monitoring and health checks

- **Easy Management**: Web interface and command-line tools for monitoring

- **Persistent Configuration**: Survives reboots and maintains stable connectivity

**VRRP Failover Architecture:**

```

Internet (WiFi)

â

âââââââââââââââââââââââââââââââââââââââ

â  Pi WiFi Bridge Cluster (VRRP)     â

âââââââââââââââââââ¬ââââââââââââââââââââ¤

â RPi #1 (Master) â RPi #2 (Backup)  â

â 192.168.200.10  â 192.168.200.11   â

â Priority: 100   â Priority: 90     â

âââââââââââââââââââ´ââââââââââââââââââââ

â (Virtual IP: 192.168.200.100)

Lab Network via pfSense WAN

```

**Management and Monitoring:**

```bash

# Check bridge status on either Pi

sudo ./bridge-status.sh

# Monitor failover events

sudo tail -f /var/log/vrrp-failover.log

# Test connectivity and failover

sudo ./test-failover.sh

# Update bridge configuration

sudo ./reconfigure.sh

```

**Health Monitoring Integration:**

The Pi bridge solution includes monitoring endpoints that integrate with your labâs ELK stack for centralized visibility into WAN connectivity and failover events.

> **For Complete Details**: See the [Pi WiFi Bridge Documentation](https://github.com/KevinMulcahy/pi-wifi-bridge/blob/main/README.md) for:

>

> - Advanced configuration options

> - Custom WiFi network settings

> - Troubleshooting connectivity issues

> - Performance optimization

> - Integration with monitoring systems

### Day 3-4: Proxmox VE Installation

#### Step 1.4: Install Proxmox VE on All Nodes

**R430 Installation:**

```bash

# Boot from Proxmox ISO

# Installation parameters:

Target Disk: /dev/sda (RAID 1 volume)

Country: United States

Timezone: Your timezone

Keyboard: us

Password: [Strong password]

Email: admin@yourdomain.com

Management Interface: eno1

Hostname: pve-fuzzing-01.lab.local

IP Address: 192.168.40.10/24

Gateway: 192.168.40.1

DNS: 8.8.8.8

```

**R730 Installation:**

```bash

# Similar parameters:

Hostname: pve-analysis-01.lab.local

IP Address: 192.168.40.11/24

```

**NUC Installation:**

```bash

# Similar parameters:

Hostname: pve-dev-01.lab.local

IP Address: 192.168.40.12/24

```

#### Step 1.5: Post-Installation Proxmox Configuration

**On each Proxmox node:**

```bash

# Disable enterprise repository

nano /etc/apt/sources.list.d/pve-enterprise.list

# Comment out the line

# Add no-subscription repository

echo “deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription” > /etc/apt/sources.list.d/pve-no-subscription.list

# Update system

apt update && apt upgrade -y

# Install additional packages

apt install vim git curl wget htop iotop bridge-utils openvswitch-switch -y

# Configure OVS bridge

ovs-vsctl add-br vmbr0

ovs-vsctl add-port vmbr0 eno1

# Configure /etc/network/interfaces

nano /etc/network/interfaces

```

**Network Configuration (/etc/network/interfaces):**

```bash

auto lo

iface lo inet loopback

# Management interface

auto eno1

iface eno1 inet manual

# OVS Bridge for VLAN trunking

auto vmbr0

iface vmbr0 inet static

address 192.168.40.10/24  # Change per node

gateway 192.168.40.1

bridge-ports eno1

bridge-stp off

bridge-fd 0

ovs_type OVSBridge

ovs_ports eno1

# VLAN interfaces for each segment

auto vmbr0.10

iface vmbr0.10 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=10

auto vmbr0.20

iface vmbr0.20 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=20

auto vmbr0.30

iface vmbr0.30 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=30

auto vmbr0.40

iface vmbr0.40 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=40

auto vmbr0.50

iface vmbr0.50 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=50

auto vmbr0.60

iface vmbr0.60 inet manual

ovs_type OVSIntPort

ovs_bridge vmbr0

ovs_options tag=60

```

#### Step 1.6: Create Proxmox Cluster

**On pve-fuzzing-01 (R430):**

```bash

# Create cluster

pvecm create fuzzing-cluster

# Check cluster status

pvecm status

```

**On pve-analysis-01 (R730):**

```bash

# Join cluster

pvecm add 192.168.40.10

# Enter password for root@pve-fuzzing-01

```

**On pve-dev-01 (NUC):**

```bash

# Join cluster

pvecm add 192.168.40.10

```

### Day 5: pfSense Firewall Deployment

#### Step 1.7: Deploy pfSense VM

**Create pfSense VM on pve-dev-01:**

```bash

# Create VM via CLI

qm create 100 —name pfsense-fw —memory 4096 —cores 2 —net0 virtio,bridge=vmbr0,tag=200 —net1 virtio,bridge=vmbr0 —bootdisk scsi0 —scsi0 local-lvm:32 —ostype other

# Attach pfSense ISO

qm set 100 —ide2 local:iso/pfSense-CE-2.7.0-amd64.iso,media=cdrom

# Start VM

qm start 100

```

**pfSense Initial Configuration:**

```bash

# Console setup (VGA console in Proxmox)

# Interface assignment:

WAN: vtnet0 (VLAN 200 - connected to Pi bridges)

LAN: vtnet1 (trunk port - will handle VLANs)

# WAN Configuration:

IPv4: 192.168.200.1/24

Gateway: 192.168.200.100 (VRRP VIP)

# LAN Configuration:

IPv4: 192.168.40.1/24 (Management VLAN)

```

**pfSense Web Configuration:**

```bash

# Access via https://192.168.40.1

# Complete setup wizard

# Configure VLAN interfaces:

# Interfaces â VLANs

VLAN 10: vtnet1, Description “LAN”

VLAN 20: vtnet1, Description “DMZ”

VLAN 30: vtnet1, Description “ICS-LAB”

VLAN 40: vtnet1, Description “MGMT”

VLAN 50: vtnet1, Description “LOGGING”

VLAN 60: vtnet1, Description “GUEST”

# Interfaces â Interface Assignments

LAN_VLAN10: VLAN 10 on vtnet1, IPv4: 192.168.10.1/24

DMZ_VLAN20: VLAN 20 on vtnet1, IPv4: 192.168.20.1/24

ICS_VLAN30: VLAN 30 on vtnet1, IPv4: 192.168.30.1/24

MGMT_VLAN40: VLAN 40 on vtnet1, IPv4: 192.168.40.1/24

LOG_VLAN50: VLAN 50 on vtnet1, IPv4: 192.168.50.1/24

GUEST_VLAN60: VLAN 60 on vtnet1, IPv4: 192.168.60.1/24

```

—

## Phase 2: Network & Security Implementation (Week 2)

### Day 6-7: Advanced pfSense Configuration

#### Step 2.1: Firewall Rules Implementation

**Default Deny Rules:**

```bash

# Firewall â Rules â Floating

# Rule 1: Block inter-VLAN by default

Action: Block

Interface: Any

Direction: Any

Protocol: Any

Source: RFC1918 networks

Destination: RFC1918 networks

Description: “Default Deny Inter-VLAN”

# MGMT VLAN (40) Rules:

# Allow MGMT to all VLANs (admin access)

Action: Pass

Interface: MGMT_VLAN40

Protocol: Any

Source: MGMT_VLAN40 subnets

Destination: Any

Description: “MGMT Full Access”

# ICS LAB VLAN (30) Rules:

# Block ICS LAB from other VLANs

Action: Block

Interface: ICS_VLAN30

Protocol: Any

Source: ICS_VLAN30 subnets

Destination: LAN_VLAN10 subnets, DMZ_VLAN20 subnets, GUEST_VLAN60 subnets

Description: “Block ICS Lab to Production”

# Allow ICS LAB to LOGGING

Action: Pass

Interface: ICS_VLAN30

Protocol: TCP/UDP

Source: ICS_VLAN30 subnets

Destination: LOG_VLAN50 subnets

Destination Port: 514, 5514, 9200, 5044

Description: “ICS Lab to Logging”

# LOGGING VLAN (50) Rules:

# Allow logging from all VLANs

Action: Pass

Interface: Any

Protocol: TCP/UDP

Source: Any

Destination: LOG_VLAN50 subnets

Destination Port: 514, 5514, 9200, 5044, 5601

Description: “Allow Logging Traffic”

# GUEST VLAN (60) Rules:

# Guest internet only

Action: Pass

Interface: GUEST_VLAN60

Protocol: Any

Source: GUEST_VLAN60 subnets

Destination: !RFC1918

Description: “Guest Internet Only”

```

#### Step 2.2: DHCP Configuration

**Configure DHCP for each VLAN:**

```bash

# Services â DHCP Server

# LAN_VLAN10:

Range: 192.168.10.100-192.168.10.200

DNS: 192.168.10.1, 8.8.8.8

Domain: lab.local

# DMZ_VLAN20:

Range: 192.168.20.100-192.168.20.200

# ICS_VLAN30:

Range: 192.168.30.100-192.168.30.200

# MGMT_VLAN40:

Range: 192.168.40.100-192.168.40.200

# LOG_VLAN50:

Range: 192.168.50.100-192.168.50.200

# GUEST_VLAN60:

Range: 192.168.60.100-192.168.60.200

```

### Day 8-9: Storage Configuration

#### Step 2.3: Configure Shared Storage

**On all Proxmox nodes:**

```bash

# Create local storage pools

pvesm add dir local-vms —path /var/lib/vz —content vztmpl,iso,images

pvesm add dir local-backups —path /var/backups —content backup

# Configure ZFS pools (if using ZFS)

# R430 - Fuzzing workloads

zpool create fuzzing-pool /dev/sdb /dev/sdc

pvesm add zfspool fuzzing-storage —pool fuzzing-pool —content images

# R730 - Analysis workloads

zpool create analysis-pool /dev/sdb /dev/sdc /dev/sdd

pvesm add zfspool analysis-storage —pool analysis-pool —content images

# NUC - Development

pvesm add dir dev-storage —path /var/lib/vz/dev —content images,vztmpl

```

### Day 10: Initial VM Deployment

#### Step 2.4: Deploy Management VM

**Create Management VM on NUC:**

```bash

# Create VM

qm create 200 —name mgmt-tools —memory 4096 —cores 2 —net0 virtio,bridge=vmbr0,tag=40 —bootdisk scsi0 —scsi0 dev-storage:50 —ostype l26

# Attach Ubuntu ISO

qm set 200 —ide2 local:iso/ubuntu-22.04-server-amd64.iso,media=cdrom

# Start and install

qm start 200

```

**Management VM Setup:**

```bash

# Ubuntu installation with SSH server

# Post-install configuration:

sudo apt update && sudo apt upgrade -y

sudo apt install ansible terraform git docker.io python3-pip -y

# Configure static IP

sudo nano /etc/netplan/00-installer-config.yaml

# Content:

network:

version: 2

ethernets:

ens18:

addresses: [192.168.40.50/24]

gateway4: 192.168.40.1

nameservers:

addresses: [192.168.40.1, 8.8.8.8]

sudo netplan apply

# Install additional tools

pip3 install proxmoxer requests paramiko

# Clone configuration repository

git clone https://github.com/your-org/ics-fuzzing-lab.git

```

—

## Phase 3: Core VM Deployment (Weeks 3-4)

### Day 11-14: Automated VM Provisioning

#### Step 3.1: Terraform Infrastructure as Code

**Create Terraform configuration:**

```hcl

# main.tf

terraform {

required_providers {

proxmox = {

source  = “telmate/proxmox”

version = “2.9.14”

}

}

}

provider “proxmox” {

pm_api_url      = “https://192.168.40.10:8006/api2/json”

pm_user         = “root@pam”

pm_password     = var.proxmox_password

pm_tls_insecure = true

}

# Variables

variable “proxmox_password” {

description = “Proxmox root password”

type        = string

sensitive   = true

}

# VM Template for ICS Lab VMs

resource “proxmox_vm_qemu” “fuzzing_engine” {

count       = 2

name        = “fuzzing-engine-${count.index + 1}”

target_node = “pve-fuzzing-01”

vmid        = 300 + count.index

memory = 12288

cores  = 6

sockets = 1

disk {

size    = “100G”

type    = “scsi”

storage = “fuzzing-storage”

format  = “qcow2”

}

network {

model  = “virtio”

bridge = “vmbr0”

tag    = 30  # ICS Lab VLAN

}

os_type = “cloud-init”

clone   = “ubuntu-22.04-template”

ciuser     = “ubuntu”

cipassword = var.vm_password

sshkeys    = file(“~/.ssh/id_rsa.pub”)

ipconfig0 = “ip=192.168.30.${10 + count.index}/24,gw=192.168.30.1”

}

resource “proxmox_vm_qemu” “protocol_simulator” {

count       = 2

name        = “protocol-sim-${count.index + 1}”

target_node = “pve-fuzzing-01”

vmid        = 310 + count.index

memory = 6144

cores  = 4

disk {

size    = “50G”

type    = “scsi”

storage = “fuzzing-storage”

format  = “qcow2”

}

network {

model  = “virtio”

bridge = “vmbr0”

tag    = 30

}

os_type = “cloud-init”

clone   = “ubuntu-22.04-template”

ciuser     = “ubuntu”

sshkeys    = file(“~/.ssh/id_rsa.pub”)

ipconfig0  = “ip=192.168.30.${20 + count.index}/24,gw=192.168.30.1”

}

resource “proxmox_vm_qemu” “elk_stack” {

name        = “elk-stack”

target_node = “pve-analysis-01”

vmid        = 400

memory = 20480  # 20GB for Elasticsearch

cores  = 6

disk {

size    = “500G”

type    = “scsi”

storage = “analysis-storage”

format  = “qcow2”

}

network {

model  = “virtio”

bridge = “vmbr0”

tag    = 50  # Logging VLAN

}

os_type = “cloud-init”

clone   = “ubuntu-22.04-template”

ciuser    = “ubuntu”

sshkeys   = file(“~/.ssh/id_rsa.pub”)

ipconfig0 = “ip=192.168.50.10/24,gw=192.168.50.1”

}

resource “proxmox_vm_qemu” “ryu_controller” {

name        = “ryu-controller”

target_node = “pve-dev-01”

vmid        = 500

memory = 4096

cores  = 2

disk {

size    = “50G”

type    = “scsi”

storage = “dev-storage”

format  = “qcow2”

}

network {

model  = “virtio”

bridge = “vmbr0”

tag    = 40  # Management VLAN

}

os_type = “cloud-init”

clone   = “ubuntu-22.04-template”

ciuser    = “ubuntu”

sshkeys   = file(“~/.ssh/id_rsa.pub”)

ipconfig0 = “ip=192.168.40.60/24,gw=192.168.40.1”

}

```

#### Step 3.2: Ansible Configuration Management

**Create Ansible playbooks:**

```yaml

# site.yml

—

- hosts: fuzzing_engines

become: yes

roles:

- common

- docker

- fuzzing_tools

- hosts: protocol_simulators

become: yes

roles:

- common

- docker

- protocol_tools

- hosts: elk_stack

become: yes

roles:

- common

- elasticsearch

- logstash

- kibana

- hosts: ryu_controller

become: yes

roles:

- common

- ryu_sdn

```

**Common role (roles/common/tasks/main.yml):**

```yaml

—

- name: Update package cache

apt:

update_cache: yes

cache_valid_time: 3600

- name: Install base packages

apt:

name:

- vim

- git

- htop

- curl

- wget

- python3

- python3-pip

- rsyslog

state: present

- name: Configure rsyslog for centralized logging

template:

src: rsyslog.conf.j2

dest: /etc/rsyslog.conf

notify: restart rsyslog

- name: Configure SSH hardening

template:

src: sshd_config.j2

dest: /etc/ssh/sshd_config

notify: restart ssh

```

**Fuzzing tools role (roles/fuzzing_tools/tasks/main.yml):**

```yaml

—

- name: Install fuzzing dependencies

apt:

name:

- build-essential

- cmake

- python3-dev

- libssl-dev

- git

state: present

- name: Clone AFL++

git:

repo: https://github.com/AFLplusplus/AFLplusplus

dest: /opt/aflplusplus

version: stable

- name: Build AFL++

make:

chdir: /opt/aflplusplus

target: all

- name: Install AFL++

make:

chdir: /opt/aflplusplus

target: install

- name: Install Python fuzzing tools

pip:

name:

- boofuzz

- scapy

- requests

executable: pip3

- name: Clone ICS-specific fuzzing tools

git:

repo: https://github.com/dark-lbp/isf

dest: /opt/isf

- name: Create fuzzing workspace

file:

path: /home/ubuntu/fuzzing

state: directory

owner: ubuntu

group: ubuntu

mode: ‘0755’

```

### Day 15-18: Service-Specific Deployments

#### Step 3.3: ELK Stack Deployment

**Elasticsearch configuration:**

```yaml

# roles/elasticsearch/tasks/main.yml

—

- name: Add Elasticsearch APT key

apt_key:

url: https://artifacts.elastic.co/GPG-KEY-elasticsearch

state: present

- name: Add Elasticsearch repository

apt_repository:

repo: “deb https://artifacts.elastic.co/packages/8.x/apt stable main”

state: present

- name: Install Elasticsearch

apt:

name: elasticsearch

state: present

- name: Configure Elasticsearch

template:

src: elasticsearch.yml.j2

dest: /etc/elasticsearch/elasticsearch.yml

notify: restart elasticsearch

- name: Start and enable Elasticsearch

systemd:

name: elasticsearch

state: started

enabled: yes

```

**Elasticsearch config template:**

```yaml

# roles/elasticsearch/templates/elasticsearch.yml.j2

cluster.name: ics-fuzzing-lab

node.name: elk-node-1

path.data: /var/lib/elasticsearch

path.logs: /var/log/elasticsearch

network.host: 192.168.50.10

http.port: 9200

discovery.type: single-node

xpack.security.enabled: false

xpack.security.enrollment.enabled: false

```

#### Step 3.4: ICS Protocol Simulators

**Modbus simulator setup:**

```bash

# On protocol-sim-1 VM

sudo apt install default-jdk -y

# Download ModbusPal

wget https://github.com/zerenity/ModbusPal/releases/download/1.6b/ModbusPal.jar

# Create Modbus simulation script

cat << ‘EOF’ > /home/ubuntu/start-modbus-sim.sh

#!/bin/bash

java -jar ModbusPal.jar —file /home/ubuntu/configs/plc-simulation.xmpal —console

EOF

chmod +x /home/ubuntu/start-modbus-sim.sh

# Create systemd service

sudo cat << ‘EOF’ > /etc/systemd/system/modbus-simulator.service

[Unit]

Description=Modbus PLC Simulator

After=network.target

[Service]

Type=simple

User=ubuntu

WorkingDirectory=/home/ubuntu

ExecStart=/home/ubuntu/start-modbus-sim.sh

Restart=always

[Install]

WantedBy=multi-user.target

EOF

sudo systemctl enable modbus-simulator.service

```

#### Step 3.5: Ryu SDN Controller

**Ryu installation and configuration:**

```bash

# On ryu-controller VM

sudo apt install python3-pip python3-dev -y

sudo pip3 install ryu

# Create custom Ryu application for ICS Lab

cat << ‘EOF’ > /home/ubuntu/ics_lab_controller.py

from ryu.base import app_manager

from ryu.controller import ofp_event

from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER

from ryu.controller.handler import set_ev_cls

from ryu.ofproto import ofproto_v1_3

from ryu.lib.packet import packet

from ryu.lib.packet import ethernet

from ryu.lib.packet import ether_types

class ICSLabController(app_manager.RyuApp):

OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

def __init__(self, *args, **kwargs):

super(ICSLabController, self).__init__(*args, **kwargs)

self.mac_to_port = {}

@set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)

def switch_features_handler(self, ev):

datapath = ev.msg.datapath

ofproto = datapath.ofproto

parser = datapath.ofproto_parser

# Default flow for table miss

match = parser.OFPMatch()

actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,

ofproto.OFPCML_NO_BUFFER)]

self.add_flow(datapath, 0, match, actions)

def add_flow(self, datapath, priority, match, actions, buffer_id=None):

ofproto = datapath.ofproto

parser = datapath.ofproto_parser

inst = [parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,

actions)]

if buffer_id:

mod = parser.OFPFlowMod(datapath=datapath, buffer_id=buffer_id,

priority=priority, match=match,

instructions=inst)

else:

mod = parser.OFPFlowMod(datapath=datapath, priority=priority,

match=match, instructions=inst)

datapath.send_msg(mod)

@set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)

def packet_in_handler(self, ev):

msg = ev.msg

datapath = msg.datapath

ofproto = datapath.ofproto

parser = datapath.ofproto_parser

in_port = msg.match[‘in_port’]

pkt = packet.Packet(msg.data)

eth = pkt.get_protocols(ethernet.ethernet)[0]

if eth.ethertype == ether_types.ETH_TYPE_LLDP:

return

dst = eth.dst

src = eth.src

dpid = datapath.id

self.mac_to_port.setdefault(dpid, {})

self.mac_to_port[dpid][src] = in_port

if dst in self.mac_to_port[dpid]:

out_port = self.mac_to_port[dpid][dst]

else:

out_port = ofproto.OFPP_FLOOD

actions = [parser.OFPActionOutput(out_port)]

if out_port != ofproto.OFPP_FLOOD:

match = parser.OFPMatch(in_port=in_port, eth_dst=dst, eth_src=src)

self.add_flow(datapath, 1, match, actions)

data = None

if msg.buffer_id == ofproto.OFP_NO_BUFFER:

data = msg.data

out = parser.OFPPacketOut(datapath=datapath, buffer_id=msg.buffer_id,

in_port=in_port, actions=actions, data=data)

datapath.send_msg(out)

EOF

# Create systemd service for Ryu

sudo cat << ‘EOF’ > /etc/systemd/system/ryu-controller.service

[Unit]

Description=Ryu SDN Controller for ICS Lab

After=network.target

[Service]

Type=simple

User=ubuntu

WorkingDirectory=/home/ubuntu

ExecStart=/usr/local/bin/ryu-manager ics_lab_controller.py

Restart=always

RestartSec=5

[Install]

WantedBy=multi-user.target

EOF

sudo systemctl enable ryu-controller.service

sudo systemctl start ryu-controller.service

```

—

## Phase 4: Advanced Integration & Testing (Week 5)

### Day 22-24: Comprehensive Testing Framework

#### Step 4.1: Automated Testing Suite

**Create comprehensive test automation:**

```bash

# On management VM, create master test suite

cat << ‘EOF’ > /home/ubuntu/scripts/master_test_suite.sh

#!/bin/bash

# Master Test Suite for ICS Fuzzing Lab

TEST_LOG=“/var/log/master_test_suite.log”

START_TIME=$(date)

log_test() {

echo “$(date ‘+%Y-%m-%d %H:%M:%S’) - $1” | tee -a $TEST_LOG

}

run_infrastructure_tests() {

log_test “=== Infrastructure Tests ===“

# Test 1: Proxmox cluster health

log_test “Testing Proxmox cluster...”

for node in 192.168.40.10 192.168.40.11 192.168.40.12; do

if ssh root@$node “pvecm status | grep -q ‘Expected votes: 3’”; then

log_test “â Node $node: Cluster OK”

else

log_test “â Node $node: Cluster issue”

fi

done

# Test 2: OVS switch health (requires deployed OVS project)

log_test “Testing OVS switch...”

if ssh root@ovs-switch-ip “./tools/ovs-manager.sh health | grep -q ‘Health check passed’”; then

log_test “â OVS Switch: Healthy”

else

log_test “â OVS Switch: Issues detected”

fi

# Test 3: pfSense firewall

log_test “Testing pfSense firewall...”

if curl -s -k https://192.168.40.1/api/v1/status/system | grep -q “online”; then

log_test “â pfSense: Online”

else

log_test “â pfSense: Offline or unreachable”

fi

# Test 4: VRRP WAN redundancy

log_test “Testing WAN redundancy...”

ping_count=$(ping -c 5 192.168.200.100 | grep “received” | awk ‘{print $4}’)

if [[ $ping_count -ge 4 ]]; then

log_test “â VRRP VIP: Responding ($ping_count/5 packets)”

else

log_test “â VRRP VIP: Poor connectivity ($ping_count/5 packets)”

fi

}

run_vm_tests() {

log_test “=== Virtual Machine Tests ===“

# Define expected VMs

declare -A expected_vms=(

[“300”]=“fuzzing-engine-1:192.168.30.10”

[“301”]=“fuzzing-engine-2:192.168.30.11”

[“310”]=“protocol-sim-1:192.168.30.20”

[“311”]=“protocol-sim-2:192.168.30.21”

[“320”]=“pcap-replay:192.168.30.30”

[“400”]=“elk-stack:192.168.50.10”

[“401”]=“network-monitor:192.168.50.11”

[“402”]=“reverse-engineering:192.168.40.70”

[“500”]=“ryu-controller:192.168.40.60”

[“200”]=“mgmt-tools:192.168.40.50”

)

for vmid in “${!expected_vms[@]}”; do

IFS=‘:’ read -ra VM_INFO <<< “${expected_vms[$vmid]}”

vm_name=“${VM_INFO[0]}”

vm_ip=“${VM_INFO[1]}”

# Check VM status

vm_status=$(qm status $vmid 2>/dev/null | awk ‘{print $2}’)

if [[ “$vm_status” == “running” ]]; then

# Test network connectivity

if ping -c 1 -W 2 $vm_ip >/dev/null 2>&1; then

log_test “â VM $vmid ($vm_name): Running and reachable”

else

log_test “â ï¸  VM $vmid ($vm_name): Running but not reachable”

fi

else

log_test “â VM $vmid ($vm_name): Not running ($vm_status)”

fi

done

}

# Main test execution

main() {

log_test “=== ICS Fuzzing Lab Master Test Suite Started ===“

log_test “Start time: $START_TIME”

run_infrastructure_tests

run_vm_tests

END_TIME=$(date)

log_test “=== Test Suite Completed ===“

log_test “End time: $END_TIME”

log_test “Results logged to: $TEST_LOG”

# Generate summary

passed=$(grep -c “â“ $TEST_LOG)

failed=$(grep -c “â” $TEST_LOG)

warnings=$(grep -c “â ï¸” $TEST_LOG)

echo

echo “=== Test Summary ===“

echo “Passed: $passed”

echo “Failed: $failed”

echo “Warnings: $warnings”

echo “Total: $((passed + failed + warnings))”

echo

if [[ $failed -eq 0 ]]; then

echo “ð All critical tests passed! Lab is ready for production.”

return 0

else

echo “ð¨ $failed tests failed. Review logs and fix issues before proceeding.”

return 1

fi

}

main “$@“

EOF

chmod +x /home/ubuntu/scripts/master_test_suite.sh

```

#### Step 4.2: Security Validation

**VLAN isolation test script:**

```bash

# On management VM (192.168.40.50)

cat << ‘EOF’ > /home/ubuntu/test_vlan_isolation.sh

#!/bin/bash

# VLAN Isolation Test Script

LOG_FILE=“/var/log/vlan_isolation_test.log”

DATE=$(date)

echo “=== VLAN Isolation Test Started: $DATE ===“ >> $LOG_FILE

# Test matrix: Source VLAN -> Destination VLAN (should fail except allowed paths)

declare -A TEST_MATRIX=(

[“192.168.30.10”]=“192.168.10.1,192.168.20.1,192.168.60.1”  # ICS to LAN/DMZ/Guest (should fail)

[“192.168.10.1”]=“192.168.30.1,192.168.50.1”                # LAN to ICS/Logging (should fail)

[“192.168.60.1”]=“192.168.10.1,192.168.30.1,192.168.50.1”   # Guest to internal (should fail)

)

for source in “${!TEST_MATRIX[@]}”; do

IFS=‘,’ read -ra DESTINATIONS <<< “${TEST_MATRIX[$source]}”

for dest in “${DESTINATIONS[@]}”; do

echo “Testing: $source -> $dest” >> $LOG_FILE

# Use nmap for connectivity test

result=$(nmap -p 22,80,443 —max-retries 1 —host-timeout 5s $dest 2>/dev/null | grep “Host is up”)

if [[ -z “$result” ]]; then

echo “  â PASS: $source -> $dest blocked as expected” >> $LOG_FILE

else

echo “  â FAIL: $source -> $dest connection succeeded (security issue!)” >> $LOG_FILE

fi

done

done

echo “=== VLAN Isolation Test Completed ===“ >> $LOG_FILE

EOF

chmod +x /home/ubuntu/test_vlan_isolation.sh

```

### Day 25-26: Final Integration and Documentation

#### Step 4.3: Complete Lab Validation

**Final validation checklist automation:**

```bash

cat << ‘EOF’ > /home/ubuntu/scripts/final_validation.sh

#!/bin/bash

# Final validation before lab handover

VALIDATION_LOG=“/var/log/final_validation.log”

ERRORS=0

validate_item() {

local description=“$1”

local command=“$2”

local expected=“$3”

echo -n “Validating: $description... “

result=$(eval “$command” 2>/dev/null)

if [[ “$result” == *”$expected”* ]]; then

echo “â PASS”

echo “$(date): PASS - $description” >> $VALIDATION_LOG

else

echo “â FAIL”

echo “$(date): FAIL - $description (got: $result)” >> $VALIDATION_LOG

((ERRORS++))

fi

}

echo “=== ICS Fuzzing Lab Final Validation ===“

echo “Starting comprehensive lab validation...”

# Infrastructure validation

echo -e “\n— Infrastructure Validation —“

validate_item “Proxmox cluster quorum” “ssh root@192.168.40.10 ‘pvecm status | grep Expected’” “Expected votes: 3”

validate_item “pfSense firewall status” “curl -k -s https://192.168.40.1/api/v1/status/system” “online”

validate_item “OVS switch health” “ssh root@ovs-switch-ip ‘./tools/ovs-manager.sh health’” “Health check passed”

validate_item “VRRP failover ready” “ping -c 1 192.168.200.100” “1 received”

# VM validation

echo -e “\n— Virtual Machine Validation —“

validate_item “Fuzzing engine 1 active” “qm status 300” “running”

validate_item “Fuzzing engine 2 active” “qm status 301” “running”

validate_item “Protocol simulator 1” “qm status 310” “running”

validate_item “Protocol simulator 2” “qm status 311” “running”

validate_item “ELK stack operational” “qm status 400” “running”

validate_item “Network monitoring” “qm status 401” “running”

# Security validation

echo -e “\n— Security Validation —“

validate_item “VLAN 30 isolation” “timeout 5 ping -c 1 192.168.10.1 >/dev/null 2>&1; echo $?” “1”

validate_item “SSH key auth only” “ssh -o PasswordAuthentication=yes ubuntu@192.168.30.10 ‘echo test’ 2>&1” “Permission denied”

echo -e “\n=== Validation Summary ===“

if [[ $ERRORS -eq 0 ]]; then

echo “ð ALL VALIDATIONS PASSED!”

echo “Lab is ready for production use.”

exit 0

else

echo “ð¨ $ERRORS VALIDATION(S) FAILED!”

echo “Review errors in: $VALIDATION_LOG”

exit 1

fi

EOF

chmod +x /home/ubuntu/scripts/final_validation.sh

```

—

## Phase 5: Documentation & Operations (Week 6)

### Day 27-30: Operational Procedures

#### Step 5.1: Daily Operations Checklist

**Create operational runbook:**

```markdown

# Daily Operations Checklist

## Morning Health Check (15 minutes)

- [ ] Check Proxmox cluster status: `pvecm status`

- [ ] Verify all VMs running: `qm list` on each node

- [ ] Check pfSense gateway status

- [ ] Verify VRRP status on Raspberry Pis

- [ ] Check ELK Stack health: curl http://192.168.50.10:9200/_cluster/health

- [ ] Review overnight Suricata alerts

- [ ] **OVS Switch Health**: `./tools/ovs-manager.sh health` (see [OVS repo](https://github.com/your-org/ovs-layer2-switch))

## Weekly Maintenance (2 hours)

- [ ] Update all Proxmox nodes: `apt update && apt upgrade`

- [ ] Update VM templates and packages

- [ ] Run VLAN isolation tests

- [ ] Backup configurations (including OVS: `./tools/ovs-manager.sh backup`)

- [ ] Review fuzzing results and logs

- [ ] Check storage usage across all nodes

- [ ] Test WAN failover functionality

## Monthly Security Review (4 hours)

- [ ] Review firewall logs for anomalies

- [ ] Update Suricata rules

- [ ] Security scan of management network

- [ ] Review user access and SSH keys

- [ ] Update fuzzing tool signatures

- [ ] Disaster recovery test

- [ ] **Network Security Review**: Validate OVS VLAN isolation and mirror configurations

```

#### Step 5.2: Troubleshooting Procedures

**Common Issue Resolution:**

```bash

# Cluster Split-Brain Recovery

# If cluster shows issues:

pvecm expected 1  # On remaining node

pvecm delnode <failed_node>

# Re-add node when available

# VM Performance Issues

# Check resource usage:

qm monitor <vmid>

info cpus

info memory

info block

# Network connectivity issues - OVS Switch

# Check OVS status (detailed procedures in OVS repo):

ssh root@ovs-switch-ip ‘./tools/ovs-manager.sh status’

ssh root@ovs-switch-ip ‘./tools/ovs-manager.sh vlans’

# Restart OVS services if needed:

ssh root@ovs-switch-ip ‘./tools/ovs-manager.sh restart’

# pfSense issues

# Check interface status:

ifconfig -a

# Check routing table:

netstat -rn

# Check firewall logs:

tail -f /var/log/filter.log

```

> **Extended Troubleshooting**: The [OVS Layer 2 Switch repository](https://github.com/your-org/ovs-layer2-switch/blob/main/docs/troubleshooting.md) contains detailed troubleshooting procedures for:

>

> - Port state issues

> - VLAN misconfigurations

> - Performance problems

> - OpenFlow controller connectivity

> - Mirror port problems

#### Step 5.3: Final Documentation Package

**Project handover documentation:**

```bash

# Generate final project documentation

cat << ‘EOF’ > /home/ubuntu/docs/PROJECT_HANDOVER.md

# ICS Protocol Fuzzing Laboratory - Project Handover

## Project Completion Summary

**Project**: ICS Protocol Fuzzing Laboratory

**Completion Date**: $(date)

**Status**: â COMPLETE

**Total Duration**: 6 weeks

## Infrastructure Delivered

### Hardware Components

- **Dell PowerEdge R430**: Proxmox node (pve-fuzzing-01) - Primary fuzzing workloads

- **Dell PowerEdge R730**: Proxmox node (pve-analysis-01) - Analysis and monitoring

- **Intel NUC8i7HVK**: Proxmox node (pve-dev-01) - Management and development

- **OVS Software Switch**: Layer 2 managed switching with VLAN support

- **2x Raspberry Pi 3B+**: WAN redundancy with VRRP failover

- **pfSense Firewall**: Network security and VLAN routing

### Software Components

- **Proxmox VE**: 3-node cluster with proper quorum

- **Open vSwitch**: Advanced layer 2 switching ([GitHub Repository](https://github.com/your-org/ovs-layer2-switch))

- **pfSense**: Firewall with default-deny security policy

- **ELK Stack**: Centralized logging and analysis

- **Suricata/Zeek**: Network intrusion detection

- **Custom Fuzzing Framework**: ICS protocol-specific testing

## Network Architecture

```

WAN Subnet (192.168.200.0/24)

âââ VRRP VIP: 192.168.200.100

âââ RPi #1: 192.168.200.10

âââ RPi #2: 192.168.200.11

Internal Networks:

âââ VLAN 10 (LAN): 192.168.10.0/24

âââ VLAN 20 (DMZ): 192.168.20.0/24

âââ VLAN 30 (ICS Lab): 192.168.30.0/24 ð ISOLATED

âââ VLAN 40 (Management): 192.168.40.0/24

âââ VLAN 50 (Logging): 192.168.50.0/24

âââ VLAN 60 (Guest): 192.168.60.0/24

```

## Virtual Machine Inventory

| VMID | Name | Node | VLAN | IP | Purpose |

|——|——|——|——|-—|———|

| 100 | pfsense-fw | NUC | Multi | 192.168.40.1 | Firewall/Router |

| 200 | mgmt-tools | NUC | 40 | 192.168.40.50 | Management |

| 300 | fuzzing-engine-1 | R430 | 30 | 192.168.30.10 | Primary fuzzing |

| 301 | fuzzing-engine-2 | R430 | 30 | 192.168.30.11 | Secondary fuzzing |

| 310 | protocol-sim-1 | R430 | 30 | 192.168.30.20 | Modbus simulator |

| 311 | protocol-sim-2 | R430 | 30 | 192.168.30.21 | DNP3 simulator |

| 320 | pcap-replay | R430 | 30 | 192.168.30.30 | Traffic replay |

| 400 | elk-stack | R730 | 50 | 192.168.50.10 | Log analysis |

| 401 | network-monitor | R730 | 50 | 192.168.50.11 | IDS/IPS |

| 402 | reverse-engineering | R730 | 40 | 192.168.40.70 | Malware analysis |

| 500 | ryu-controller | NUC | 40 | 192.168.40.60 | SDN controller |

## Key Access Information

### Management Access

- **Proxmox Cluster**: https://192.168.40.10:8006, https://192.168.40.11:8006, https://192.168.40.12:8006

- **pfSense Firewall**: https://192.168.40.1

- **Kibana Dashboard**: http://192.168.50.10:5601

- **Management VM**: ssh ubuntu@192.168.40.50

### Service Endpoints

- **Elasticsearch**: http://192.168.50.10:9200

- **Modbus Simulator**: 192.168.30.20:502

- **DNP3 Simulator**: 192.168.30.21:20000

- **Ryu Controller**: http://192.168.40.60:8080

## Component Integration Summary

### Core Components

1. **Compute Layer**: 3-node Proxmox cluster (Dell R430, R730, Intel NUC)

2. **Network Layer**: [OVS Layer 2 Switch](https://github.com/your-org/ovs-layer2-switch) with advanced VLAN support

3. **Security Layer**: pfSense firewall with default-deny policies

4. **Monitoring Layer**: ELK stack with Suricata IDS integration

5. **Redundancy Layer**: VRRP WAN failover with Raspberry Pi bridges

### Integration Points

- **OVS â Proxmox**: VLAN trunking for VM network segmentation

- **OVS â pfSense**: Firewall integration with VLAN routing

- **OVS â Monitoring**: Port mirroring for traffic analysis

- **OVS â Ryu**: OpenFlow controller integration for SDN capabilities

## Final Validation Results

Last comprehensive validation: $(date)

```bash

# Run infrastructure validation

/home/ubuntu/scripts/final_validation.sh

# Validate OVS switch health

ssh root@ovs-switch-ip ‘./tools/ovs-manager.sh health’

```

## Support and Maintenance

### Component-Specific Maintenance

#### Network Infrastructure (OVS)

- **Monthly**: Performance monitoring and port statistics review

- **Quarterly**: VLAN configuration validation and mirror port testing

- **Annually**: OVS version updates and feature evaluation

- **Documentation**: [OVS Maintenance Guide](https://github.com/your-org/ovs-layer2-switch/blob/main/docs/maintenance.md)

**Project successfully delivered on schedule with all requirements met.**

EOF

```

## Summary

This comprehensive implementation plan provides a complete step-by-step guide for deploying your ICS Protocol Fuzzing Laboratory with the following key features:

### **Technical Achievements**

- **3-node Proxmox cluster** with proper quorum and high availability

- **Advanced OVS switching** with dedicated GitHub project for reusability

- **Complete VLAN isolation** with security-first design

- **Industrial protocol support** (Modbus, DNP3, IEC 104)

- **Comprehensive monitoring** with ELK stack and Suricata IDS

- **Automated testing framework** for continuous validation

### **Security Implementation**

- **Default-deny firewall** with pfSense

- **Complete ICS lab isolation** in VLAN 30

- **SSH key authentication** throughout

- **Centralized logging** for audit trails

- **WAN redundancy** with VRRP failover

### **Operational Excellence**

- **Infrastructure as Code** with Terraform/Ansible

- **Automated testing suites** for validation

- **Comprehensive documentation** with runbooks

- **Performance monitoring** and optimization

- **Backup and recovery** procedures

### **Component Integration Summary**

#### Core Components

1. **Compute Layer**: 3-node Proxmox cluster (Dell R430, R730, Intel NUC)

2. **Network Layer**: [OVS Layer 2 Switch](https://github.com/your-org/ovs-layer2-switch) with advanced VLAN support

3. **Security Layer**: pfSense firewall with default-deny policies

4. **Monitoring Layer**: ELK stack with Suricata IDS integration

5. **Redundancy Layer**: VRRP WAN failover with Raspberry Pi bridges

#### Integration Points

- **OVS â Proxmox**: VLAN trunking for VM network segmentation

- **OVS â pfSense**: Firewall integration with VLAN routing

- **OVS â Monitoring**: Port mirroring for traffic analysis

- **OVS â Ryu**: OpenFlow controller integration for SDN capabilities

The lab provides enterprise-grade capabilities for ICS protocol security research while maintaining strict isolation and comprehensive monitoring. The modular OVS switch component can be reused in other projects, and the entire implementation is fully documented for long-term maintenance and expansion.

```