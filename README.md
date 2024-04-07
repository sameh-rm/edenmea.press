
# Press Deploying Steps

### Update and Upgrade Packages

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

### Install Required Packages

A software like ERPNext, which is built on Frappe Framework, requires a number of packages in order to run smoothly. These are the packages we will be installing in this step.

### Install GIT

```bash
sudo apt-get install git
```

### Install Python

requires Python version 3.10+. This is what we will install in this step.

```bash
sudo apt-get install python3-dev python3.10-dev python3-setuptools python3-pip python3-distutils
```

### Install Python Virtual Environment

A virtual environment helps in managing the dependencies for one software at one place, without having to interfere with other sections in the computer or server in which the software is running.

```bash
sudo apt-get install python3.10-venv
```

### Install Software Properties Common

Software Properties Common will help in repository management.

```bash
sudo apt-get install software-properties-common
```

### Install MariaDB

ERPNext is built to naively run on MariaDB. The team is working to have the same working on PostgreSQL, but this is not ready yet.

```bash
sudo apt install mariadb-server mariadb-client
```

### Install Redis Server

```bash
sudo apt-get install redis-server
```

### Install other packages

ERPNext functionality also relies on other packages we will install in this step. These will load fonts, PDFs, and other resources to our instance.

```bash
sudo apt-get install xvfb libfontconfig wkhtmltopdf
sudo apt-get install libmysqlclient-dev
```

### Configure MYSQL Server

Setup the server

```bash
sudo mysql_secure_installation
```

When you run this command, the server will show the following prompts. Please follow the steps as shown below to complete the setup correctly.

Enter current password for root: (Enter your SSH root user password)

- Switch to unix_socket authentication [Y/n]: Y
- Change the root password? [Y/n]: Y
- It will ask you to set new MySQL root password at this step. This can be different from the SSH root user password.
- Remove anonymous users? [Y/n] Y
- Disallow root login remotely? [Y/n]: N
- This is set as N because we might want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.
  - Remove test database and access to it? [Y/n]: Y
  - Reload privilege tables now? [Y/n]: Y

### Edit MYSQL default config file

```bash
sudo nano /etc/mysql/my.cnf
```

Add the following block of code exactly as is:

```bash
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

### Restart the MYSQL Server

```bash
sudo service mysql restart
```

### Instal CURL, Node, NPM and Yarn

Install CURL

```bash
sudo apt install curl
```

Install NVM

```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash

source ~/.profile
```

### install Node 18+

```bash
nvm install 18

# Install NPM
sudo apt-get install npm -y

# Install Yarn
sudo npm install -g yarn

# Install Frappe Bench
sudo pip3 install frappe-bench

# Initialize Frappe Bench
bench init --frappe-branch version-15 frappe-bench

# Switch directories into the Frappe Bench directory
cd frappe-bench

# Change user directory permissions
# This will give the bench user execution permission to the home directory.
chmod -R o+rx /home/[frappe-user]
```

### Install Press dependencies

- install unzip

```bash
sudo apt install unzip
```

- install aws

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Create Press required directories
>
> use these directories in press Settings Infrastructer tab

```bash
mkdir -p /home/frappe/.certbot/webroot
mkdir -p /home/frappe/frappe-bench/.clones
mkdir -p /home/frappe/frappe-bench/.docker-builds
```

#### Configure AWS Credentials
>
> fill the following with your Credentials and do the same **with sudo**

```bash
aws configure && sudo aws configure
# AWS Access Key ID [None]:
# AWS Secret Access Key [None]:
# Default region name [None]:
# Default output format [None]:
```

#### install nginx and certbot

```bash
sudo snap install core; sudo snap refresh core
sudo apt update
sudo apt install nginx -y
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
pip3 install certbot-dns-route53
```

#### install docker

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update


sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER
# Reboot the VM
sudo reboot
```

### Fix sudo node version

```bash
sudo cp $(which node) /usr/bin/node
```

### Create a New Site

A site is a requirement in ERPNext, Frappe and all the other apps we will be needing to install. We will create the site in this step.

``` bash
bench new-site [site-name]
```

### Install ERPNext and other Apps

Download all the apps we want to install
The first app we will download is the payments app. This app is required when setting up ERPNext.

```bash
bench get-app payments
```

Next, we will download ERPNext app

```bash
bench get-app --branch version-15 erpnext
```

### Donwload Press App

```bash
bench get-app https://github.com/sameh-rm/press.git
```

### Install all the apps on our site

```bash
bench --site [site-name] install-app erpnext payments press
```

We have successfully setup ERPNext on ubuntu 22.04. You can start the server by running the below command:

```bash
bench start
```

> If you didn’t have any other ERPNext instance running on the same server, ERPNext will get started on port **8000**. If you visit **[YOUR SERVER IP:8000]**, you should be able to see ERPNext version 14 running.

> Please note that instances which are running on develop mode, like the one we just setup, will not get started when you restart your server. You will need to run the bench start command every time the server restarts.

In the below steps, we will learn how to deploy the production mpde.

Setting ERPNext for Production

```bash
# Enable Scheduler
bench --site [site-name] enable-scheduler


# Disable maintenance mode
bench --site [site-name] set-maintenance-mode off

# Setup production config
sudo bench setup production frappe

# Setup NGINX to apply the changes
bench setup nginx

# Restart Supervisor and Launch Production Mode
sudo supervisorctl restart all
sudo bench setup production frappe

# configure TLS certificate for site
bench config dns_multitenant on
sudo bench setup lets-encrypt [site-name]
```

## Configure Press

- create 3 Servers [proxy, mysql, apps]
- add route53 records for these 3 servers:
  - proxy.cloud.edenmea.com
  - mysql.cloud.edenmea.com
  - apps.cloud.edenmea.com
- create ssh on the Press Server

    ```bash
    ssh-keygen -t rsa -b 4096 -C "Main Server"
    ```

- show ssh public key

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

- add the Press Server Public key into all 3 servers

    ```bash
    sudo -i
    echo "[PUBL_KEY]" >> .ssh/authorized_keys
    ```

- make sure you can access all 3 servers via Press Server SSH

    ```bash
    ssh root@proxy.cloud.edenmea.com
    ```

    ```bash
    ssh root@mysql.cloud.edenmea.com
    ```

    ```bash
    ssh root@apps.cloud.edenmea.com
    ```

## open your site and go to press settings

    - add the directories we created before into Infrastructer and docker tabs and press Save
    - create new root domain
    - get docker registry config from azure registry
    - add registry config into docker tab
    - configure stripe settings PUBL_KEY and PRIVATE_KEY
    - create Email Account for the default outgoing for email registrations
    - create site plan to use in new sites
    - create Server Plan for DB Server and  Apps Server
    - create proxy Server
    - create DB Server
    - create Server For Apps this Server Ram should be 8GB+
    - create release group
    - create new App, app release, and app source
    - first app in release group should be frappe app
    - make sure app source status is cloned and approved
    - create a deploy candidate from release group actions
