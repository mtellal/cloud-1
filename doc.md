
## Vm (virtualbox) environment configuration
- Crate a virtual machine 
- Create 2 interfaces:
    - host-only-network
    - NAT network
Add them to the machine adaptaters
- Verify the network interfaces (private and public addresses)
If the interfaces are set but doesn't have an ip address. Use the `dhclient` command or edit the `/etc/network/interfaces` file to define the interfaces with the dhcp option 
```
auto enp0sX
iface enp0sX inet dhcp
```
- Install ssh
- Add user ssh key 
If the user is root you need to edit the `/etc/ssh/sshd_config` file to permit the connection via password `PermiRootLogin: yes`
Add the key wit the command `ssh-copy-id` and edit once again the variable `PermiRootLogin: prohbit-password` to restrict the root password connection.


# Ansible 

### Installation and Configuration
To install ansilbe we can use:
- pip
- apt

!!! Cautions ansible search the old path of python2: `/usr/bin/python`. To use python3 you need to set the ansible interpreter variable in the inventory configuration `ansible_python-interpreter=/usr/bin/python2`

#### SSH 
Useful commands:
- `ssh-copy-id user@ip` - copy the ssh public key to user@ip  
- `eval \`ssh-agent\`` - launch a ssh agent
- `ssh-add` - add a key to an agent
- `ssh-add -l` - check the key associated of the agent
- `~/.ssh/config` - configuration file of ssh agent 
```
~/.ssh/config - example
Host 192.168.56.100
    User root
```
SSH agent try to connect as root from 192.168.56.100 

#### ansible.cfg
- ANSIBLE_CONFIG= - env var that specify the path of the configuration file 
- Playbook folder - specific to the playbook
- User home directory - specific to the user used
- Global configuration `/etc/ansible/ansible.cfg` - apply to each call to ansible

To see the differents options possible in the configuration file you can see the `/etc/ansiblbe/ansible.cfg` file or generate a default config file with the command: `ansible-config init --disabled > ansible.cfg`

### Environment vars 

- Globally in the playbook 
```
...
environment:
    var1: "FOO"
...
```
- Individually in a specific task 
```
- name: xxx
...
environment:
    var1: xxx
...
```