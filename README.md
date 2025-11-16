# Ansible is here to serve you

## Install Ansible
```bash
sudo apt-get install ansible -y
```

## Write your inventory file

Your **inventory** file typically contains list of hosts you are going to configure with ansible. It may look like:
```txt
172.31.5.10
172.31.12.60
```

## Ping Hosts

You can essentially check if the connection to hosts is possible or not with `ping` module:
```ansible
ansible all --key-file ~/.ssh/private_key -i inventroy -m ping
```
You gave private key for auth which is one of the pair of public keys of hosts in **inventory** file.

## Limit Targets

It is possible that you run ansible commands specifc to certain host. Use `--limit`:
```ansible
ansible all --key-file ~/.ssh/private_key -i inventory -m ping --limit 172.31.5.10
```

## Set Defaults

If you are like me to feel bore writing long commands like above, you should definately use config file for ansible. Check the following:
```cfg
[defaults]
# ansible.cfg
inventory = inventory
private_key_file = ~/.ssh/private_key
```

You may have guessed. We already give **inventory** and **key** file for ansible to read from `ansible.cfg` in current directory. So,our shortened command is:
```ansible
ansible all -m ping
```
Quite Fantastic!

## List Hosts

You can list hosts which are listed in your **inventory** file:
```ansible
ansible all --list-hosts
```

## Gather Facts

You can even get quick informtion about your hosts or server such as: memory details, cpu, networking details, etc. using `gather_facts` module:
```ansible
ansible all -m gather_facts
```

