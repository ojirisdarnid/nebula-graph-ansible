
# Nebula Graph Deployment Automation with Ansible

This repository contains an Ansible playbook designed to automate the deployment of Nebula Graph on Ubuntu 20.04. By using this playbook, you can achieve faster, consistent, and error-free installations while minimizing the risks of human error. The flexible structure of the playbook allows for easy customization to suit the specific requirements of your environment.

## 📌 Key Features

* **Fast Installation**, Automates the entire process, saving time and ensuring smooth deployments.
* **Consistency**, Delivers the same configuration for every installation, eliminating discrepancies.
* **Error Minimization**, Reduces the risk of misconfigurations caused by manual steps.

## 🛠️ Responsibilities of the Playbook
This playbook handles the following tasks:

* **Install Dependencies**, Ensures all required dependencies for Nebula Graph are installed on the system.

* **Configure the System for Nebula Graph**, Prepares the system environment, including system settings and optimizations kernel for Nebula Graph.

* **Download & Install Nebula Graph**, Automates the downloading and installation of the Nebula Graph binary.

* **Backup Nebula Graph Configuration Files**, Creates backups of existing configuration files to ensure recovery options.

* **Update Nebula Graph Configuration**, Updates the configuration files to meet your deployment requirements.

* **Start Nebula Graph & Download Nebula Console**, Starts the Nebula Graph service and downloads the Nebula Console for easy database interaction.

* **Install Nebula Graph Studio**, Automates the installation of Nebula Graph Studio, a web-based UI tool for managing and interacting with Nebula Graph.
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
├── 📄 ansible.cfg                                 # Ansible configuration file
├── 📁 group_vars/                                 # Group variable files
│    └── 📄 all                                    # Variables applied to all hosts
├── 📄 hosts                                       # Inventory file for target nodes
├── 📁 roles/
│    └── 📁 nebula/                                # Role for Nebula Graph installation
│        ├── 📁 tasks/ 
│        │    ├── 📄 01install_dependency.yaml     # Install dependencies
│        │    ├── 📄 02setup_server.yaml           # Configure server
│        │    ├── 📄 03install_nebula.yaml         # Install Nebula Graph
│        │    ├── 📄 04backup_nebula_config.yaml   # Backup Nebula configuration
│        │    ├── 📄 05update_nebula_config.yaml   # Update Nebula configuration
│        │    ├── 📄 06start_nebula.yaml           # Start Nebula services
│        │    └── 📄 main.yaml                     # Main task file to include all steps
│        📁 nebula_studio/                         # Role for Nebula Graph Studio installation
│        └── 📁 tasks/
│             ├── 📄 01install_nebula_studio.yaml  # Install Nebula Graph Studio
│             └── 📄 main.yaml                     # Main task file to include all steps
├── 📄 nebula_playbook.yaml                        # Main playbook to run the tasks
└── 📄 README.md                                   # Project documentation

```
## 📜 Playbook Overview
This playbook automates the installation and configuration of Nebula Graph and Nebula Graph Studio. You can customize which tasks to run by adjusting the variables.

```yaml
- name: Install Nebula Graph
  hosts: nebula
  roles:
    - nebula

# Set these variables to true to run specific tasks
  vars:
    install_dependency: false
    setup_server: false
    install_nebula: false
    backup_nebula_config: false
    update_nebula_config: true
    start_nebula: false

- name: Install Nebula Graph Studio
  hosts: nebula_studio
  roles:
    - nebula_studio

# Set these variables to true to run specific tasks
  vars:
    install_nebula_studio: false

```

Simply set the task-related variables to `true` to run specific tasks (e.g., install dependencies, update configurations).
## 📋 Environment Variables

This playbook uses a hosts file and group_vars to manage environment-specific variables. Below is the guide on how to use these configurations effectively, modify the inventory and configuration variables to suit your environment. 

### 🗂️ Group Variables (`group_vars`)
The `group_vars` directory contains configuration variables that are applied to groups of hosts defined in your inventory. These variables are critical for tailoring the deployment and tuning the environment for optimal Nebula Graph performance.

Group Variables in this Repository (`group_vars/all`):
```yaml
## Version Vars
nebula_version: 3.8.0
studio_version: 3.10.0

