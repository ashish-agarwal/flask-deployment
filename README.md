# flask-deployment
Configuring a Linux server to host a web app securely._

# Server details
IP address: `54.197.30.225`

SSH port: `2200`

# Configuration changes
## Add user
Add user `grader` with command: `sudo adduser grader`

## Add user grader to sudo group
```
usermod -aG sudo grader
```

## Update all currently installed packages

`apt-get update` - to update the package indexes

`apt-get upgrade` - to actually upgrade the installed packages

## Set-up SSH keys for user grader

```
mkdir /home/grader/.ssh
chmod 700 /home/grader/.ssh
touch /home/grader/.ssh/authorized_keys
nano /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

Can now login as the `grader` user using the command:
`ssh -i privatekey grader@54.197.30.225`

## Disable root login
Change the following line in the file `/etc/ssh/sshd_config`:

From `PermitRootLogin without-password` to `PermitRootLogin no`.

Also, uncomment the following line so it reads:
```
PasswordAuthentication no
```

Do `service ssh restart` for the changes to take effect.

## Change timezone to UTC
Check the timezone with the `date` command. This will display the current timezone after the time.
If it's not UTC change it like this:

`sudo timedatectl set-timezone UTC`

## Change SSH port from 22 to 2200
Edit the file `/etc/ssh/sshd_config` and change the line `Port 22` to:

`Port 2200`

Then restart the SSH service:

`sudo service ssh restart`

## Configuration Uncomplicated Firewall (UFW)

`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow 2200/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/tcp`

To check the rules that have been added before enabling the firewall use:

`sudo ufw show added`

To enable the firewall, use:

`sudo ufw enable`

To check the status of the firewall, use:

`sudo ufw status`

## Install Apache to serve a Python mod_wsgi application
Install Apache:

`sudo apt-get install apache2`

Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'test123';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## Install Flask, SQLAlchemy, etc
Issue the following commands:
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
```

## Clone the repository that contains Project 3 Catalog app
Move to the `/srv` directory and clone the repository of the catalog app.
```
cd /srv
sudo git clone https://github.com/ashish-agarwal/retaurant.git
```

## Configure Apache2 to serve the app
To serve the catalog app using the Apache web server, a virtual host configuration file
needs to be created in the directory `/etc/apache2/sites-available/`, in this case called
`catalog.conf`. Here are its contents:

```
   <VirtualHost *:80>

WSGIDaemonProcess  user=root group=root threads=5
        WSGIScriptAlias / /srv/project-catalog/app.wsgi
        <Directory /srv/project-catalog>
                 <IfVersion < 2.3 >
                        Order allow,deny
                        Allow from all
                </IfVersion>
                 <IfVersion >= 2.3>
                        Require all granted
                </IfVersion>
        </Directory>
        Alias /static /srv/project-catalog/catalog/static
        <Directory /srv/project-catalog/catalog/static/>
                 <IfVersion < 2.3 >
   Order allow,deny
   Allow from all
  </IfVersion>
  <IfVersion >= 2.3>
   Require all granted
  </IfVersion>
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Documentation for the WSGI parameters can be found [here][5].

Disable the default virtual host with:

`sudo a2dissite 000-default.conf`

Then enable the catalog app virtual host:

`sudo a2ensite catalog.conf`

To make these Apache2 configuration changes live, reload Apache:

`sudo service apache reload`

The catalog app should now be available at `http://54.197.30.225`
