
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

- `-m raw` - doesn't use the python interpreter

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


copy files and folders
- copy module - from host to target
- fetch module - from target to host



## Inventory 

-> INI simpe to read 
-> YAML tree and hierarchy organized (best practice)

```
all:
    children:
        parent1:            <- first group 'parent1'
            hosts:
                srv3        <- hosts/target of parent1
            children:
                child1:     
                    children:
                        srv1:
                        srv2:
...
```

### Vars / Env vars

in the inventory folder:
- inventory.yml 
- group_vars - Must be a name of specific group in the inventory or name of the folder 
- hosts_vars - It must be name of a specifi host or a folder, it include the vars for the target
```
|- inventory.yml
|  |
|  |-- group_vars
|      |-- [ file.yml || file/ ] ...
```

`ansible-inventory --graph`
```
@all:
  |--@ungrouped:
  |--@hosts:
  |  |--scaleway
  |  |--vm
```
`ansible-inventory --graph --vars`
```
@all:
  |--@ungrouped:
  |--@hosts:
  |  |--scaleway
  |  |  |--{ansible_host = 51.159.180.166}
  |  |  |--{ansible_user = root}
  |  |  |--{inception_folder = {{ ansible_env.HOME }}/inception}
  |  |--vm
  |  |  |--{ansible_host = 192.168.56.104}
  |  |  |--{ansible_user = root}
  |  |  |--{inception_folder = {{ ansible_env.HOME }}/inception}
  |  |--{ansible_user = root}
  |  |--{inception_folder = {{ ansible_env.HOME }}/inception}
```

## Playbook 

- Bind roles and hosts in inventory and perform tasks

- Idempotence:
If a task are played twice with the same inputs and the same result is equal then nothing is done 

- Stateful:
Save a state

-> We can task execute task in local, ssh is not used in this case 
```
hosts: localhost
connection: local
```

### Register 

Save the output of a command to a register that can be used after inside the playbook

```
...
    # task 
    register: __register_var
...
```

### Template

Allow us to generate a file in fly with dynamic var and copy it to the target machine 
```
./template.j2
# env vars file or configuration file
hostname={{ ansible_hostname }} 
```

```
./playbook.yml
...
    ansible.builtin.template:
        src: ./template.j2
...     dest: ./srcs/.env
```

### Notify / Handlers

```
...
    # ex: copy project files
    notify: handler_action
...

 handlers:
    - name: handler_action
        systemd:
          name: project_service
          state: reloaded
```

By adding `notify: handler_action` to a task and define a handlers section we can perform action based on a updated task. 
If the files in our project folder change then a handler action will be performed and `systemd` will reload our `project_service`.


### Roles

-> Group of actions 
-> Common templates, files etc ....





### Tasks

### Gather facts 