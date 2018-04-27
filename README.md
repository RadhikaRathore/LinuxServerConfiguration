# Linux Server Configuration
This project is linked to the Configuring Linux Web Servers, where we will secure and set up a Linux server.
By the end of this project, we will have one of our web applications running live on a secure web server.

## Prerequisites

   -  Linux server instance
   
## Instace details
  
   - Instace used: Amazon Lightsail instance
   - SSH PORT used: 2200
   - Public IP:  35.154.216.249
   - url to access application running on linux server :
     http://35.154.216.249
	 http://ec2-35-154-216-249.ap-south-1.compute.amazonaws.com/

##  Software to install during the configuration
   - Apache2
   - mod_wsgi
   - PostgreSQL
   - git
   - pip
   - virtualenv
   - httplib2
   - Python Requests
   - oauth2client
   - SQLAlchemy
   - Flask
   - libpq-dev
   - Psycopg2

##  Configuration steps

### Connect to Linux Server
1. Create an instance with Amazon Lightsail

2.Connect to the instance on a local machine
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Windows machines (using Git Bash)

- Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

- Click on 'Download default key'

-  A file of .pem type will be downloaded ,we can rename it to make log in simple.(I have rename it AwsKeyPair.pem)

- move AwsKeyPair.pem to  ~/.ssh,

    mv ~/Downloads/AwsKeyPair.pem ~/.ssh/

- Run `chmod 600 ~/.ssh/AwsKeyPair.pem

- Log in with the following command: `ssh -i ~/.ssh/AwsKeyPair.pem ubuntu@XXX.XXX.XXX.XXX`.

### Upgrade currently installed packages
- Notify the system of what package updates are available by running `sudo apt-get update`

- Download available package updates by running `sudo apt-get upgrade`


### Configure the firewall
- Start by changing the SSH port from `22` to `2200` (open up the /etc/ssh/sshd_config file, change the port number to `2200`from 22. then restart SSH by running `sudo service ssh )

- Check to see if the ufw (the preinstalled ubuntu firewall) is active by running `sudo ufw status`

- Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in

- Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing

- Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH

- Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200` so that SSH will work

- Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

- Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP

- Run `sudo ufw deny 22` to deny port `22` (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port `2200`)

- Run `sudo ufw enable` to enable the ufw firewall

- Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

	```
	To                         Action      From
	--                         ------      ----
	22                         DENY        Anywhere
	2200/tcp                   ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	123/udp                    ALLOW       Anywhere
	22 (v6)                    DENY        Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	123/udp (v6)               ALLOW       Anywhere (v6)
	```

- Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed; make sure to deny the default port `22`)

- Now, to login , open up the Terminal and run:

	`ssh -i ~/.ssh/AwsKeyPair.pem -p 2200 ubuntu@XXX.XXX.XXX.XXX`, where XXX.XXX.XXX.XXX is the public IP address of the instance

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.


### Create a new user named `grader`
- Run `sudo adduser grader`

- Enter in a new UNIX password (twice) when prompted

- Fill out information for the new `grader` user

- To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
- Run `sudo visudo`

- Search for a line that looks like this:

	`root    ALL=(ALL:ALL) ALL`

- Add the following line below this one:

	`grader	   ALL=(ALL:ALL) ALL`

- Save and close the visudo file

- To verify that `grader` has sudo permissions, `su` as `grader` (run `su - grader`), enter the password, and run `sudo -l`; after entering in the password (again), a line like the following should appear, meaning `grader` has sudo permissions:

	```
	Matching Defaults entries for grader on
	    ip-XX-XX-XX-XX.ec2.internal:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

	User grader may run the following commands on
		ip-XX-XX-XX-XX.ec2.internal:
	    (ALL : ALL) ALL
	```


### Allow `grader` to log in to the virtual machine
- Run `ssh-keygen` on the local machine

- Choose a file name for the key pair (such as grader_key)

- Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

- Log in to the virtual machine

- Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

- Run `touch .ssh/authorized_keys`

- On the local machine, run `cat ~/.ssh/insert-name-of-file.pub`

- Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

- Run `chmod 700 .ssh` on the virtual machine

- Run `chmod 644 .ssh/authorized_keys` on the virtual machine

- Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

- Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XXX.XXX.XXX.XXX`


### Configure the local timezone to UTC

- Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)


### Install and configure Apache
- Run `sudo apt-get install apache2` to install Apache

- Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load

### Install mod_wsgi
- Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

- Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`


### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
- Install PostgreSQL by running `sudo apt-get install postgresql`

- Open the /etc/postgresql/9.5/main/pg_hba.conf file

- Make sure it looks like this (comments have been removed here for easier reading):

	```
	local   all             postgres                                peer
	local   all             all                                     peer
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
	```

### Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. Something like the following should appear:

	Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
	[GCC 5.4.0 20160609] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> 

### Create a new PostgreSQL user named `catalog` with limited permissions
- PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the `postgres` user for accessing the PostgreSQL software)

- Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

- Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

- Next, give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

- Finally, give the `catalog` user a password by running `\password catalog`

- Check to make sure the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of 
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

- Exit psql by running `\q`

- Switch back to the `ubuntu` user by running `exit`


### Create a Linux user called `catalog` and a new PostgreSQL database
- Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted
	- fill out information for `catalog`

- Give the `catalog` user sudo permissions:
    
	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file
	- to verify that `catalog` has sudo permissions, `su` as `catalog` (run `sudo su - catalog`), and run `sudo -l`
	- after entering in the UNIX password, a line like the following should appear (meaning `catalog` has sudo permissions):

		```
		User catalog may run the following commands on
			ip-XX-XX-XX-XX.ec2.internal:
		    (ALL : ALL) ALL
		```

- While logged in as `catalog`, create a database called catalog by running `createdb catalog`

- Run `psql` and then run `\l` to see that the new database has been created

- Switch back to the `ubuntu` user by running `exit`


### Install git and clone the catalog project
- Run `sudo apt-get install git`

- Create a directory called 'projectManagementSystem' in the /var/www/ directory

- Change to the 'projectManagementSystem' directory, and clone the catalog project:

	`sudo git clone https://github.com/RadhikaRathore/ProjectManagementSystem.git projectManagementSystem`
	
