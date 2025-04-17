
# Cloud-1 

This topic is inspired by the subject Inception. </br>
The goal is to deploy your site and the necessary docker infrastructure on an instance provided by a cloud provider. </br>
In this version, each process will have its container. </br>
You CANNOT deploy the same
images from Inception and be done with it ; You can of course get the source of the
website (Your WordPress blog for instance), but you have to deploy it using a container
per process and automation.</br>
Automation is essential here. The stages of deployment must be automated by a tool
of your choice (We suggest Ansible). </br>
This complete web server must be able to run several services in parallels such as
Wordpress, PHPmyadmin and a database.

## Documentation 
- [Xavki - ansible playlist](https://www.youtube.com/watch?v=kzmvwc2q_z0&list=PLn6POgpklwWoCpLKOSw3mXCqbRocnhrh-)
- [Ansible doc](https://docs.ansible.com/ansible/latest/getting_started/index.html)

## Vm (virtualbox) environment configuration
- Create a virtual machine
- Create 2 interfaces:
    - Host-only network
    - NAT network
Add them to the machine's adapters.
- Verify the network interfaces (private and public addresses).
If the interfaces are configured but don't have an IP address, use the `dhclient` command or edit the `/etc/network/interfaces` file to define the interfaces with the DHCP option:

```
auto enp0sX  
iface enp0sX inet dhcp  
```
- Install SSH
- Add the SSH user key
If the user is root, you need to edit the `/etc/ssh/sshd_config` file to allow password-based connections by setting `PermitRootLogin: yes`.
Add the key using the command `ssh-copy-id`, then edit the `sshd_config` file again and change the `PermitRootLogin value to prohibit-password` to restrict root password login.


## How to use 

Clone the repository 
```
git clone git@github.com:mtellal/cloud-1
```
Install the `inception` submodule
```
git submodule init
git submodule update
```
Copy the env file to `/roles/inception/templates/env` and set the variables
```
DOMAIN_NAME={{ ansible_host }}
# certificates
CERTS_=/etc/nginx/ssl/inception.crt

# MYSQL SETUP
SQL_DATABASE=inception_db
SQL_ROOT_PASSWORD=
SQL_USER=
SQL_PASSWORD=

# WORDPRESS SETUP
WP_TITLE=inception_wp
WP_ADMIN_USER=
WP_ADMIN_PASSWORD=
WP_ADMIN_EMAIL=
WP_USER_LOGIN=
WP_USER_EMAIL=
WP_USER_PASSWORD=

#phpmyadmin 
PMA_HOST=mariadb
PMA_PORT=3306
#PMA_USER=
#PMA_PASSWORD=
PMA_ABSOLUTE_URI=https://{{ansible_host}}/phpmyadmin/
```

Finally, launch the `cloud-1` playbook
```
ansible-playbook playbooks/cloud_1.yml
```

To remove packages and folders use the `clean_cloud-1` playbook
```
ansible-playbook playbooks/clean_cloud-1.yml
```

# Ansible 

### Installation and Configuration
To install Ansible you can use:
- pip
- apt 

>  **Warning !** Ansible search the old path of python2: `/usr/bin/python`. To use python3 you need to set the `ansible interpreter` variable in the inventory configuration `ansible_python-interpreter=/usr/bin/python2`

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
- `ANSIBLE_CONFIG=` - env var that specify the path of the configuration file 
- Playbook folder - specific to the playbook
- User home directory - specific to the user
- Global configuration `/etc/ansible/ansible.cfg` - apply to each call to ansible

To see the differents options in the configuration file you can see the `/etc/ansiblbe/ansible.cfg` file or generate a default config file with the command: `ansible-config init --disabled > ansible.cfg`

- `-m raw` - precise to doesn't use the python interpreter

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


Copy files and folders
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

In the inventory folder:
- inventory.yml 
- group_vars - Must be a name of specific group in the inventory or name of the folder 
- hosts_vars - It must be name of a specific host or a folder, it include the vars for the target
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

- Bind roles and hosts in inventory, perform tasks

- Idempotence:
If a task are played twice with the same inputs and get the same results then the state is not updated 

- Stateful:
Save a state

-> We can execute tasks in local, ssh is not used in this case 
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