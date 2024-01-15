+++
title = 'DF Jobs Architecture'
date = 2024-01-10T12:10:46-07:00
draft = false
+++

# Ansible

When I first moved away from PaaS platforms like Heroku and Render, it quickly became daunting running a VPS manually. Eventually I stumbled upon Ansible. 

Ansible helps automate your server instances, it can manage your servers or hosts from a single workstation. The best part about it is that you don't need to host it on a server, instead you can run it straight from your laptop.
    

## Getting started with Ansible

Ansible allows you to ssh into your servers and carry out commands defined in what's called a "playbook". You can see already that this means in one repo you can carry out automated instructions that provision your server across various instances.

## Ansible Config

Ansible essientially has two groups, a directory with ansible config, and a playbooks config. In the anbile config files, aptly named ansible.cfg, you specifiy an inventory file and a few others.
    
        ```yaml
        [defaults]
        inventory = inventory.ini
        remote_user = your_local_machine_user
        private_key_file = ~/.ssh/private_ssh_key
        ```

The inventory file holds all of your available servers. The remote user and the private key will allow ansible to ssh into your servers to carry out the commands you specifiy in a playbook.

I like to create an ansible folder within the root project


    ```bash
    mkdir ansible/ 
    touch ansible/ansible.cfg
    cd ansible && nano ansible.cfg
    ```

Afterwards you are ready to define your inventory.


## Inventory File
Here you can add your list of servers IPs or if you domain names. There can be many servers here, you can even break them down into groups using roles, but for now we will have all of the servers here configured the same.

You can define a group of your servers as whatever you like, in this case we will keep it simple - servers. 

    ```yaml
    [servers]
    1.23.45.678
    2.34.56.789
    ```

Here we defined two server IP addresses, if you've set up ssh keys properly ansible will be able to connect and run your commands in a playbook.

## Playbooks

A playbook is a yaml file consisting of a list of task. I like to create a playbooks directory:

    ```bash
    mkdir playbooks
    touch playbooks/server_setup.yaml
    ```

These task can perform anything from creating a user on a remote machine, setting up cron jobs, installing and configuring packages, etc.

The file is senstive with its syntax, so you want to make sure you are identing properly. Essientialy the file gets converted to a script that runs on your remote machine.


Here is an example from my own ansible server setup.

    ```yaml
    ---
    hosts: servers
    become: true
    tasks:
      - name: Install Packages
        apt:
          name:
          - docker.io
          - vim-nox
          - nginx
          - certbot
          - python-certbot-nginx
        state: present
        update_cache: yes
      
      - name: Install Docker Compose
        get_url: "https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-{{ ansible_system }}-{{ ansible_architecture }}"
          url: ""
          dest: "/usr/local/bin/docker-compose"
          mode: 'u+x,g+x"
    ```

Notice the hosts variable, I pass the name of the group of remote machines here, as we specified in our inventory file. That means on every machine this set of code will run.

This line means that ansible will use sudo to run these commands. 


    ```yaml
    become: true
    ```
    

It's best practice to create an ansible user on your workstation (i.e., your laptop in this sceanrio), and an ansible user on each of the remote machine to run these commands. Though this is outside of the scope of this post. More on that in a future post.

Afterwards we define a series of tasks:

    ```yaml
    tasks:
      - name:
        ...
    ```

Each task can be given a name, though not necessary its always best practice. Depending on the type of task you want to pass there are additional arguments.

In this case we want to install packages across all of our servers.


    ```yaml
    tasks:
      - name: Install Packages
        apt:
          name:
          - docker.io
          - vim-nox
          - nginx
          - certbot
    ```

Because my server is a ubuntu machine, I have access to apt directly, depending on your machine's OS you may have to consult ansible docs.

In this case I amm listing a number of packages that will be on each of the machines. Docker is essiential, vim-nox for my editing needs, nginx, certbot, and python-cerbot-nginx for my reverse proxy and ssl certification generation. That last 3 packages are web app related packages.

I'll get to nginx and whate are reverse proxies in a future post.

The last two lines ensures that these packages are installed, and that they are the latest packages available. 

    ```yaml
    state: present
    update_cache: yes
    ```

A great feature of ansible is the impotency of it all. you can add packages later on and run the command and ansible will add the newly added packages.

## Putting it all together 

Now that you have the essientials set up, you can run ansible!

    ```bash
    $ ansile-playbook -i ansible/inventory.ini playbooks/server_setup.yml
    ```

    ```bash
    PLAY [servers] ***************************************************************************************

    TASK [Gathering Facts] *******************************************************************************
    pk: [1.23.45.678]

    TASK [Install Packages] ********************************************************************************
    changed: [1.23.45.678]

    PLAY RECAP *******************************************************************************************
    1.23.45.678                : ok=1    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    ```


Again if SSH is all setup then, you should see something like the above in your terminal. Ssh into your machine and check for yourself!

## Additional info
Ansible can do a great many things, essientially aything you need to setup on your machines can be done through a single ansible repo, you can automate db backups, cron jobs, git repo pulls, and anything else.


    




