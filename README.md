# Ansible Collection - maveno_de.forks

Version 0.1.0 created by Javanaut 2022

This is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.


## NOTE

This is pretty experimental stuff currently. Better read before trying out :)


## Features

This collection provides a toolset for setup, configuration and management of a software defined cluster of nodes for the Chia Network Blockchain and compatible forks. The cluster may have a flexible structure concerning number and type of forks and nodes. Separate and remote operation of fullnodes, harvesters, farmers and wallets is possible.

## Requirements and prerequisites

This collection is designed to manage nodes running Debian type OS, e.g. Debian, Ubuntu, Raspberry Pi OS.

NOTE: Root access or access to a system user with sudo privileges preferable via SSH ist required on all nodes.

Hardware suitable to run nodes are servers along with common PC or laptops as well as ARM based smallscale systems like RPi4 (>= 4GB RAM). Farmer- and/or harvester-only nodes are running in multiple instances on a RPI3 easily.

NOTE: Several forks are not updated to be compatible with Python 3.10. A Python 3.9 alternative is required to be available on the hosts. The following steps perform the installtion:

Download and unpack latest [Python 3.9 sources](https://www.python.org/downloads/source/)

```bash
sudo su
apt-get update
apt-get install openssl python3-distutils build-essential libncurses*-dev liblzma-dev libgdbm-dev libsqlite3-dev libbz2-dev tk-dev libreadline-dev libgdbm-compat-dev libssl-dev libffi-dev

cd <unpacked Python 3.9 source directory>
./configure --enable-loadable-sqlite-extensions --enable-optimizations
make
make altinstall
```

Role maveno_de.forks.node provides modified installer scripts to work with python 3.9.

## Quick Start

The following examples provide insight on configuration and usage of the collection. It is adviced to create a project directory and keep it under version control in a private repository, e.g. on github. Specific files to configure a forks cluster are given relative to an assumed \<project directory\> in the following. As Ansible has a complex path referencing system, changing these advices pathes may cause issues when executing playbooks.


## Example inventory file

Location: \<project directory\>/inventory/forks.yml

```yaml
forks:

  hosts:
    hostname1:
      ansible_host: '<domainname 1>'
      forksBackupDataDirectory: /path/to/blockchain/db/backup/directory
    hostname2:
      ansible_host: '<domainname 2>'
    hostname3:
      ansible_host: '<domainname 3>'
    hostname4:
      ansible_host: '<domainname 4>'
  vars:

    ansible_user: "{{ forksManagingSystemUsername|default('root') }}"

    forksNodesConfiguration:

      - identifier: '<node_id 1>'
        host: hostname1
        fork: '<fork_id>'
        services:
          - node
          - harvester
        #username: <node username>
        #serviceName: <node service name>
        config:
          harvester:
            farmer_peer:
              host: '<domainname 2>'
            plot_directories: >-
              {{ forksHostPlotDirectories['hostname1']['chia']|default([]) }}

      - identifier: '<node_id 2>'
        host: hostname2
        fork: '<fork_id>'
        services:
          - farmer-only
        #username: <node username>
        #serviceName: <node service name>
        config:
          harvester:
            farmer_peer:
              host: '<domainname 2>'
        certs: '<node_id 1>'
        tunnels:
          - target: '<node_id 1>'
            ports:
              - '<Tunnel port 1>'
              - '<Tunnel port 2>'

      - identifier: '<node_id 3>'
        host: hostname3
        fork: '<fork_id>'
        services:
          - wallet
          - harvester
        #username: <node username>
        #serviceName: <node service name>
        config:
          wallet:
            full_node_peer:
            host: '<domainname 1>'
          harvester:
            farmer_peer:
              host: '<domainname 2>'
            plot_directories: >-
              {{ forksHostPlotDirectories['hostname3']['chia']|default([]) }}
        certs: '<node_id 1>'

      - identifier: '<node_id 4>'
        host: hostname4
        fork: '<fork_id>'
        services:
          - harvester
        #username: <node username>
        #serviceName: <node service name>
        config:
          harvester:
            farmer_peer:
              host: '<domainname 2>'
            plot_directories: >-
              {{ forksHostPlotDirectories['hostname4']['chia']|default([]) }}
        certs: '<node_id 1>'
```

## Example group variables

Location: \<project directory\>/inventory/group_vars/forks.yml

```yaml
forksVendorIdentifier: forks
forksManagingSystemUsername: forks

forksManagingHomeDirectory: >-
  {{ forksUsersRootDirectory }}/{{
  forksManagingSystemUsername|default('root') }}

forksSettingsLookupTable:
  chia:
    backupHour: 3
    backupMinute: 0
    updateHour: 4
    updateMinute: 0
  chives:
    backupHour: 4
    backupMinute: 30
    updateHour: 5
    updateMinute: 30

forksHostPlotDirectories:
  hostname1:
    chia:
      - /path/to/chia/plots/1
      - /path/to/chia/plots/2
      - /path/to/chia/plots/3
    chives:
      - /path/to/chives/plots/1
      - /path/to/chives/plots/1
  hostname2:
    chia:
      - /path/to/chia/plots/1
    chives:
      - /path/to/chives/plots/1
  hostname3:
    chia:
      - /path/to/chia/plots/1
      - /path/to/chia/plots/2
```

## Example playbook

Location: \<project directory\>/playbooks/cluster.yml

```yaml
- name: Build Forks cluster
  hosts: forks
  tasks:

    - name: Build Forks cluster
      include_role:
        name: maveno_de.forks.node
      tags: always 
```

## Usage

### Cluster definition

The topology of a forks cluster is defined in variable forksNodesConfiguration that must be visible to all hosts. Proper locations are \<project directory\>/inventory/group_vars/forks.yml, \<project directory\>/inventory/group_vars/all.yml or the inventory file as shown in example above. It is organized as a numeric array of nodes of arbitrary fork type to be run on an arbitrary number of hosts.

#### Nodes

Each node is a list entry containing a dict that describes a chia or fork instance running under a dedicated daemon and system user on its assigned host of the cluster. The dicts keys define the properties of the corresponding node.

#### Mandatory node properties

| Key | Property | type | default |
| :--- | :----: | :----: | :----: |
| identifier | Used to reference nodes to each other, for example to exchange certificates | string | |
| host | Host to place the node on, refering to ansible inventory hostnames | string | |
| fork | Fork type to run on the node<br />HINT: Valid fork identifiers are defined in tasks file properties.yml in role maveno_de.forks.node | string | |
| services | Components of the fork instance to be run. This corresponds to the forks start subparameters, shown below. <br />NOTE: Start fullnode only via service <i>node</i> and add further services with the corresponding {service}-only option<br />Services are started in the given order. | list of strings. | |

#### Optional node properties

| Key | Property | type | default |
| :--- | :----: | :----: | :----: |
| username | System user to run the forks instance under | string | {identifier} |
| serviceName | Systemd service name to manage the forks instance | string | {identifier} |
| config | Subkeys of this variable will be merged into the nodes configuration | dict | |
| certs | Certificates will be created from the CA of the references node. This is required for the connections of harvesters to farmers and of farmers to fullnodes. | string | |
| tunnels | SSH tunnels are created and configured to the referenced target node with the given ports. <br />NOTE: SSH connections can be considered as very secure, the necessary keys are created automatically. <br />HINT: For a farmer working properly on a fullnode the fullnode's port and RPC port should be set here.  | list of dicts | |
| alias | Command aliases of the forks nodes executables that are available for the managing user | list of dicts | |

#### Start Options


|                   | Full Node | Wallet | Farmer | Harvester | Timelord | Timelord Launcher | Introducer | Full Node Simulator |
| :---              | :----:    | :----: | :----: | :----:    | :----:   | :----:            | :----:     | :----:              |
| all	              |     X     |    X   |    X   |  X        |        X |    X              |            |                     |
| node              |   X       |        |        |           |          |                   |            |                     |
| harvester         |           |        |        |      X    |          |                   |            |                     |
| farmer            |   X       |  X     |    X   |     X     |          |                   |            |                     |
| farmer-no-wallet  |    X      |        |   X    |   X       |          |                   |            |                     |
| farmer-only       |           |        |   X    |           |          |                   |            |                     |
| timelord          |   X       |        |        |           |        X |                X  |            |                     |
| timelord-only     |           |        |        |           | X        |                   |            |                     |
| timelord-launcher |           |        |        |           |          |               X   |            |                     |
| wallet            |   X       |   X    |        |           |          |                   |            |                     |
| wallet-only*       |           |  X     |        |           |          |                   |            |                     |
| introducer        |           |        |        |           |          |                   |    X       |                     |
| simulator         |           |        |        |           |          |                   |            | X                   |

\* <i>wallet-only</i> is now only available on some outdated forks for which no light wallet is available. Use <i>wallet</i> for current chia version and modern fork types.

### Managing system user

The role can be executed as root of the managed hosts or a specific user may be used, that is required to be able to perform passwordless sudo commands. <i>forksVendorIdentifier</i> may be defined that will be used as suffix for the used path prefixes. These variables are allowed to be defined on group level, host level or be omitted.

```yaml
forksVendorIdentifier: <Vendor Identifier>
forksManagingSystemUsername: <Managing Systemuser Username>
forksManagingHomeDirectory: <Managing Systemuser Home Directory>
```

### Example command lines

The complete run will install and/or upgrade all configured nodes and additional tools.

```bash
ansible-playbook -i inventory/forks.yml playbooks/cluster.yml
```

Run for selected nodes:

[More on limits](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)


```bash
ansible-playbook -i inventory/forks.yml --limit "node1,node2" playbooks/cluster.yml
```

Run only subtasks. This example will update all certificates on nodes in the inventory file. 

```bash
ansible-playbook -i inventory/forks.yml --tags certificates playbooks/cluster.yml
```

### Tags

[More on tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html)

Tags allow to define a specific subset of tasks that should be executed during a playbook run. This collection uses the tags described in the following table.

NOTE: The tasks in this collection are not designed to run multiple tags at once.

| Tag | Purpose |
| :--- | :----: |
| config | Apply the configuration modifications that are defined in <i>forksNodesConfiguration.[*].config</i> to each fork node |
| patches | Apply patches that are located in files/patches/<fork_id> to each fork node  |
| updates | Setup/update auto-updates to each fork node <br />NOTE: Run with tag patches before this |
| tunnels | Setup SSH tunnels according to definitions in <i>forksNodesConfiguration.[*].tunnels</i>  |
| logging | Setup logging to ramdisk for each fork node, also configure Promtail accordingly if detected* |
| certificates | Setup certificates for each fork node propery to enable communication between all nodes<br />NOTE: This will not work when nodes whose CA certs should be used are excluded with --limit |
| aliases | Recreate/update command aliases for nodes executables available for the managing user |
| backups\*\* | Setup/update scripts for automatic daily backup of the blockchain database  |

\* Promtail is detected by presence of its configuration file at location /etc/promtail/config.yml

\*\* <i>forksBackupDataDirectory</i> must be set for the host as well as <i>backupHour</i> and <i>backupMinute</i> in <i>forksSettingsLookupTable</i> for the corresponding fork type