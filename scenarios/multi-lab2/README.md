Scenario: multi-lab2
=========

Topology
------------

![multi-lab2 topology](multi-lab2-topo.png)

Scenario
--------

This topology presents a simplified data center construct with an inside network, an outside network, and a management network.  Two hosts are placed in the inside network that present web services.  Two VNFs are also placed to provides firewall and load balancing services, a Palo Alto firewall and am F5 Big-IP.  The Palo Alto firewall provides outbound NAT services to the inside network.  The F5 load balances incoming connections to port 80 on its outside port to port 80 on host1 an host2

All of the playbooks are assumed to be run from the an-demo-kit directory in the 'labuser' directory.

The Palo Alto modules require setting the password from the ssh key provided to the cloud provider:
```
ansible-playbook -i inventory pan/pan-passwd.yml
```

Configure the interfaces on PAN to pass traffic:
```
ansible-playbook -i inventory pan/pan-system.yml
```

Add the rule to NAT all outgoing connections:
```
ansible-playbook -i inventory pan/pan-nat.yml
```

Add the security rule to allow all outgoing connections:
```
ansible-playbook -i inventory pan/pan-security-rules.yml
```

At this point, you should be able to log into host1 and ping outside the setup.

Once the app hosts can reach outside, the web servers can be configured:
```
ansible-playbook -i inventory common/configure-webservers.yml
```

The next step is to configure the F5.  As with the PAN modules, the F5 modules require a password to be set to use the API.  To do that, run:
```
ansible-playbook -i inventory f5/f5-passwd.yml
```

Set the hostname, timezone, create the VLANs, and configure the interfaces:
```
ansible-playbook -i inventory f5/f5-system.yml
```

Finally, setup pool, add the hosts in the app_hosts group, and create the vserver:
```
ansible-playbook -i inventory f5/f5-vserver.yml
```

At this point, you should be able to log into the vserver VIP.  You can find that IP from the return of the previous playbook:

TASK [debug] *******************************************************************************
ok: [f5] => {
    "msg": "The Virtual Server Address is 34.197.114.76"
}

License
-------

GPL-3