## Nebula Config Vars
nebula_timezone: 'UTC+07:00'                           # Time zone for the Nebula Graph deployment.
nebula_install_path: '/usr/local/nebula'               # Installation directory for Nebula Graph.
nebula_meta_server_addrs: '[node1_ip]:9559,[node2_ip]:9559,[node3_ip]:9559'  # List of Nebula Meta servers in the cluster.

nebula_auth: '--enable_authorize=true'                 # Enables authentication for Nebula Graph. Set to "true" to require login credentials for access.
nebula_maxjobsize: '--max_job_size=16'                 # Specifies the maximum size for job management tasks in Nebula Graph. (Default value; tweak based on workload.)
nebula_listen_backlog: '--listen_backlog=65535'        # Maximum number of pending connection requests allowed on listening sockets. Optimized for high concurrency.
nebula_heartbeat: '--heartbeat_interval_secs=3'        # Interval (in seconds) at which the Nebula storage service sends heartbeat signals to the meta service.

metad_datapath: '--data_path=/data/nebula/meta'        # Path for storing metadata in the Meta service.
storaged_datapath: '--data_path=/data/nebula/storage'  # Path for storing graph data in the Storage service.

## sysctl.conf Vars
kernel_config:
- 'fs.inotify.max_user_watches=5242880'     # Increases the maximum number of file watches for inotify.
- 'vm.swappiness=0'                         # Reduces swapping to improve performance.
- 'vm.min_free_kbytes=20971520'             # Ensures enough memory is reserved for the system.
- 'vm.max_map_count=1048576'                # Increases the maximum number of memory map areas a process may use.
- 'net.ipv4.tcp_slow_start_after_idle=0'    # Prevents TCP slow start after idle to optimize network performance.
- 'net.core.somaxconn=65535'                # Sets the maximum number of connections in the backlog queue.
- 'net.ipv4.tcp_max_syn_backlog=65535'      # Increases the maximum SYN backlog to handle high connection rates.
- 'net.core.netdev_max_backlog=1048576'     # Increases the maximum number of packets queued in the backlog before processing.
- 'net.ipv4.tcp_rmem=4096 87380 16777216'   # Configures TCP read buffer sizes.
- 'net.ipv4.tcp_wmem=4096 65535 16777216'   # Configures TCP write buffer sizes.
- 'fs.file-max=9223372036854775807'         # Sets the maximum number of open files allowed system-wide.

## limits.conf Vars
limits_config:
- 'root           soft         core           unlimited'    # No limits on core file size.
- 'root           hard         core           unlimited'
- 'root           soft         nproc          65535'        # Limits for the maximum number of processes.
- 'root           hard         nproc          65535'
- 'root           soft         nofile         65535'        # Maximum number of open files.
- 'root           hard         nofile         65535'

## grub Vars
grup_cmdline: 'GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"'         # Default GRUB command line options for booting the Linux kernel.
grup_cmdline2: 'GRUB_CMDLINE_LINUX="transparent_hugepage=never"'  # Disables transparent huge pages, which can cause performance issues for database workloads.


