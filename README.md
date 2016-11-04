Project
-------
Linux Server Configuration on AWS

Description
-----------
The objective of this project is to get a base installation of Linux (Ubuntu) which is hosted in AWS, to be securely configured from attacks and also host the web application (Item Catalogue) from the earlier completed projects.


Server Details
--------------

- Web application: http://ec2-35-162-25-39.us-west-2.compute.amazonaws.com
- Server IP: 35.162.25.39
- SSH Port: 2200

- Username: grader
- Connection Paramaters: ssh -i udacity_key_grader.rsa grader@35.162.25.39 -p2200

Summary of Changes
------------------

## 1) Adding a new user and granting sudo rights.

Creating user;

```
sudo adduser grader
```

Adding user to sudo and geting to prompt for password for sudo;

```
sudo nano /etc/sudoers.d/grader
grader ALL=(ALL) ALL
```

## 2) Enabling SSH login for the "grader" user.

Generate SSH key on local PC for "grader" user;

```
ssh-keygen
```

Create SSH directory for "grader" user and place public key there;

```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
nano /home/grader/.ssh/authorized_keys
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

Now the "grader" user can login with the following;

```
ssh -i udacity_key_grader.rsa grader@35.162.25.39
```

## 3) Disable remote root login.

Edit the sshd_config file

```
nano /etc/ssh/sshd_config
```

Change "PermitRootLogin without-password" to "PermitRootLogin no".

Restart the SSH service.

```
service ssh restart
```

## 4) Update all currently installed packages.

Update the repository listings and then fetch updates;

```
apt-get update
apt-get upgrade
reboot
```

## 5) Change SSH port from 22 to 2200.

Edit the sshd_config file;

```
nano /etc/ssh/sshd_config
```

Change the line "Port 22" to "Port 2200".

Then restart the SSH service;

```
sudo service ssh restart
```

Now for future logins the port number has to be specified;

```
ssh -i udacity_key_grader.rsa grader@35.162.25.39 -p2200
```

## 6) Configuration UFW (Uncomplicated Firewall).

Close all incoming ports and then only enable ports 80 (www), 2200 (ssh) and 123 (ntp);

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw show added
sudo ufw enable
sudo ufw status
```

## 7) Change date/time to UTC.

Run the "date" command to see the current date/time format. Date/time in this instance was
already set to UTC.

```
date
sudo dpkg-reconfigure tzdata
```

Install NTP so time can be syncronised with servers online.

```
sudo apt-get install ntp
sudo service ntp reload
```

## 8) Install Apache web server.

```
sudo apt-get install apache2
```

Confirm installation was sucessfull. Navigate to http://ec2-35-162-25-39.us-west-2.compute.amazonaws.com

## 9) Install WSGI.

```
sudo apt-get install libapache2-mod-wsgi
```

## 10) Install and configure PostgreSQL.

```
sudo apt-get install postgresql
```

Check if no remote connections are allowed;

```
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```

Opened the config file and all entries point to 127.0.0.1 or ::1 (localhost) only.

Now creating a user "catalog" which has access to the catalog database only;

```
sudo -u postgres createuser -P catalog
```

Create the catalog database and give the "catalog" user rights to it;

```
sudo -u postgres createdb -O catalog catalog
```

## 11) Install Git and Clone Item Catalogue Project.

Install prerequists for Item Catalogue project - psycopg2, flask, sqlalchemy, pip,
oauth2client;

```
sudo apt-get install python-psycopg2
sudo apt-get install python-flask python-sqlalchemy
sudo apt-get install python-pip
pip install oauth2client
```

Install Git;

```
sudo apt-get install git
```

Clone the item catalouge project;

```
cd /var/www
sudo git clone https://github.com/aspoljaric/itemcatalogue.git
```

## 12) Configure Item Catalogue Project.

The following lines had to be replaced in the following files;
- "db_connection.py"

```
create_engine('sqlite:///categories.db')
create_engine('postgresql://catalog:password@localhost/catalog')
```

- "database.py"

```
create_engine('sqlite:///categories.db')
create_engine('postgresql://catalog:password@localhost/catalog')
```

The database then had to be initialised;

```
sudo python database.py
```

The other change required was to authorise the host address on Google App Engine to allow Oauth authentication from the AWS URL.

## 13) Adding a New Virtual Host to Apache.

Create a new file;

```
sudo nano /etc/apache2/sites-available/itemcatalogue.conf
```

Populate the file with the following configuration;

```
<VirtualHost *:80>
        ServerName http://ec2-35-162-25-39.us-west-2.compute.amazonaws.com
        ServerAdmin albert.spoljaric@gmail.com
        WSGIScriptAlias / /var/www/itemcatalogue/catalog.wsgi
        <Directory /var/www/itemcatalogue>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/itemcatalogue/static
        <Directory /var/www/itemcatalogue/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Add the new virtual host and then reload the Apache service;

```
sudo a2ensite itemcatalogue
sudo service apache2 reload
```

## 14) WSGI Configuration.

Create a new file;

```
sudo nano /var/www/itemcatalogue/catalog.wsgi
```

Populate the file with the following configuration;

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/itemcatalogue/")
from application import app as application
application.secret_key = '17763f3d2dd9f6762cd967ee53efdb82'
```

Restart Apache;

```
sudo service apache2 restart
```

Resources Referenced
--------------------
- [Ubuntu Time Management](https://help.ubuntu.com/community/UbuntuTime)
- [PostgreSQL 9.3.15 Documentation](https://www.postgresql.org/docs/9.3/static)
- [Flask Deploying](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [Apache HTTP Server Version 2.4 Documentation](https://httpd.apache.org/docs/2.4/)

Author
------
Albert Spoljaric (albert.spoljaric@gmail.com)