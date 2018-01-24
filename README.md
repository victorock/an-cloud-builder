Ansible Networking Demo Kit
=========

This is a set of playbooks used for provisioning ansible networking demos in clouds.  It takes a Blueprint and creates that architecture in the cloud provider specified by that blueprint.

Requirements
------------

These playbooks require that you have your cloud environment setup correctly for authentication.  To pull down the repository:

```
git clone git@github.com:ismc/an-demo-kit.git
```

Examples
--------

To build the cloud given the model:

```
ansible-playbook -e @model/csr-lab1.yml build-demo.yml
```

To destroy the cloud given the blueprint:

```
ansible-playbook -e @model/csr-lab1.yml destroy-demo.yml
```

To configure the control node (install Ansible, setup Ansible Inventory, etc)

```
ansible-playbook -e @models/csr-lab1.yml configure-control.yml
```

You should see the IP address of the control node on successful completion of this playbook:

TASK [debug] ************************************************************************************
ok: [control] => {
    "msg": "Control node public IP':' 34.235.93.223"
}

At this point, you can ssh to labuser@<control IP address>

## Scenarios
- csr-lab1: Two router setup: 2 x Cisco CSR (IOS)
- multi-lab1: Two router setup: Cisco CSR (IOS) and Juniper MX
- [multi-lab2](scenarios/multi-lab2/README.md): A Palo Alto firewall and a F5 Big-IP

## Inventory

In cloud environments, it is best to pull inventory dynamically.  For the purposes of this, however, we create a static inventory on the control node as follows:

```
[all]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
control ansible_host=34.193.197.125 ansible_network_os=none ansible_ssh_user=ec2-user
host1 ansible_host=172.18.2.38 ansible_network_os=none ansible_ssh_user=ec2-user
[network]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
[routers]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
[ungrouped]
[hosts]
control ansible_host=34.193.197.125 ansible_network_os=none ansible_ssh_user=ec2-user
host1 ansible_host=172.18.2.38 ansible_network_os=none ansible_ssh_user=ec2-user
```

The inventory items are grouped function and are stored in the `hosts` file in the inventory directory:

```
.
├── inventory/                  # Inventory directory
│   │
│   ├── host_vars/              # device specific vars files
│   │   ├── rtr1
│   │   │   ├── interfaces.yml  # Device specific interface config
│   │   │   └── ospf.yml        # Device specific OSPF config
│   │   └── switch1
│   │       ├── interfaces.yml  # Device specific interface config
│   │       └── vlans.yml       # Device specific vlan config
│   │
│   ├── group_vars/             # group specific vars files
│   │   ├── all.yml             # Global config   
│   │   ├── network.yml         # Network specific
│   │   └── firewall.yml        # Function specific
│   │   
│   └── hosts         # Contains only the hosts in the dev environment
└── . . .
```

## Playbook Variables

Certain vars files are also created and pre-populated with key/value pairs that represent the Source of Truth for the network.  For example, an `interfaces.yml` file is create in `inventory\host_vars\{{ invetnory_hostname }}` with the interface information for that host:

```yaml
interfaces:
  - name: GigabitEthernet1
    ip: 172.17.0.254/24
    public_ip_address: 107.23.75.127
    public_dns_name: ec2-107-23-75-127.compute-1.amazonaws.com
  - name: GigabitEthernet2
    ip: 172.17.1.254/24
    public_ip_address: 34.192.63.70
    public_dns_name: ec2-34-192-63-70.compute-1.amazonaws.com
  - name: GigabitEthernet3
    ip: 172.17.2.254/24
    public_ip_address:
    public_dns_name:
```


## Running the example playbooks
In order to run the example playbooks, create an inventory directory using
`inventory/example` as a prototype.  The playbooks can then be run with the
command:

```
ansible-playbook -i inventory network/network-facts.yml
```

`-i inventory/example` tells ansible-playbook where to look for the inventory.


License
-------

GPL-3

---
![Ansible Red Hat Engine](ansible-engine-small.png)

In addition to open source Ansible, there is Red Hat® Ansible® Engine which includes support and an SLA for the nxos_facts module shown above.

Red Hat® Ansible® Engine is a fully supported product built on the simple, powerful and agentless foundation capabilities derived from the Ansible project.  Please visit [ansible.com](https://www.ansible.com/ansible-engine) for more information.
