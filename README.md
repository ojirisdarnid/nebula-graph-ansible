
# Nebula Graph Deployment Automation with Ansible

This repository contains an Ansible playbook designed to automate the deployment of Nebula Graph on Ubuntu 20.04. By using this playbook, you can achieve faster, consistent, and error-free installations while minimizing the risks of human error. The flexible structure of the playbook allows for easy customization to suit the specific requirements of your environment.

## 📌 Key Features

* Fast Installation: Automates the entire process, saving time and ensuring smooth deployments.
* Consistency: Delivers the same configuration for every installation, eliminating discrepancies.
* Error Minimization: Reduces the risk of misconfigurations caused by manual steps.

## 🛠️ Responsibilities of the Playbook
This playbook handles the following tasks:

* **Install Dependencies**, Ensures all required dependencies for Nebula Graph are installed on the system.

* **Configure the System for Nebula Graph**, Prepares the system environment, including system settings and optimizations kernel for Nebula Graph.

* **Download & Install Nebula Graph**, Automates the downloading and installation of the Nebula Graph binary.

* **Backup Nebula Graph Configuration Files**, Creates backups of existing configuration files to ensure recovery options.

* **Update Nebula Graph Configuration**, Updates the configuration files to meet your deployment requirements.

* **Start Nebula Graph & Download Nebula Console**, Starts the Nebula Graph service and downloads the Nebula Console for easy database interaction.

## 🔑 Prerequisites

Before running this playbook, ensure that Ansible is installed on your control node. Follow these steps:

Update System Packages
```bash
sudo apt update && sudo apt upgrade -y
```

Install Ansible  
Add the official Ansible PPA and install Ansible:
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

Verify Ansible Installation  
Confirm Ansible is installed by checking its version:
```bash
ansible --version
```
## 📁 Folder Structure

```bash
ansible-nebula-graph/
├── 📄 ansible.cfg                # Ansible configuration file
├── 📁 group_vars/                # Group variable files
│    └── 📄 all                    # Variables applied to all hosts
├── 📄 hosts                      # Inventory file for target nodes
├── 📁 roles/
│    └── 📁 nebula/                # Role for Nebula Graph installation
│        └── 📁 tasks/             # Task files for role
│             ├── 📄 01install_dependency.yaml     # Install dependencies
│             ├── 📄 02setup_server.yaml           # Configure server
│             ├── 📄 03install_nebula.yaml        # Install Nebula Graph
│             ├── 📄 04backup_nebula_config.yaml  # Backup Nebula configuration
│             ├── 📄 05update_nebula_config.yaml  # Update Nebula configuration
│             ├── 📄 06start_nebula.yaml          # Start Nebula services
│             └── 📄 main.yaml                    # Main task file to include all steps
├── 📄 nebula_playbook.yaml       # Main playbook to run the tasks
└── 📄 README.md                  # Project documentation

```
## 📋 Environment Variables

This playbook uses a hosts file and group_vars to manage environment-specific variables. Below is the guide on how to use these configurations effectively, modify the inventory and configuration variables to suit your environment. 

### 🗂️ Group Variables (`group_vars`)
The group_vars directory contains variables applicable to groups of hosts defined in your inventory. These variables are essential for configuring Nebula Graph and tuning the system for optimal performance.  
Group Variables in this repo (`group_vars/all`):
```bash
# Nebula Graph settings
nebula_version: 3.8.0
nebula_meta_server_addrs: "node_ip1:9559,node_ip2:9559,node_ip3:9559"
nebula_timezone: "UTC+07:00"
nebula_install_path: /usr/local/nebula

# Kernel configurations
kernel_config:
  - 'fs.inotify.max_user_watches=5242880'
  - 'vm.swappiness=0'
  - 'vm.min_free_kbytes=20971520'
  - 'vm.max_map_count=1048576'
  - 'net.ipv4.tcp_slow_start_after_idle=0'
  - 'net.core.somaxconn=65535'
  - 'net.ipv4.tcp_max_syn_backlog=65535'
  - 'net.core.netdev_max_backlog=1048576'
  - 'net.ipv4.tcp_rmem=4096 87380 16777216'
  - 'net.ipv4.tcp_wmem=4096 65535 16777216'
  - 'fs.file-max=9223372036854775807'

# System limits
limits_config:
  - '*              soft         core           unlimited'
  - '*              hard         core           unlimited'
  - '*              soft         nofile          130000'
  - '*              hard         nofile          130000'

```
You do not need to change the group variables unless the default values (e.g., `limits_config`, `kernel_config`, `nebula_version`, `nebula_timezone`, or `nebula_install_path`) differ from your requirements. However, double-check the `nebula_meta_server_addrs` variable to ensure it matches the IPs of your Nebula Graph cluster nodes.