```
### 🔧 Customization Tips:

* #### Defaults Are Ready to Use
The default values for variables such as `limits_config`, `kernel_config`, `nebula_version`, and `nebula_timezone` are optimized for most environments. You only need to customize them if your setup has specific requirements.

* #### Update Meta Server Addresses
Ensure the `nebula_meta_server_addrs` variable reflects the IP addresses of your Nebula Graph cluster nodes. Accurate configuration here is crucial for proper communication within the cluster.

* #### Path and Resource Adjustments
Modify paths like `nebula_install_path`,` metad_datapath`, and `storaged_datapath` to match your directory structure if needed. Fine-tune `kernel_config` and `limits_config` based on your server's workload and resource capacity for optimal system performance.

* #### Enable Authentication (`nebula_auth`)
Set `--enable_authorize=true` to enforce user authentication for Nebula Graph. This ensures secure access with credential verification.

* #### Control Job Sizes (`nebula_maxjobsize`)
Defines the maximum size of job tasks, such as compaction and balancing operations. Adjust this value to fit the scale of your cluster and workload requirements.

* #### Manage Client Connections (`nebula_listen_backlog`)
Sets the maximum number of pending connections allowed on the server’s listening sockets. Higher values are suitable for high-concurrency environments to handle more simultaneous client connections.

* #### Heartbeat Frequency (`nebula_heartbeat`)
Determines how often storage nodes send heartbeat signals to meta nodes, in seconds. Tuning this helps balance between responsiveness and network overhead, depending on your cluster's size.

* #### Optimize Boot Performance (`grup_cmdline` and `grup_cmdline2`)
`grup_cmdline: ~` Configures default boot parameters to enhance system stability and startup performance.  
`grup_cmdline2: ~` Disables transparent huge pages, improving database performance and preventing memory fragmentation issues

**💡 Pro Tip**, Carefully review the variable settings to ensure they align with your system and deployment requirements. These configurations significantly impact the performance and stability of Nebula Graph in production environments.

Example Updated `group_vars/all`:
```yaml
## Version Vars
~

## Nebula Config Vars
nebula_timezone: 'UTC+07:00'
nebula_install_path: '/usr/local/nebula'
nebula_meta_server_addrs: "192.168.1.10:9559,192.168.1.11:9559,192.168.1.12:9559"
nebula_auth: '--enable_authorize=true'
nebula_maxjobsize: '--max_job_size=16'
nebula_listen_backlog: '--listen_backlog=65535'
nebula_heartbeat: '-heartbeat_interval_secs=3'
metad_datapath: '--data_path=/data/nebula/meta'
storaged_datapath: '--data_path=/data/nebula/storage'

## sysctl.conf Vars
~
## limits.conf Vars
~
## grub Vars
~
```

### 🗂️ Inventory Configuration (`hosts`)  
The inventory file specifies the hosts and their groupings, along with relevant connection parameters for Ansible.  
Inventory in this repo (`inventory`):

```yaml
[all:vars]
ansible_user=root
ansible_ssh_pass=your_root_password_here

[nebula]
node_ip1 host_name='your.host.name1' ansible_host=node_ip1
node_ip2 host_name='your.host.name2' ansible_host=node_ip2
node_ip3 host_name='your.host.name3' ansible_host=node_ip3

[nebula_studio]
node_ip
```

Update the following placeholders in your inventory file:  

* Replace `node_ip1`, `node_ip2`, and `node_ip3` with the IP addresses of the machines running Nebula Graph.  
* Replace `your.host.name1`, `your.host.name2`, and `your.host.name3` with the actual hostnames of those machines.  
* Replace `your_root_password_here` with the SSH password for the root user on your target machines.  
* If deploying Nebula Studio, include the IP addresses or hostnames under the [`nebula_studio`] group 

Example Updated `hosts`: 
```yaml
[all:vars]
ansible_user=root
ansible_ssh_pass=johndoe123456

[nebula]
192.168.1.10 host_name='nebula-node1' ansible_host=192.168.1.10
192.168.1.11 host_name='nebula-node2' ansible_host=192.168.1.11
192.168.1.12 host_name='nebula-node3' ansible_host=192.168.1.12

[nebula_studio]
192.168.1.13
```
## 🚀 How to Use

Before running the playbook, test connectivity to all nodes.  
Use the following command to ensure Ansible can communicate with all nodes in the hosts inventory file:  

```bash
ansible all -m ping -i inventory
```
A successful response (e.g., "`pong`") indicates that Ansible can connect to all target nodes.  

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
```
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
## 📖 Official Documentation

[Nebula Graph Documentation](https://docs.nebula-graph.io/3.8.0/)  
[Ansible Documentation](https://docs.ansible.com/)
## Authors

- [@ojirisdarnid](https://www.github.com/ojirisdarnid)
- [@jordiirandi](https://www.instagram.com/jordiirsandi/)
