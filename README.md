#LINUX SERVER VM CONFIGURATION#

>Public IP Address:
>35.161.147.129

###1. Launch your Virtual Machine with your Udacity account.
  1. Create AWS Remote Server via Udacity account interface
  2. Download Private Key
  3. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory).
   * ```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
  4. Open your terminal and type in
   * ```chmod 600 ~/.ssh/udacity_key.rsa```

###2. SSH into your server
  1. ```ssh -i ~/.ssh/udacity_key.rsa root@35.161.147.129```

###3. Create a new user named grader
  1. ```sudo adduser grader```

###4. Give the grader the permission to sudo
  1. ```sudo cat /etc/sudoers```
  2. ```sudo ls /etc/sudoers.d```
  3. ```sudo nano  /etc/sudoers.d/grader```
  4. Add `grader ALL=(ALL) NOPASSWD:ALL` to the grader file to give the user sudo permission.
  5. To enforce key based authentication, type `sudo nano /etc/ssh/sshd_config` and make sure the `PasswordAuthentication` line is set to no and restart the ssh server by typing `sudo service ssh restart`

###5. Update all currently installed packages
  1. ```sudo apt-get update```
  2. ```sudo apt-get upgrade```

###6. Change the SSH port from 22 to 2200
  1. ```sudo nano /etc/ssh/sshd_config```
  2. Change Port to 2200

###7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  1. Check if the firewall is active or inactive
   * ```sudo ufw status```
  2. Close all incoming ports
   * ```sudo ufw default deny incoming```
  3. Allow all outgoing ports
   * ```sudo ufw default allow outgoing```
  4. Allow ssh port
   * ```sudo ufw allow ssh```
  5. Allow http port
   * ```sudo ufw allow www```
  6. Allow ntp port
   * ```sudo ufw allow ntp```
  7. Enable the firewall
   * ```sudo ufw enable```

###8. Configure the local timezone to UTC
  1. Type:
   * ```sudo dpkg-reconfigure tzdata```
  2. Go to "Etc" or "None of the above" 
  3. Choose UTC and enter

###9. Install and configure Apache to serve a Python mod_wsgi application
  1. Install Apache: ```sudo apt-get install apache2```
  2. Install mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi```
  3. ```nano /etc/apache2/sites-enabled/000-default.conf```
  4. Add the following line at the end of the <VirtualHost *:80> block, right before the closing</VirtualHost> line: ```WSGIScriptAlias / /var/www/html/myapp.wsgi```
  5. Restart Apache: ```sudo apache2ctl restart```
  6. Create the /var/www/html/myapp.wsgi: ```sudo nano /var/www/html/myapp.wsgi```. 
  7. Within this file, write the following application to print "Hello World!":
```
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
````

###10. Install and configure PostgreSQL:
  1. ```sudo apt-get install postgresql```
  2. Login to automatically created postgres user: ```sudo su - postgres```
  3. From here, we can connect to the system: ```psql```
  4. Create catalog user: ```CREATE ROLE catalog WITH login;```
  5. Create catalog database and make catalog user the owner```CREATE DATABASE catalog WITH OWNER catalog;```
  6. Double check that no remote connections are allowed by looking in the host based authentication file: `sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
   * As you can see, the first two security lines specify "local" as the scope that they apply to. This means they are using Unix/Linux domain sockets.
   * The second two declarations are remote, but if we look at the hosts that they apply to (127.0.0.1/32 and ::1/128), we see that these are interfaces that specify the local machine.
  7. Exit out of PostgreSQL and the postgres user by typing: `exit`

###11. Install git, clone and setup your Catalog App project
  1. ```sudo apt-get install git```
  2. ```cd /var/www/```
   * Create new directory named FlaskApp: ```mdir FlaskApp```
   * Move to the new directory: ```cd FlaskApp```
  3. Git clone our existing Catalog app into this directory: ```git clone https://github.com/ruslanml/Udacity-Catalog-Project.git```
  4. Move inside the Catalog app directory: ```cd Udacity-Catalog-Project```
  5. Rename the ```project.py``` file to ```__init__.py```
   * ```mv project.py __init__.py```
  6. Install **pip** and **virtualenv** 
   * ```sudo apt-get install python-pip ```
   *  ```sudo pip install virtualenv```
   * ```sudo virtualenv venv```
  7. Install **Flask** and other dependencies in the new created virtual environment
   * ```source venv/bin/activate``` 
   * ```sudo pip install Flask```
   * ```sudo pip install psycopg2```
   * ```sudo pip install sqlalchemy```
   * ```sudo pip install oauth2client```
  8. Enable port 5432 in order to allow PostgreSQL to run properly
   * ```sudo ufw allow 5432```
  9. Run the following command to test if the installation is successful and the app is running:
   * ```sudo python __init__.py```
   * If app is successfully running then deactive the virtual environment by running: `deactivate`
  10. Configure and enable new virtual host
   * ```sudo nano /etc/apache2/sites-available/FlaskApp.conf```
   * Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:
```
<VirtualHost *:80>
  ServerName mywebsite.com
  ServerAdmin admin@mywebsite.com
  WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
  <Directory /var/www/FlaskApp/Udacity-Catalog-Project/>
  Order allow,deny
  Allow from all
  </Directory>
  Alias /static /var/www/FlaskApp/Udacity-Catalog-Project/static
  <Directory /var/www/FlaskApp/Udacity-Catalog-Project/static/>
    Order allow,deny
    Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

   * Enable the virtual host: ```sudo a2ensite FlaskApp```

  11. Create the .wsgi File
   * ```cd /var/www/FlaskApp```
   * ```sudo nano flaskapp.wsgi```
   * Add the following lines of code to the flaskapp.wsgi file:
```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/")

  from Udacity-Catalog-Project import app as application
  application.secret_key = 'Add your secret key' 
```

  12. Restart Apache
   * ```sudo apache2ctl restart```


###Sources:
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

###Other Useful Commands:
* Login to Server: `ssh -i ~/.ssh/udacity_key.rsa root@35.161.147.129`
* View Error Log: `cat /var/log/apache2/error.log`


###Errors and Warnings:
1. Error: sudo: unable to resolve host ip-10-20-47-235. To fix:
 * ```sudo nano /etc/hosts```
 * ```127.0.1.1 ip-10-20-47-235```
2. Error: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
 * This is just a friendly warning and not really a problem (as in that something does not work).
 * If you insert a ```ServerName localhost``` in either httpd.conf or apache2.conf in /etc/apache2 and restart apache the notice will disappear.