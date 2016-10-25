#LINUX SERVER VM CONFIGURATION#

>Public IP Address
>35.160.225.160

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
  1. ```sudo apt-get install apache2```
  2. ```sudo apt-get install libapache2-mod-wsgi```

###10. Install and configure PostgreSQL:
  1. ```sudo apt-get install postgresql```
  2. ```sudo su - postgres```
  3. ```psql```
  4. ```CREATE ROLE catalog WITH login;```
  5. ```CREATE DATABASE catalog WITH OWNER catalog;```

###11. Install git, clone and setup your Catalog App project so that it functions correctly when visiting your server’s IP address in a browser.
  1. ```sudo apt-get install git```
  2. ```cd /var/www/html```
  3. ```git clone https://github.com/ruslanml/Udacity-Catalog-Project.git```
  4. ```sudo apache2ctl restart```




**Create a new user named grader
***sudo adduser grader***

**Give the grader the permission to sudo
sudo cat /etc/sudoers
sudo ls /etc/sudoers.d
sudo nano  /etc/sudoers.d/grader
grader ALL=(ALL) NOPASSWD:ALL

1. Error: sudo: unable to resolve host ip-10-20-47-235
2. ***sudo nano /etc/hosts***
3. Add ***127.0.1.1 ip-10-20-47-235***
4. Error: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
5. This is just a friendly warning and not really a problem (as in that something does not work).
6. If you insert a ***ServerName localhost*** in either httpd.conf or apache2.conf in /etc/apache2 and restart apache the notice will disappear.

**Update all currently installed packages
***sudo apt-get update***
***sudo apt-get upgrade***

**Change the SSH port from 22 to 2200
***sudo nano /etc/ssh/sshd_config***
Change Port to 2200

**Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
***sudo ufw status*** -is the firewall active or inactive
***sudo ufw default deny incoming*** -close all incoming ports
***sudo ufw default allow outgoing*** -allow all outgoing ports
***sudo ufw allow ssh*** -allow ssh port
***sudo ufw allow www*** -allow http port
***sudo ufw allow ntp*** -allow ntp port
***sudo ufw enable*** -enable the firewall

**Configure the local timezone to UTC
***sudo dpkg-reconfigure tzdata***
then go to "Etc" or "None of the above", there to UTC, enter, you're done. It actually changes files /etc/timezone and /etc/localtime.

**Install and configure Apache to serve a Python mod_wsgi application
sudo apt-get install apache2
Apache, by default, serves its files from the /var/www/html directory.
sudo apt-get install libapache2-mod-wsgi
***nano /etc/apache2/sites-enabled/000-default.conf***
For now, add the following line at the end of the <VirtualHost *:80> block, right before the closing</VirtualHost> line: ***WSGIScriptAlias / /var/www/html/myapp.wsgi***
Finally, restart Apache with the ***sudo apache2ctl restart*** command.
You just defined the name of the file you need to write within your Apache configuration by using theWSGIScriptAlias directive. Despite having the extension .wsgi, these are just Python applications. Create the /var/www/html/myapp.wsgi file using the command sudo nano /var/www/html/myapp.wsgi. Within this file, write the following application:


def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]

This application will simply print return Hello World! along with the required HTTP response headers.

**Install and configure PostgreSQL:
Do not allow remote connections
Create a new user named catalog that has limited permissions to your catalog application database
***sudo apt-get install postgresql***

We can double check that no remote connections are allowed by looking in the host based authentication file:
***sudo nano /etc/postgresql/9.3/main/pg_hba.conf***
As you can see, the first two security lines specify "local" as the scope that they apply to. This means they are using Unix/Linux domain sockets.
The second two declarations are remote, but if we look at the hosts that they apply to (127.0.0.1/32 and ::1/128), we see that these are interfaces that specify the local machine.


Upon installation, Postgres creates a Linux user called "postgres" which can be used to access the system. We can change to this user by typing:
***sudo su - postgres***

From here, we can connect to the system by typing:
***psql***

Exit out of PostgreSQL and the postgres user by typing the following:
\q
exit


CREATE ROLE catalog WITH login;

We can now create a database owned by "access_role":

CREATE DATABASE demo_application WITH OWNER access_role;

CREATE DATABASE catalog WITH OWNER catalog;

Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

sudo apt-get install git
cd /var/www/html
git clone https://github.com/ruslanml/Udacity-Catalog-Project.git
mv  -v /var/www/html/Udacity-Catalog-Project/* /var/www/html/


***sudo apache2ctl restart***