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

### 10 - Install and Configure Git

1. `$ sudo apt-get install git`.
2. Configure your username: `$ git config --global user.name <username>`.
3. Configure your email: `$ git config --global user.email <email>`.

### 11 - Clone the Catalog app from Github

1. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
2. Change owner for the *catalog* folder: `$ sudo chown -R grader:grader catalog`.
3. Move inside that newly created folder: `$ cd /catalog` and clone the catalog repository from Github: `$ git clone https://github.com/iliketomatoes/catalog.git catalog`.
4. Make a *catalog.wsgi* file to serve the application over the *mod_wsgi*. That file should look like this:

```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

### 12 - Install virtual environment, Flask and the project's dependencies

1. Install *pip*, the tool for installing Python packages: `$ sudo apt-get install python-pip`.
2. If *virtualenv* is not installed, use *pip* to install it using the following command: `$ sudo pip install virtualenv`.
3. Move to the *catalog* folder: `$ cd /var/www/catalog`. Then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
4. Activate the virtual environment: `$ source venv/bin/activate`.
5. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.
6. Install Flask: `$ pip install Flask`.
7. Install all the other project's dependencies: `$ pip install bleach httplib2 request oauth2client sqlalchemy psycopg2`. 

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).

### 13 - Configure and enable a new virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 35.163.210.85
    ServerAlias 
    ServerAdmin admin@35.163.210.85
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* The **WSGIDaemonProcess** line specifies what Python to use and can save you from a big headache. In this case we are explicitly saying to use the virtual environment and its packages to run the application.

3. Enable the new virtual host: `$ sudo a2ensite catalog`.

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps).

### 14 - Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'postgres';`.
5. Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `# \c catalog`.
8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
10. Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.
11. Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:postgres@localhost/catalog')
```
12. Setup the database with: `$ python /var/www/catalog/catalog/database_setup.py`.
13. To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it, if necessary, to make it look like this: 
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

### 15 - Install system monitor tools

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

Source: [eHowStuff](http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/).

### 16 - Restart Apache to launch the app

1. `$ sudo service apache2 restart`.

### 17 - Install system monitor tools.

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

## ACKNOWLEDGMENTS

1. [kssawicki] - (https://github.com/kssawicki/) (Being there on slack in the middle of the night when no one else was.)
2. [iliketomatoes] - (https://github.com/iliketomatoes/linux_server_configuration)
3. [stueken] - (https://github.com/stueken/)
