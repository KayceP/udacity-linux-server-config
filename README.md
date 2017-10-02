# Udacity Linux Server Configuration

## Intro

For this project I had to initialize a complex server-side application Host for the Item Catalog Project.

## Login Information

* IP address: 35.163.210.85
* Accessible via ssh port 2200

## Guide

### 1 - Create a new user named *grader* and grant this user sudo permissions

1. Log into the remote VM as *ubuntu* user through ssh: `$ ssh ubuntu@34.214.229.172`.
2. Add a new user called *grader*: `$ sudo adduser grader`.
3. Though Udacity and others may reccommend editing /etc/sudouser file - *DONT*. Instead, use this command: 
`sudo usermod -a -G sudo (grader)`
_Lightsail is very finicky, and will either not allow the changes to be saved, or will allow it to be saved but will 
not be able to log you back in._

### 2 - Update all currently installed packages

1. ```$ sudo apt-get update```.
2. ```$ sudo apt-get upgrade```.
3. Install *finger*, a utility software to check users' status: ```$ apt-get install finger```.


### 4 - Configure the key-based authentication for *grader* user

1. Generate an encryption key **on your local machine** with: `$ ssh-keygen -f ~/.ssh/grader.rsa`.
2. Log into the remote VM as *ubuntu* user through ssh and create the following file: `$ touch /home/grader/.ssh/authorized_keys`.
3. Copy the content of the *grader.pub* file from your local machine to the */home/grader/.ssh/authorized_keys* file you just created on the remote VM. Then change some permissions:
  1. `$ sudo chmod 700 /home/grader/.ssh`.
  2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
  3. Finally change the owner from *root* to *grader*: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/grader.rsa grader@35.163.210.85`.

### 5 - Enforce key-based authentication

1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and edit it to *no*.
2. `$ sudo service ssh restart`.

### 6 - Change the SSH port from 22 to 2200

1. `$ sudo nano /etc/ssh/sshd_config`. Find the *Port* line and edit it to *2200*.
2. `$ sudo service ssh restart`.
3. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@34.214.229.172`
4. This limits the port connection to only port 2200
_Alert! If you are using Amazon Lightsail to initialize the server you must also open the port through the Lightsail Firewall. Further instructions in Configuring the Firewall

### 7 - Disable ssh login for *root* user

1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit it to *no*.
2. `$ sudo service ssh restart`.
3. This deny's the access of *root* user.


### 8 - Configure the Uncomplicated Firewall (UFW)

Project requirements are for the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

**Alert! If you are using AWS Lightsail. Go to the Lightsail Firewall console and add port 2200. If you don't you will get locked out of your server via ssh until you open it.
1. `$ sudo ufw allow 2200/tcp`.
2. `$ sudo ufw allow 80/tcp`.
3. `$ sudo ufw allow 123/udp`.
4. `$ sudo ufw enable`.

### 9 - Install Apache, mod_wsgi

1. `$ sudo apt-get install apache2`.
2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
3. `$ sudo service apache2 start`.