- Change the ownership of the 'projectManagementSystem' directory to `ubuntu` by running (while in /var/www):

	`sudo chown -R ubuntu:ubuntu projectManagementSystem/`

- Change to the /var/www/projectManagementSystem/projectManagementSystem directory

- Change the name of the application.py file to \_\_init__.py by running `mv application.py __init__.py`

- In \_\_init__.py, find line 508:

	`app.run(host='0.0.0.0', port=8000)`

	Change this line to:

	`app.run()`

- edit databasesetup.py,dummydata.py,__init__.py and change 
    engine = create_engine('sqlite:///projectmanagement.db')  to 
	
	engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
	
### Add client_secrets.json file
- Authenticate login through Google:

	- Create a new project on the Google API Console

	- Create an OAuth Client ID (under the Credentials tab), and make sure to add http://XXX.XXX.XXX.XXX and http://ec2-XX-XX-XX-XX.compute--amazonaws.com as authorized JavaScript origins

	- Add http://ec2-XX-XX-XX-XX.compute--amazonaws.com/login, http://ec2-XX-XX-XX-XX.compute--amazonaws.com/gconnect, and http://ec2-XX-XX-XX-XX.compute--amazonaws.com/oauth2callback as authorized redirect URIs

	- Create a file called client_secrets.json in the /var/www/projectManagementSystem/projectManagementSystem/ directory 

	- Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

	- Add the client ID to line 16 of the templates/login.html file in the project directory

	- Add the complete file path for the client_secrets.json file in lines 33 and 63 in the \_\_init__.py file; change it from 'client_secrets.json' to '/var/www/projectManagementSystem/projectManagementSystem/client_secrets.json'

	
### Set up a vitual environment and install dependencies
- Start by installing pip (if it isn't installed already) with the following command:

	`sudo apt-get install python-pip`

- Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

- Change to the /var/www/projectManagementSystem/projectManagementSystem/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running `virtualenv venv` (make sure to _not_ use `sudo` here as it can cause problems later on)

- Activate the new environment, `venv`, by running `. venv/bin/activate`

- With the virtual environment active, install the following dependenies (note: with the exception of the libpq-dev package, make sure to _not_ use `sudo` for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):

	`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` (Note: this will install to the global evironment)

	`pip install psycopg2`

- In order to make sure everything was installed correctly, run `python __init__.py`; the following (among other things) should be returned:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

- Deactivate the virtual environment by running `deactivate`


### Set up and enable a virtual host
- Create a file in /etc/apache2/sites-available/ called projectManagementSystem.conf

- Add the following into the file:

	```
	<VirtualHost *:80>
			ServerName XXX.XXX.XXX.XXX
			ServerAdmin ben.in.campbell@gmail.com
			WSGIScriptAlias / /var/www/projectManagementSystem/projectManagementSystem.wsgi
			<Directory /var/www/projectManagementSystem/projectManagementSystem/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/projectManagementSystem/projectManagementSystem/static
			<Directory /var/www/projectManagementSystem/projectManagementSystem/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```

	Note: the `Options -Indexes` lines ensure that listings for these directories in the browser is disabled.

- Run `sudo a2ensite projectManagementSystem` to enable the virtual host

	The following prompt will be returned:

	```
	Enabling site projectManagementSystem.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

- Run `sudo service apache2 reload`


### Write a .wsgi file
- Apache serves Flask applications by using a .wsgi file; create a file called projectManagementSystem.wsgi in /var/www/projectManagementSystem

- Add the following to the file:

	```
	activate_this = '/var/www/projectManagementSystem/projectManagementSystem/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/projectManagementSystem/")

	from projectManagementSystem import app as application
	application.secret_key = '12345'
	```

- Resart Apache: `sudo service apache2 restart`


### Disable the default Apache site
- At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

- Run `sudo service apache2 reload`  


### Change the ownership of the project direcotries
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

	sudo chown -R www-data:www-data projectManagementSystem/


### Set up the database schema and populate the database
- While in the /var/www/projectManagementSystem/projectManagementSystem/ directory, activate the virtualenv by running `. venv/bin/activate`

- Then run `python dummydata.py`

- Deactivate the virtualenv (run `deactivate`)

- Resart Apache again: `sudo service apache2 restart`

- Now open up a browser and check to make sure the app is working by going to http://XXX.XXX.XXX.XXX or http://ec2-XX-XX-XX-XX.compute--amazonaws.com


## 6. Sources
Below is a list of sources used to complete this project.

Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

Udacity course: [Linux Command Line Basics](https://www.udacity.com/course/linux-command-line-basics--ud595)

Stack OverFlow

Tutorials Point

Flask [documentation](http://flask.pocoo.org/docs/0.12/installation/) on virtualenv

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/latest/orm/cascades.html) on cascade delete

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/rel_1_0/orm/basic_relationships.html) on one-to-many relationships

Flask [documenation](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments) on making a virtualenv work with mod_wsgi


# Creator

**Radhika Rathore**



