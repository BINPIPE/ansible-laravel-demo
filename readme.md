# Ansible + Laravel Demo

`Learning Resources for DevOps, SRE, Cloud & Engineering Management`

[![BINPIPE](https://img.shields.io/badge/BINPIPE-YouTube-red)](https://www.youtube.com/channel/UCPTgt4Wo0MAnuzNEEZlk90A)
[![Learn DevOps!](https://img.shields.io/badge/BINPIPE-Learn--DevOps-orange)](https://github.com/BINPIPE/resources/blob/master/devops-lesson-plans.md)
[![BINPIPE](https://img.shields.io/badge/Live--Classroom-blue)](https://forms.gle/tDJxDyj2nJyfsgsk7)
---

## Quick Setup

TL;DR:

Make sure you have [Ansible installed](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-18-04) on your control node, which can be your local machine or a remote server dedicated to running Ansible.
You'll need one Ubuntu 18.04 server to serve as node / host.

1. Clone this repository
2. Set up your inventory file - you can use `inventory-example` as base
3. Adjust values on your `group_vars/all.yml` file
4. Run the `server-setup.yml` playbook to set up the LEMP server
5. Run the `laravel-deploy.yml` playbook to deploy the demo Laravel application
6. Access your server's IP address or hostname to test the setup

<hr>




## Prerequisites

To follow this tutorial, you will need:

-   **One Ansible control node**: an Ubuntu 18.04 machine with Ansible installed and configured to connect to your Ansible hosts using SSH keys. 
-   **One or more Ansible Hosts**: one or more remote Ubuntu 18.04 servers. 

## Step 1 — Cloning the Demo Repository

The first thing we need to do is clone the repository containing the Ansible provisioning scripts and the demo Laravel application that we’ll deploy to the remote servers. All the necessary files can be found at the  [https://github.com/BINPIPE/ansible-laravel-demo](https://github.com/BINPIPE/ansible-laravel-demo)  Github repository.

After logging in to your Ansible control node as your sudo user, clone the repository and navigate to the directory created by the  `git`  command:

```
   git clone https://github.com/do-community/ansible-laravel-demo.git
   cd ansible-laravel-demo
```

Now, you can run an  `ls`  command to inspect the contents of the cloned repository:

```
ls -l --group-directories-first
```

You’ll see output like this:

ansible-laravel-demo

```
drwxrwxr-x 3 sammy sammy 4096 Mar 24 15:24 application
drwxrwxr-x 2 sammy sammy 4096 Mar 24 15:24 group_vars
drwxrwxr-x 7 sammy sammy 4096 Mar 24 15:24 roles
-rw-rw-r-- 1 sammy sammy  102 Mar 24 15:24 inventory-example
-rw-rw-r-- 1 sammy sammy 1987 Mar 24 15:24 laravel-deploy.yml
-rw-rw-r-- 1 sammy sammy  794 Mar 24 15:24 laravel-env.j2
-rw-rw-r-- 1 sammy sammy  920 Mar 24 15:24 readme.md
-rw-rw-r-- 1 sammy sammy  318 Mar 24 15:24 server-setup.yml

```

Here’s an overview of each of these folders and files and what they are:

-   `application/`: This directory contains the demo Laravel application that is going to be deployed on the remote server by the end of the workshop.
-   `group_vars/`: This directory holds variable files containing custom options for the application setup, such as database credentials and where to store the application files on the remote server.
-   `roles/`: This directory contains the different  [Ansible roles](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-roles-to-abstract-your-infrastructure-environment)  that handle the provisioning of an Ubuntu LEMP server.
-   `inventory-example`: This file can be used as a base to create a custom inventory for your infrastructure.
-   `laravel-deploy.yml`: This playbook will deploy the demo Laravel application to the remote server.
-   `laravel-env.j2`: This template is used by the  `laravel-deploy.yml`  playbook to set up the application environment file.
-   `readme.md`: This file contains general information about the provisioning contained in this repository.
-   `server-setup.yml`: This playbook will provision a LEMP server using the roles defined in the  `roles/`  directory.

## Step 2 — Setting Up the Inventory File and Testing Connection to Nodes

We’ll now create an inventory file to list the hosts we want to manage using Ansible. First, copy the  `inventory-example`  file to a new file called  `hosts`:

```
cp inventory-example hosts
```

Now, use your text editor of choice to open the new inventory file and update it with your own servers. Here, we’ll use  `nano`:

```
nano hosts
```

The example inventory that comes with the workshop kit contains two Ansible groups:  `dev`  and  `production`. This is meant to demonstrate how to use group variables to customize deployment in multiple environments. If you wish to test this setup with a single node, you can use either the  `dev`  or the  `production`  group and remove the other one from the inventory file.

ansible-laravel-demo/hosts

```
[dev]
203.0.113.0.101

[prod]
203.0.113.0.102

[all:vars]
ansible_python_interpreter=/usr/bin/python3

```

**Note**: the  `ansible_python_interpreter`  variable defines the path to the Python executable on the remote host. Here, we’re telling Ansible to set this variable for all hosts in this inventory file.  

Save and close the file when you’re done. If you are using  `nano`, you can do that by hitting  `CTRL+X`, then  `Y`  and  `ENTER`  to confirm.

Once you’re done adjusting your inventory file, you can execute the  `ping`  Ansible module to test whether the control node is able to connect to the hosts:

```
ansible all -i hosts -m ping -u root
```

Let’s break down this command:

-   `all`: This option tells Ansible to run the following command on  **all**  hosts from the designated inventory file.
-   `-i hosts`: Specifies which inventory should be used. When this option is not provided, Ansible will try to use the default inventory, which is typically located at  `/etc/ansible/hosts`.
-   `-m ping`: This will execute the  `ping`  Ansible module, which will test connectivity to nodes and whether or not the Python executable can be found on the remote systems.
-   `-u root`: This option specifies which remote user should be used to connect to the nodes. We’re using the  **root**  account here as an example because this is typically the only account available on fresh new servers. Other connection options might be necessary depending on your infrastructure provider and SSH configuration.

If your SSH connection to the nodes is properly set up, you’ll get the following output:


```
203.0.113.0.101 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
203.0.113.0.102 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```


The  `pong`  response means your control node is able to connect to your managed nodes, and that Ansible is able to execute Python commands on the remote hosts.

## Step 3 — Setting Up Variable Files

Before running the playbooks that are included in this workshop kit, you’ll first need to edit the variable file that contains settings such as the name of the remote user to create and the database credentials to set up with MariaDB.

Open the  `group_vars/all`  file using your text editor of choice:

```
nano group_vars/all.yml
```

ansible-laravel-demo/group_vars/all.yml

```
---
# Initial Server Setup
remote_user: sammy

# MySQL Setup
mysql_root_password: MYSQL_ROOT_PASSWORD
mysql_app_db: travellist
mysql_app_user: travellist_user
mysql_app_pass: DB_PASSWORD

# Web Server Setup
http_host: "{{ ansible_facts.eth0.ipv4.address }}"
remote_www_root: /var/www
app_root_dir: travellist-demo
document_root: "{{ remote_www_root }}/{{ app_root_dir }}/public"

# Laravel Env Variables
app_name: Travellist
app_env: dev
app_debug: true
app_url: "http://{{ http_host }}"
db_host: localhost
db_port: 3306
db_database: "{{ mysql_app_db }}"
db_user: "{{ mysql_app_user }}"
db_pass: "{{ mysql_app_pass }}"

```

The variables that need your attention are:

-   `remote_user`: The specified user will be created on the remote server and granted  `sudo`  privileges.
-   `mysql_root_password`: This variable defines the database root password for the MariaDB server. Note that this should be a secure password of your own choosing.
-   `mysql_app_db`: The name of the database to create for the Laravel application. You don’t need to change this value, but you are free to do so if you wish. This value will be used to set up the  `.env`  Laravel configuration file.
-   `mysql_app_user`: The name of the database user for the Laravel application. Again, you are not required to change this value, but you are free to do so.
-   `mysql_app_pass`: The database password for the Laravel application. This should be a secure password of your choosing.
-   `http_host`: The domain name or IP address of the remote host. Here, we’re using an Ansible fact that contains the ipv4 address for the  `eth0`  network interface. In case you have domain names pointing to your remote hosts, you may want to create separate variable files for each of them, overwriting this value so that the Nginx configuration contains the correct hostname for each server.

When you are finished editing these values, save and close the file.

### Creating additional variable files for multiple environments

If you’ve set up your inventory file with multiple nodes, you might want to create additional variable files to configure each node accordingly. In our example inventory, we have created two distinct groups:  `dev`  and  `production`. To avoid having the same database credentials and other settings in both environments, we need to create a separate variable file to hold production values.

You might want to copy the default variable file and use it as base for your production values:

```
 cp group_vars/all.yml group_vars/production.yml
 nano group_vars/production.yml
```

Because the  `all.yml`  file contains the default values that should be valid for all environments, you can remove all the variables that won’t need changing from the new  `production.yml`  file. The variables that you should update for each environment are highlighted here:

ansible-laravel-demo/group_vars/production

```
---
# Initial Server Setup
remote_user: prod_user

# MySQL Setup
mysql_root_password: MYSQL_PROD_ROOT_PASSWORD
mysql_app_pass: MYSQL_PROD_APP_PASSWORD

# Laravel Env Variables
app_env: prod
app_debug: false

```

Notice that we’ve changed the  `app_env`  value to  `prod`  and set the  `app_debug`  value to  `false`. These are recommended Laravel settings for production environments.

Once you’re finished customizing your production variables, save and close the file.

### Encrypting variable files with Ansible Vault

If you plan on sharing your Ansible setup with other users, it is important to keep the database credentials and other sensitive data in your variable files safe. This is possible with Ansible Vault, a feature that is included with Ansible by default. Ansible Vault allows you to encrypt variable files so that only users with access to the vault password can view, edit or unencrypt these files. The vault password is also necessary to run a playbook or a command that makes use of encrypted files.

To encrypt your production variable file, run:

```
ansible-vault encrypt group_vars/production.yml
```

You will be prompted to provide a vault password and confirm it. Once you’re finished, if you check the contents of that file, you’ll see that the data is now encrypted.

If you want to view the variable file without changing its contents, you can use the  `view`  command:

```
ansible-vault view group_vars/production.yml
```

You will be prompted to provide the same password you defined when encrypting that file with  `ansible-vault`. After providing the password, the file’s contents will appear in your terminal. To exit the file view, type  `q`.

To edit a file that was previously encrypted with Ansible Vault, use the  `edit`  vault command:

```
ansible-vault edit group_vars/production.yml
```

This command will prompt you to provide the vault password for that file. Your default terminal editor will then be used to open the file for editing. After making the desired changes, save and close the file, and it will be automatically encrypted again by Ansible Vault.

You have now finished setting up your variable files. In the next step, we’ll run the playbook to set up Nginx, PHP-FPM, and MariaDB (which, along with a Linux-based operating system like Ubuntu, form the LEMP stack) on your remote server(s).

## Step 4 — Running the LEMP Playbook

Before deploying the demo Laravel app to the remote server(s), we need to set up a LEMP environment that will serve the application. The  `server-setup.yml`  playbook includes the Ansible roles necessary to set this up. To inspect its contents, run:

```
cat server-setup.yml
```

ansible-laravel-demo/server-setup.yml

```
---
- hosts: all
  become: true
  roles:
    - { role: setup, tags: ['setup'] }

    - { role: mariadb, tags: ['mysql', 'mariadb', 'db', 'lemp'] }

    - { role: php, tags: ['php', 'web', 'php-fpm', 'lemp'] }

    - { role: nginx, tags: ['nginx', 'web', 'http', 'lemp'] }

    - { role: composer, tags: ['composer'] }

```

Here’s an overview of all the roles included within this playbook:

-   `setup`: Contains the tasks necessary to create a new system user and grant them  `sudo`  privileges, as well as enabling the  `ufw`  firewall.
-   `mariadb`: Installs the MariaDB database server and creates the application database and user.
-   `php`: Installs  `php-fpm`  and PHP modules that are necessary in order to run a Laravel application.
-   `nginx`: Installs the Nginx web server and enables access on port  `80`.
-   `composer`: Installs Composer globally.

Notice that we’ve set up a few tags within each role. This is to facilitate re-running only parts of this playbook, if necessary. If you make changes to your Nginx template file, for instance, you might want to run only the Nginx role.

The following command will execute this playbook on all servers from your inventory file. The  `--ask-vault-pass`  is only necessary in case you have used  `ansible-vault`  to encrypt variable files in the previous step:

```
ansible-playbook -i hosts server-setup.yml -u root --ask-vault-pass
```

You’ll get output similar to this:

```
PLAY [all] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************
ok: [203.0.113.0.101]
ok: [203.0.113.0.102]

TASK [setup : Install Prerequisites] ********************************************************************
changed: [203.0.113.0.101]
changed: [203.0.113.0.102]

...

RUNNING HANDLER [nginx : Reload Nginx] ******************************************************************
changed: [203.0.113.0.101]
changed: [203.0.113.0.102]

PLAY RECAP **********************************************************************************************
203.0.113.0.101             : ok=31   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
203.0.113.0.102            : ok=31   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   

```

Your node(s) are now ready to serve PHP applications using Nginx+PHP-FPM, with MariaDB as database server. In the next step, we’ll deploy the included demo Laravel app with the  `laravel-deploy.yml`  Ansible playbook.

## Step 5 — Deploying the Laravel Application

Now that you have a working LEMP environment on your remote server(s), you can execute the  `laravel-deploy.yml`  playbook. This playbook will execute the following tasks:

1.  Create the application document root on the remote server, if it hasn’t already been created.
2.  Synchronize the local application folder to the remote server using the  `sync`  module.
3.  Use the  `acl`  module to set permissions for the  **www-data**  user on the storage folder.
4.  Set up the  `.env`  application file based on the  `laravel-env.j2`  template.
5.  Install application dependencies with Composer.
6.  Generate application security key.
7.  Set up a public link for the  `storage`  folder.
8.  Run database migrations and seeders.

This playbook should be executed by a non-root user with  `sudo`  permissions. This user should have been created when you executed the  `server-setup.yml`  playbook in the previous step, using the name defined by the  `remote_user`  variable.

When you’re ready, run the  `laravel-deploy.yml`  playbook with:

```
ansible-playbook -i hosts laravel-deploy.yml -u sammy  --ask-vault-pass
```

The  `--ask-vault-pass`  is only necessary in case you have used  `ansible-vault`  to encrypt variable files in the previous step.

You’ll get output similar to this:

```
PLAY [all] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************
ok: [203.0.113.0.101]
ok: [203.0.113.0.102]

TASK [Make sure the remote app root exists and has the right permissions] *******************************
ok: [203.0.113.0.101]
ok: [203.0.113.0.102]

TASK [Rsync application files to the remote server] *****************************************************
ok: [203.0.113.0.101]
ok: [203.0.113.0.102]

...

TASK [Run Migrations + Seeders] *************************************************************************
ok: [203.0.113.0.101]
ok: [203.0.113.0.102]

PLAY RECAP **********************************************************************************************
203.0.113.0.101             : ok=10   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
203.0.113.0.102             : ok=10   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

When the execution is finished, you can access the demo application by pointing your browser to your node’s domain name or IP address:

```
http://node_domain_or_IP
```

Credits: digitalocean.com




<pre>
<a href="https://www.binpipe.org">BINPIPE</a> aims to simplify learning for those who are looking to make a foothold in the industry.
Write to me at <b>nixgurus@gmail.com</b> if you are looking for tailor-made training sessions.
For self-study resources look around in this repository, <a href="https://www.binpipe.org/">the Binpipe Blog</a> and <a href="https://www.youtube.com/channel/UCPTgt4Wo0MAnuzNEEZlk90A">Youtube Channel</a>.
</pre>

___
:ledger: Maintainer: **[Prasanjit Singh](https://www.linkedin.com/in/prasanjit-singh)** | **www.binpipe.org**
___

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
