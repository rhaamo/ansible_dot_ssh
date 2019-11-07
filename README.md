# dashie.dot_ssh

## Intro

Generates `~/.ssh/infra/<hostname>.cfg` files for your infra.

## Usage

playbook:
```
---

- hosts:
    - all
  roles:
      - { role: dashie.dot_ssh, become: false }
```

```
ansible-playbook playbooks/ssh_config_infra.yml
```

hosts.ini: defines your bastion if any
```
[bastion_xxx]
my_host.remote.tld

[bastion_yyy]
my_host2.local.tld
```

group_vars/bastion_xxx.yml:

```
dot_ssh_bastion: true
dot_ssh_bastion_fqdn: my.remote.host
```

group_vars/bastion_yyy.yml:
```
dot_ssh_bastion: true
dot_ssh_bastion_fqdn: bastion.local.tld
dot_ssh_bastion_suffix: lan
```

Hosts:
```
# host_vars/ananas.lan.local.tld.yml
# local server
dot_ssh_host_ip: 192.168.10.10
```

```
# host_vars/rainbowdash.lxc.remote.tld.yml
# remote, behind bastion
dot_ssh_host_ip: 10.0.0.110
```

```
# host_vars/celestia.remote.tld.yml
# remote, no bastion
dot_ssh_host_ip: celestia.remote.tld
```

That .lan thingy story: my day job VPN overwrites my resolvers, which are only accessible through the VPN and I don't want to do ugly things with dnsmasq, so I can't `ssh mylocalhost` anymore nor the long fqdn because they will resolve externally with my public facing DSL IP.

So with the `.lan` suffix, the config generated will have `mylocalhost` (with local IP) and `mylocalhost.lan` using the defined bastion.

## Example generated
```
# ~/.ssh/infra/ananas.cfg
# Managed by ansible
# ananas


Host ananas
    HostName 192.168.10.10

Host ananas.lan
    HostName 192.168.10.10
    ProxyCommand ssh -W %h:%p lan.local.tld
```

```
# ~/.ssh/infra/rainbowdash.cfg
# Managed by ansible
# rainbowdash

Host rainbowdash.lxc.remote.tld
    HostName 10.0.0.110
    ProxyCommand ssh -W %h:%p celestia.remote.tld
```

```
# # Managed by ansible
# celestia

Host celestia.remote.tld
    HostName celestia.remote.tld
```