Example Updated `group_vars/all`:
```bash
nebula_version: 3.8.0
nebula_meta_server_addrs: "192.168.1.10:9559,192.168.1.11:9559,192.168.1.12:9559"
nebula_timezone: "UTC+07:00"
nebula_install_path: /usr/local/nebula

# Kernel configurations
kernel_config:
~
# System limits:
~
```

### 🗂️ Inventory Configuration (`hosts`)  
The inventory file specifies the hosts and their groupings, along with relevant connection parameters for Ansible.  
Inventory in this repo (`inventory`):

```bash
[all:vars]
ansible_user=root
ansible_ssh_pass=your_root_password_here

[nebula]
node_ip1 host_name='your.host.name1' ansible_host=node_ip1
node_ip2 host_name='your.host.name2' ansible_host=node_ip2
node_ip3 host_name='your.host.name3' ansible_host=node_ip3
```

Update the following placeholders in your inventory file:  

**`node_ip1, node_ip2, node_ip3`**: Replace these with the IP addresses of your target nodes.  
**`your.host.name1, your.host.name2, your.host.name3`**: Replace these with the corresponding hostnames of the target nodes.  
**`your_root_password_here`**: Replace this with the SSH password for the root user on your target machines.

Example Updated `hosts`: 
```bash
[all:vars]
ansible_user=root
ansible_ssh_pass=johndoe123456

[nebula]
192.168.1.10 host_name='nebula-node1' ansible_host=192.168.1.10
192.168.1.11 host_name='nebula-node2' ansible_host=192.168.1.11
192.168.1.12 host_name='nebula-node3' ansible_host=192.168.1.12
```
## 🚀 How to Use

Before running the playbook, test connectivity to all nodes.  
Use the following command to ensure Ansible can communicate with all nodes in the hosts inventory file:  

```bash
ansible all -m ping -i inventory
```
A successful response (e.g., "pong") indicates that Ansible can connect to all target nodes.  

Run the playbook with:

```bash
  ansible-playbook -i hosts nebula_playbook.yaml
```


## 🔧 Post-Installation Setup

After deploying Nebula Graph and starting the service, you need to add the hosts to Nebula Graph before it can run properly.

Run the following command on one of the nodes,  replace `{nebula_hosts1}` with your IP:
```bash
./nebula-graph-console --addr {nebula_hosts1} --port 9669 -u root -p nebula
```
After logging into the console, run this command to add hosts to Nebula Graph:
```bash
ADD HOSTS {nebula_hosts1}:9779, {nebula_hosts2}:9779, {nebula_hosts3}:9779;
```
Replace `{nebula_hosts1}`, `{nebula_hosts2}`, and `{nebula_hosts3}` with the IPs or hostnames of your Nebula Graph cluster nodes. After that run this command to confirm if the hosts have been added successfully:
```bash
SHOW HOSTS;
```
The result will be like this:
```bash
+-------------------+---------------+--------------+--------------+----------------------+------------------------+---------------------+
| Host              | Port          | Status       | Leader count | Leader distribution  | Partition distribution | Version             |
+-------------------+---------------+--------------+--------------+----------------------+------------------------+---------------------+
| "{nebula_hosts1}" | 9779          | "ONLINE"     | 0            | "No valid partition" | "No valid partition"   | "3.8.0"             |
| "{nebula_hosts2}" | 9779          | "ONLINE"     | 0            | "No valid partition" | "No valid partition"   | "3.8.0"             |
| "{nebula_hosts3}" | 9779          | "ONLINE"     | 0            | "No valid partition" | "No valid partition"   | "3.8.0"             |
+-------------------+---------------+--------------+--------------+----------------------+------------------------+---------------------+
```

Notes:
- The output above shows that the nodes are online but there are no partitions or leaders yet. This is expected when starting with a fresh setup, as partitions will be initialized during the first use.
## 🔧 Customization
This playbook is designed to be flexible and modular. You can easily adjust tasks, roles, or variables to adapt to your specific deployment needs.
## 📖 Documentation

[Nebula Graph Documentation](https://docs.nebula-graph.io/3.8.0/)  
[Ansible Documentation](https://docs.ansible.com/)