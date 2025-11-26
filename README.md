# Ansible is here to serve you

## Install Ansible
```shell
sudo apt-get install ansible -y
```

## Write your inventory file

Your **inventory** file typically contains list of hosts you are going to configure with ansible. It may look like:

```ini
172.31.5.10
172.31.12.60
```

This **inventroy** file will work only if your servers has the same username as yours which is often not the case. So you should write instead in `user@host` format. For example:

```ini
ubuntu@172.31.5.10
jose@172.31.12.60
```

There is nothing wrong with the above file but in the above case if you want to run [ansible ad hoc commands](https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html), you can run in only the following two ways:

1. Against `all` hosts:
    ```shell
    ansible -m ping all
    ```

2. Against specific hosts:
    ```shell
    ansible -m ping jose@172.31.12.60
    ```

But there is a better way i.e. to group hosts according to usuage or department or type etc. Check this:

```ini
# web servers (bbs)
[webservers]
172.31.37.59

# database servres (bba)
[dbservers]
172.31.39.155

# variables for all
[all:vars]
ansible_user=ubuntu
```
What you did here it group the servers according to their type. You even provide specific variables for **all** webservers. If you need specific variable for each grouping you can use `[dbservers:vars]` to list your variables for **webservers** group.

## Common Ansible Options

### List Hosts

You can list hosts which are listed in your **inventory** file:

```shell
ansible all --list-hosts
```


## Ansible Modules

### Ping Hosts

You can essentially check if the connection to hosts is possible or not using [ping module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html):


```shell
ansible --key-file ~/.ssh/private_key -i inventroy -m ping all
```
You gave private key for auth which is one of the pair of public keys of hosts in **inventory** file.

### Create Files

You can do file operations using [file module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/file_module.html). For example, to create file: 

```shell
ansible --key-file ~/.ssh/private_key -i inventory -m file -a 'path=/tmp/hello state=touch' all
```
You specified arguments with `-a` in the above command.

### Gather Facts

You can even get quick informtion about your hosts or server such as: memory details, cpu, networking details, etc. using `gather_facts` module:

```shell
ansible all -m gather_facts
```

### Others

You can find different builtin modules in [ansible builtin modules](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin) and all modules [here](https://docs.ansible.com/projects/ansible/2.9/modules/list_of_all_modules.html).

## Ansible Config File

If you are like me to feel bore writing long commands like above, you should definately use config file for ansible. Check the following:

```ini
# ansible.cfg
[defaults]
inventory = inventory
private_key_file = ~/.ssh/private_key
```

You may have guessed. We already give **inventory** and **key** file for ansible to read from `ansible.cfg` in current directory which means no need to provide `--key-file` and `-i` option. So,our shortened command is:

```shell
ansible -m ping all
```
Quite Fantastic!


## Ansible Playbooks

Ansible playbook is a list of plays where you provides list of tasks to run on remote host on behalf of specific remote user. It is written in `yaml` format. So, I insist you learn [yaml](learn.yaml.md) first.


### Nginx Server with Ansible

Here is a small example of writing ansible playbook to install `nginx` server and deploying it's home page.

```yaml
---
- name: Webserver deployment
  hosts: webservers
  become: true
  tasks:
    - name: Install or Update nginx server
      ansible.builtin.apt:
        name: nginx
        update_cache: yes
    - name: Start nginx service, if not started and enable if not enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes
    - name: Deploy code for nginx server
      ansible.posix.synchronize:
        src: website/
        dest: /var/www/html/
```


## Ansible Roles

It is great to write playbook in a single `yaml` file when task is not complex but when your task is complex a single playbook may reach thousands of lines of code which is not great for readability and even hard to maintain. So, we use roles which is nothing but a way to modularity. You will divide sections of a playbook into different directory and files inside the module. As shown in [ansible docs](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure), directory structure of ansible role is similar to [following](#custom-ansible-role).

### Custom Ansible Role
```
nginx
├── README.md
├── defaults
│   └── main.yml -> defaults here
├── files -> files here
├── handlers
│   └── main.yml -> handlers here
├── meta
│   └── main.yml -> metadata here
├── tasks
│   └── main.yml -> tasks here
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml -> vars here
```

This structure can be created using:

```shell
ansible-galaxy role init nginx
```


This can be overwhleming but trust me it is okay. You don't need to know all but enough. You will place different section of [the above example to deploy nginx](#Nginx-Server-with-Ansible) into different files in the given directory structure.

It is quite easy for the above playbook to turn into role which is because it has some tasks and some files where tasks will be placed in `nginx/tasks/main.yml` and files i.e. `src: website/` will be moved to `nginx/files` and instead of writing `src: website/` as in tasks, it will be written as `src: files/website/`. See, it was simple as that. But wait?

Now, your main playbook will be reduced to:

```yaml
---
- name: Webserver deployment
  hosts: webservers
  become: true
  roles:
    - nginx
```
See [nginx playbook](nginx_deployment.yaml) for code reference.


That's it, you call `nginx` role where all the instructions related webserver deployment is written. Run your playbook as previously:

```shell
ansible-playbook nginx-pb.yaml
```

### Ansible Roles from Ansible Galaxy

There are a lot of standalone roles in [ansible galaxy](https://galaxy.ansible.com/ui/standalone/roles/). They are like pre-defined docker containers in docker hub or pre-defined modules for terraform. You can install roles created by others, you can publish your own role in the ansible galaxy.

For example, if you are required to install docker container in various OS, you got two option i.e. either write your own role or you search for roles written by community in galaxy and it's free.


While searching role for docker, we can find [role by geerlingguy](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/docker/) is quite reputable. So, we can install and start using it.

#### Step 1: Install and Verify Ansible Role

```shell
ansible-galaxy role install geerlingguy.docker
```
You can verify if the role is installed using any of the given command:
```shell
ls ~/.ansible/roles
```
You should at least see `geerlingguy.docker` folder.

```shell
ansible-galaxy role list
```
You should see at least following output:
```console
# /home/sonu-nigam/.ansible/roles
- geerlingguy.docker, 7.8.0
```

#### Step 2: Use Ansible Role with playbook

Just as in [custom ansible role](#custom-ansible-role), you can use this role with playbook. For example:

```yaml
---
- name: Docker Deployment
  hosts: all
  become: true
  roles:
    - geerlingguy.docker
```
See [docker playbook](docker_pb.yaml) for code reference.

Enjoy!