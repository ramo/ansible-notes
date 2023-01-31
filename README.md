# Introduction
- Ansible is an IT automation tool.
- Used for
    - Configure systems
    - Deploy software
    - Orchestrate IT tasks (Continuous Deployment, zero downtime rolling updates)
## Why Ansible?
- Agentless and uses OpenSSH for connectivity.
- Simple enough to everyone in IT, yet powerful enough to peform more complex tasks.
- Complex scripts can be written in few lines of ansible-playbook

# Ansible Concepts
- `Control node` - The machine from which ansible CLI tools run.
- `Managed node (hosts)` - Target devices managed by Ansible. 
## Inventory
- A list of managed nodes.
- Default inventory file - `/etc/ansible/hosts`.
- `ini` format file.
- Alias of a managed node can be given in the beginning of the line. 
- Inventory parameters:
    - ansible_connection - ssh/winrm/localhost
    - ansible_poert - 22/5986
    - ansible_user - root/administrator
    - ansible_ssh_pass - Password
    - ansible_password - Windows password.

### Sample inventory file
```ini
# DB servers
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass

# Web servers
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes
```

## Playbooks
- Playbook - A single YAML file `eg. playbook.yml`
- Play - Set of activities (tasks) to be run on hosts. Contains of `name` `hosts` `tasks`
- Task - Single action to be performed on the host. Contains of `name` and modules.
    - Execute a command
    - Run a script
    - Install a package
    - Shutdown / Restart
- `hosts` can have node name or group name mentioned in the inventory. When group name is mentioned the play runs `simaltaneously` on the hosts.
- `ansible-doc -l` command is used to know more about modules. 

### Sample Playbook
```yaml
- name: 'Execute a command to display hosts file on localhost'
  hosts: localhost
  tasks:
    - name: 'Execute a date command'
      command: 'date'
```

## Modules
- The code or binaries that Ansible copies to and executes on managed nodes.
- System, Commands, Files, Database, Cloud, Windows, etc.,

### command
- Executes a command on a remote node.
```yaml
# Simple example
- name: Play1
  hosts: localhost
  tasks:
    - name: 'Execute command date'
      command: date

# Using module parameters
- name: Play2
  hosts: localhost
  tasks:
    - name: 'Display /etc/resolv.conf contents'
      command: cat resolv.conf chdir=/etc
```
| Note |
| :---- |
|`free_form` in the module documentation indicate that the module takes free form command to run. There is no parameter actually named free_form.|

### script
- Runs a local script on a remote node after transferring it.
```yaml
- name: Play1
  hosts: localhost
  tasks:
    - name: 'Run a script on remote server'
      script: /some/local/script.sh -arg1 -arg2
```

### service
- Manage services - Start, Stop, Restart
```yaml
- name: Start services in order
  hosts: localhost
  tasks:
    - name: Start the database service
      service: name=postgresql state=started

    - name: Start the httpd service
      service:
        name: httpd
        state: started
```

### lineinfile
- Search for a line in a file and replace it or add it if it doesn't exist.
```yaml
- name: Add DNS server to resolv.conf
  hosts: localhost
  tasks:
    - name: Add DNS server to resolv.conf
      lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 10.1.250.10'
```

## Variables
- Stores information that varies with each host
- In inventory file, `ansilbe_host`, `ansible_connection`, etc., are variables.
- Defined in the playbook using `vars` directive. key/value pairs of variables will be child of `vars`.
- Defined in separate file with host name. Eg: `web.yaml` where `web` is the ansible host alias.
- Accessed using `{{ variable_name }}` **Jinja2** templating syntax.

| Note |
|:---|
| `{{ variable_name }} neeed to be written inside quotes if it is not concatenated with other strings.`|



