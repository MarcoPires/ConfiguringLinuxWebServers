# P5: Linux Server Configuration

This project aims to install the application "Restaurant app" on a web server and make it available to the world, as well as install updates on the server, securing it from a number of attack vectors and installing/configuring web and database server.

#### About Restaurant app
Restaurant app is an application where you are shown the restaurant and its menus, organized by type
This project allows us to quickly know the pairs list for the next round.
Users can add their own restaurants and menus.

## Version
1.0.0 - 21/12/2015

## Tech

Restaurant app uses:

* Python 2.7
* Flask 0.10.1
* Postgresql
* Html
* CSS

## Requirements

Restaurant app needs:

* Python 2.7
* Flask 0.10.1
* postgresql 9.4.5
* sqlalchemy 1.0.10


## How to connect

#### Public IP address:

52.33.123.83

#### SSH port:

2200

#### Application URL:

http://ec2-52-33-123-83.us-west-2.compute.amazonaws.com/

## Step By Step
1. Downloaded ssh private key for online development environment through Udacity at https://www.udacity.com/account#!/development_environment
```
# (where ~ is your environment's home directory)
> ssh -i ~/.ssh/udacity_key.rsa root@52.33.123.83
```
2. Create a new user named **grader**
```
# Add user named "grader", and set is password
> sudo adduser grader

# Configure the user "grader" password to expire in next login
> sudo passwd -e grader

# Create root access to user "grader"
> cd /etc/sudoers.d
> sudo touch grader
> nano grader
# Add the following line to the "grader" file
grader ALL=(ALL) NOPASSWD:ALL
```
3. Grant access to "grader" user
```
# On local machine, generate a new key pair
> ssh-keygen
# create a directory under the "grader" home directory on the server
> mkdir .ssh
# create and edit the public key file
> touch .ssh/authorized_key
> nano .ssh/authorized_key
# Now past the public key content into the new file
# Update the folder and file access permitions 
> chmod 700 .ssh
> chmod 664 .ssh/authorized_key
```
4. Set login to only accept key pair and not allow password
```
> sudo nano /etc/ssh/sshd_config
# Change "PasswordAuthentication" to "no" 
> sudo service ssh restart
```
5. Upgrade the system
```
# Download lists from the repositories and "updates" them
> sudo apt-get update
# Upgrade system
> sudo apt-get upgrade
# Removes packages that were automatically installed but now their dependent package has either been removed or no longer depends on them
> sudo apt-get autoremove
```
6. Install required software
```
> sudo apt-get install finger
> sudo apt-get install ntp
> sudo apt-get install apache2
> sudo apt-get install libapache2-mod-wsgi
> sudo apt-get install postgresql
> sudo apt-get install python-psycopg2
> sudo apt-get install git
> sudo apt-get install python-virtualenv
> sudo easy_install sqlalchemy
> sudo easy_install Flask
> pip install oauth2client
```
7. Set up the firewall
```
# Check the firewall status, by default is inactive
> sudo ufw status
# Deny any entry attempt
> sudo ufw default deny incoming
# Deny access to default ssh port
> sudo ufw deny 22/tcp
> sudo ufw deny 22
# Enable outputs
> sudo ufw default allow outgoing
# Enable ssh access, at port 2200
> sudo ufw allow ssh
> sudo ufw allow 2200/tcp
# Enable http access, at default port 80
> sudo ufw allow www
# Enable http ntp, at port 123
> sudo ufw allow ntp
> sudo ufw allow 123/tcp
# Turn on the firewall
> sudo ufw enable
# Once again check the firewall status, which should be enabled
> sudo ufw status
> logout
# From now on, it is necessary to specify the port at login
> ssh -i udacity_key.rsa root@52.33.123.83 -p 2200
```
8. Config timezone to UTC
```
> sudo dpkg-reconfigure tzdata
# after the command (select "None of the above", then select "UTC" near the bottom of the list)
```
9. Config apache server
```
# Test editing the default main page
> cd /var/www/html
> nano /var/www/html/index.html
# Add the following line to the file
"hello world"
# Configure Apache2 to handle WSGI module
nano /etc/apache2/sites-enabled/000-default.conf
# Add the following block

<VirtualHost *:80>
        ServerName http://ec2-52-33-123-83.us-west-2.compute.amazonaws.com/
        ServerAdmin mpires@leya.com
        DocumentRoot /var/www/html/restaurantApp
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIScriptAlias / /var/www/html/restaurantApp/__main__.wsgi
        <Directory /var/www/html/restaurantApp/>
                Order deny,allow
                Allow from all
        </Directory>
        Alias /static /var/www/html/restaurantApp/restaurantApp/static
        <Directory /var/www/html/restaurantApp/restaurantApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /restaurantApp/data /var/www/html/restaurantApp/restaurantApp/data
        <Directory /var/www/html/restaurantApp/restaurantApp/data/>
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost>

# Restart apache service
> sudo service apache2 restart
# Add the "__main__.wsgi", application entry point
> touch __main__.wsgi
```
10. configure PostgreSQL
```
# Logoin with prostgres root user "postgres"
> su - postgres
# Configure a new template
> psql template1
# Create new user "catalog"
> CREATE USER catalog WITH PASSWORD 'catalog';
# Create new database "catalog"
> CREATE DATABASE catalog;
# Grant privileges only to "catalog" user 
> GRANT ALL PRIVILEGES ON catalog TO catalog;
```
11.  Clone Restaurant App 
```
> git clone https://github.com/MarcoPires/restaurantApp.git
```
12. Install "glaces" for monitoring the application and provide some feedback on the server status
```
# Install "glaces" and the dependencies 
> curl -L http://bit.ly/glances | /bin/bash
> pip install glances
# Run "glaces"
> glances
```
13. A Cron script has configured to "update" and "upgrade" the system
```
> crontab -e
# Add the following block
0 5 * * 1 sudo apt-get update
2 5 * * 1 sudo apt-get upgrade
```
14. Firewall has been configured to block IPs after repeated failed login attempts
```
# Install "fail2ban"
> sudo apt-get install fail2ban
# Backup the config file
> sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
# Configure "Fail2Ban"
> sudo nano /etc/fail2ban/jail.local
# Stop and deploy the service
> sudo service fail2ban stop
> sudo service fail2ban start
```

## References
* [Udacity Forum P5 How I got through it](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342)
* [How to setup SSH access to development environment](https://www.udacity.com/account#!/development_environment)
* [Ubunto serverguide](https://help.ubuntu.com/lts/serverguide/index.html)
* [Configuring timezone and NTP](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)
* [Package information on libapache2-mod-wsgi](https://packages.debian.org/unstable/python/libapache2-mod-wsgi)
* [Deploying Flask app on Apache](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#installing-mod-wsgi)
* [Set up PostgreSQL on Linux](http://www.postgresql.org/docs/manuals/)
* [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

## General use and License
This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  If not, see <http://www.gnu.org/licenses/>.


## Contacts
For more information feel free to contact me:
e-mail : mpires@leya.com

