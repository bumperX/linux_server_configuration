# Linux Server Configuration

This is a project for the Udacity [FSND Course](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). 

## Description
This project links to the [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299). It takes a baseline installation of a Linux server and prepare it to host the Item Catalog website, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

* URL: `http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com`
* IP address: `18.217.0.163`
* SSH port: `2200`
* Login: `ssh -i ~/.ssh/[privateKeyFileName] -p 2200 grader@18.217.0.163`

## Get the server
Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances)

## Instructions for SSH access to the instance

1. Download Private Key from the __SSH keys__ section in the __Account__ section on Amazon Lightsail
2. Move the private key file into the folder `mv ~/Downloads/Lightsail.pem ~/.ssh/`
3. `chmod 400 ~/.ssh/Lightsail.pem`
4. Login ubuntu with `ssh -i ~/.ssh/Lightsail.pem ubunut@18.217.0.163`

## Create a new user __grader__

1. Add user grader `sudo adduser grader`
2. Edit the sudoers file `sudo visudo`
3. Copy and paste`(grader ALL=(ALL:ALL) ALL)` below `root ALL=(ALL:ALL) ALL`, save and quit
4. Check sudo access with `sudo cat /etc/sudoers`

## Set ssh login using keys

1. Generate keys on local machine with `ssh-keygen`
2. Save the private key in `~/.ssh` on local machine
3. Key-based SSH authentication

	On virtual machine:
	```
	$ sudo su grader
	$ cd
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ nano .ssh/authorized_keys 
	```
	Copy the content of public key (.pub) on your local machine and paste here
  
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```

3. Reload SSH `sudo service ssh restart`
4. Login with __grader__ `ssh -i ~/.ssh/[privateKeyFilename] grader@18.217.0.163`

## Firewall configuration and change port
1. `sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200` and `PermitRootLogin without-password` to `PermitRootLogin no`
2. Reload SSH with `sudo service ssh restart`
3. Allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	```
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
	sudo ufw status
	```

4. Add and save __port 2200__ with Application as Custom as TCP in the Networking section of your instance on Amazon Lightsail.
5. Now login with __ubuntu__ or __grader__

	```
	ssh -i ~/.ssh/Lightsail.pem -p 2200 ubuntu@18.217.0.163
	ssh -i ~/.ssh/[privateKeyFilename] -p 2200 grader@18.217.0.163
	```

## Update packages
	sudo apt-get update 
	sudo apt-get upgrade

To automatically install security updates

	sudo apt-get install unattended-upgrades
	sudo dpkg-reconfigure --priority=low unattended-upgrades



## Install and configure __Apache__ to serve Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Restart Apache `sudo service apache2 restart`

## Install and configure __PostgreSQL__

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Login `sudo su - postgres`
3. Get into postgreSQL shell `psql`
	1. Create a new database named catalog 
	`postgres=# CREATE DATABASE catalog;`
	2. Create a new user named catalog
	`postgres=# CREATE USER catalog;`
	3. Set a password for user catalog 
	`postgres=# ALTER ROLE catalog WITH PASSWORD 'password';`
	4. Give user "catalog" permission to "catalog" application database
	`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
	5. Quit postgreSQL `postgres=# \q`
	6. Exit `exit`

 
## Setup Catalog App project
1. Install Git `sudo apt-get install git`
2. `cd /var/www` 
4. `cd FlaskApp`
3. Create directory `sudo mkdir FlaskApp`
5. Clone the Catalog App `sudo git clone https://github.com/bumperX/catalog_web_app.git`
6. Move all project files to `var/www/FlaskApp/FlaskApp` 
	
	```
	sudo mv -v catalog_web_app/catelog/* catalog_web_app/
	sudo rm -r catalog_web_app/catelog/
	sudo mv ./catalog_web_app ./FlaskApp
	```

7. `cd FlaskApp`
8. Rename application.py `sudo mv application.py __init__.py`
9. Edit database path in __init__.py, database_setup.py, populate_db.py

	```
	sudo nano __init__.py
	sudo nano database_setup.py
	sudo nano populate_db.py
	```
	In each file, change `sqlite:///catalog.db` to `postgresql://catalog:password@localhost/catalog`



10. Install pip `sudo apt-get install python-pip`
11. Update pip `pip install --upgrade pip`
12. Install project dependencies `sudo -H pip install sqlalchemy flask-sqlalchemy psycopg2 requests flask oauth2client`

13. Install psycopg2 `sudo apt-get install postgresql python-psycopg2`
14. Create database `python database_setup.py`
15. Populate database `python populate_db.py`


## Configure virtual host
1. `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 18.217.0.163
		ServerAdmin admin@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```

3. Enable the virtual host `sudo a2ensite FlaskApp`
4. Restart Apache `sudo service apache2 restart`

## Create wsgi file
1. `cd /var/www/FlaskApp`
2. Create the .wsgi File `sudo nano flaskapp.wsgi`
3. Add the following lines to __flaskapp.wsgi__
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")
	from FlaskApp import app as application
	application.secret_key = 'super_secret_key'
	```

4. Restart Apache `sudo service apache2 restart`

## Oauth Login
1. Go to console.cloud.google.com and edit __Credentails__
2. Add the Authorized JavaScript origins
	
	```
	http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com
	http://18.217.0.163
	```

3. Add the Authorized redirect URIs
	
	```
	http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com/oauth2callback
	http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com/gconnect
	http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com/login
	http://ec2-18-217-0-163.us-east-2.compute.amazonaws.com/catalog
	```
4. Save and download the new __client_secrets.json__ file
5. Update `client_secret.json` in the virtual machine with the new content

## Run application
1. Connect to __Flask.conf__ `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the hostname below ServerAdmin and paste `ServerAlias ec2-18-217-0-163.us-east-2.compute.amazonaws.com`
3. Connect to __client_secrets.json__ and give it to the absolute path by changing `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json` in `__init__.py`



## References:
1. [Udacity's FSND Forum](https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration)
2. [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
3. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
4. [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
5. [Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)
6. [mod_wsgi(Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#installing-mod-wsgi)
7. [Ubuntu Packages Search](https://packages.ubuntu.com)
8. [Reverse IP Lookup](https://mxtoolbox.com/ReverseLookup.aspx)
9. [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

