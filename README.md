<p align="center"> <img src="https://user-images.githubusercontent.com/34257858/129839002-15e3f2c7-3f75-46d4-afae-0fd207d7fdde.png" width="100" height="100"></p>

<h1 align="center">
    Ansible Basic Dynamics Inventory Server
</h1>

<p align="center" style="font-size: 1.2rem;">
    It is a tool designed with automation in mind from the start.
</p>

<p align="center">

<a href="https://www.ansible.com">
  <img src="https://img.shields.io/badge/Ansible-2.10-red?style=flat&logo=ansible" alt="Ansible">
</a>
<a href="LICENSE.md">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="Licence">
</a>
<a href="https://ubuntu.com/">
  <img src="https://img.shields.io/badge/ubuntu-20.x-orange?style=flat&logo=ubuntu" alt="Distribution">
</a>
<a href="https://ubuntu.com/">
  <img src="https://img.shields.io/badge/ubuntu-22.x-orange?style=flat&logo=ubuntu" alt="Distribution">
</a>
<a href="https://www.centos.org/">
  <img src="https://img.shields.io/badge/CentOS-7-green?style=flat&logo=centos" alt="Distribution">
</a>
<a href="https://rockylinux.org/">
  <img src="https://img.shields.io/badge/RockyLinux-9-blue?style=flat&logo=rockylinux" alt="Distribution">
</a>
<a href="https://aws.amazon.com/amazon-linux-2/">
  <img src="https://img.shields.io/badge/AmazonLinux-2-orange?style=flat&logo=amazon" alt="Distribution">
</a>

> Minimal **Ansible** >= 2.10

# Vagrant Setup (Vagrantfile) for Testing

Vagrant plugin installed on your host machine:

-   Vagrant Host Manager [Doc](https://github.com/devopsgroup-io/vagrant-hostmanager)
-   Vagrant DNS [Doc](https://github.com/BerlinVagrant/vagrant-dns)

# Directory Structure

Here is the basic suggested skeleton for your ansible repo that each of the starter templates conforms to:

```bash
├── ansible
│   ├── collections                 # Ansible collections
│   ├── facts                       # Ansible facts
│   ├── roles                       # Ansible roles
├── inventories                     # Inventories environment
│   ├── development
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   │   ├── main.yml        # Main group vars
│   │   │   │   ├── vault.yml       # Ansible vault
│   │   ├── host_vars
│   │   ├── hosts
│   ├── production
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   │   ├── main.yml        # Main group vars
│   │   │   │   ├── vault.yml       # Ansible vault
│   │   ├── host_vars
│   │   ├── hosts
│   ├── staging
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   │   ├── main.yml        # Main group vars
│   │   │   │   ├── vault.yml       # Ansible vault
│   │   ├── host_vars
│   │   ├── hosts
│   ├── testing
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   │   ├── main.yml        # Main group vars
│   │   │   │   ├── vault.yml       # Ansible vault
│   │   ├── host_vars
│   │   ├── hosts
├── playbooks                       # Setup Ansible Playbook
├── vars                            # Setup variables
├── .ansible-lint
├── .editorconfig
├── .gitignore
├── .yamllint
├── ansible.cfg
├── README.md
├── requirements.yml                # Collections & Roles
├── ping.yml                        # Test use Ping module each server
├── playbook.yml                    # Playbook import setup from "playbooks" directory
└── Vagrantfile
```

## Inventories

Inventory with multiple environment setup:

-   Development Lab (Vagrant)
-   Testing (Cloud & On Premise)
-   Staging
-   Production

## Ansible Configuration (custom ansible.cfg)

```ini
[defaults]
inventory = ./inventories/development
collections_path = ./ansible/collections
roles_path = ./ansible/roles
private_key_file = $HOME/.ssh/id_rsa
vault_password_file = $HOME/.config/.vault_pass
remote_user = root
retry_files_enabled = False
fork = 30
gathering = smart
gather_subset = all
gather_timeout = 60
local_tmp = $HOME/.ansible/tmp
# Caching
fact_caching = jsonfile
fact_caching_timeout = 3600
fact_caching_connection = ./ansible/facts

[galaxy]
role_skeleton_ignore = ^.git$, ^.*/.git_keep$

[diff]
always = True

[privilege_escalation]
become = True
become_ask_pass = False
become_method = sudo
become_user = root

[ssh_connection]
pipelining = True
scp_if_ssh = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o PreferredAuthentications=publickey
```

## Ansible Configuration for support ssh bastion/jump (ansible.cfg)

```ini
[netconf_connection]
# (string) This variable is used to enable bastion/jump host with netconf connection. If set to True the bastion/jump host ssh settings should be present in ~/.ssh/config file, alternatively it can be set to custom ssh configuration file path to read the bastion/jump host settings.
ssh_config=./ssh.cfg

[ssh_connection]
pipelining=True
ssh_args=-F ./ssh.cfg
control_path=~/.ssh/mux-%r@%h:%p
```

## Dynamics Inventories

-   development
-   testing
-   staging
-   production

## Ansible Galaxy (install collection/role):

```bash
ansible-galaxy install -r ./requirements.yml

OR

ansible-galaxy install -r ./requirements.yml --force
```

## AdHoc ansible command for check host response:

```bash
ansible -i inventories/{inventory_name} all -m ping
```

## Running playbook

Running playbook

### Running playbook with multiple environment variables

-   `server` _(server group on `host` file inventory)_

#### Ping server:

```bash
ansible-playbook -i inventories/{inventory_name} ping.yml --extra-vars "server=frontend"
```

#### Running Playbook `(per group or per host)`:

```bash
ansible-playbook -i inventories/{inventory_name} playbook.yml --extra-vars "server=frontend"

or

ansible-playbook -i inventories/{inventory_name} playbook.yml --extra-vars "server=frontend_1"
```

##### With `limit` host group and `tags` name:

```bash
ansible-playbook -i inventories/{inventory_name} playbook.yml --extra-vars "server=frontend" --tags "init,update"

OR

ansible-playbook -i inventories/{inventory_name} playbook.yml -e "server=frontend" -t "init,update"
```

#### With extra vars:

```bash
ansible-playbook playbook.yml -e '@vars_{{ environment }}.json'
```

## CLI password generator for user authentication

> Install `mkpasswd` on OS

Command generate password:

```bash
mkpasswd -m sha-512 <password> -S <salt>
```

Random `salt` generate from [RANDOM.ORG](https://www.random.org/strings/)

## License

MIT / BSD

## Author Information

This Ansible IAC was created in 2022 by [Asapdotid](https://github.com/asapdotid).
