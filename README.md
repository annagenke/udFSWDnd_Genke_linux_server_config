# Project 5: Linux Server Configuration steps
Below are the server configuration steps for project 5 of the 
Full Stack Web Development Nanodegree.
#### Details
* IP address: 54.167.166.145
* http://54.167.166.145.xip.io/
* ssh port: 2200
### getting a lightsail instance
1. Go to [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) and create an account.
2. Click Create instance
3. Select Linux/Unix platform
4. Select OS Only and Ubuntu as blueprint
5. Select the instance plan that includes the free trial
6. Name your instance and click create.

### ssh into your instance
1. create a new rsa key or download the default
2. create a file called `lightsail_rsa` under you `~/.ssh directory`. 
3. put the key from your downloaded .pem file into your new `lightsail_rsa` file
4. $ chmod 600 ~/.ssh/lightsail_key.rsa to set the permission to owner only.
5. SSH into the instance:  `ssh -i ~/.ssh/lightsail_rsa ubuntu@54.167.166.145`


### create grader user
1. `sude adduser grader`
2. `sudo touch /etc/sudoers.d/grader`
3. `sudo vi /etc/sudoers.d/grader`
4. add line `grader ALL=(ALL:ALL) ALL`, save and exit
5. on your local machine, generate a key-pair named grader_rsa using `ssh-keygen`. you may add a password.
6. you will now have 2 new files, `grader_rsa` and `grader_rsa.pub`
7. paste the contents of `grader_rsa.pub` into `/home/grader/.ssh/authorized_keys` (you may need to create this file)
8. test that you can ssh as grader

### 

### update all packages
run the following to update packages
1. `sudo apt-get update`
2. `sudo apt-get upgrade`
3. `reboot`

### change ssh ports
1. use sudo vi /etc/ssh/sshd_config
2. find the line that says Port=22 and change to Port=2200
3. save and close the file
4. `sudo service ssh restart` to restart ssh
5. From lightsail dashboard under networking tab add custom firewall rule for tcp connection on port 2200.

### Configure Uncomplicated Firewall (UFW)
You can check firewall status with sudo ufw status
1. set the defaults to deny incoming and allow outgoing 
    * `sudo ufw default deny incoming`
    * `sudo ufw default allow outgoing`
2. use the following to allow SSH (port 2200), HTTP (port 80), and NTP (port 123).
    * `sudo ufw allow 2200/tcp`
    * `sudo ufw allow www`
    * `sudo ufw allow 123/udp`
3. Deny connection on port 22:
    * `sudo ufw deny 22`
4. enable firewall
    * `sudo ufw enable`

### configure the local timezone to UTC
If the default timezone is not UTC:
* `sudo dpkg-reconfigure tzdata`
* select "none of the above"
* select UTC

The time zone is saved in /etc/timezone

### Install and configure Apache to serve a Python mod_wsgi application
* `sudo apt install apache2`

If installed correctly, going to 54.167.166.145 on a wed browser 
will show the ubuntu apache default page

Install mod-wsgi:
* `sudo apt install libapache2-mod-wsgi-py3`
* `sudo apt install python-dev`

enable mod-wsgi:
* `sudo a2enmod wsgi`

restart apache: `sudo service apache2 restart`

### Install and configure postgreSQL
* `sudo apt-get install PostgreSQL`
* `sudo apt-get install ppostgresql-contrib`

the config file is located at `/etc/postgresql/9.5/main/pg_hba.conf` Do not allow remote connections
##### Create catalog database user
1. Switch to the postgreSQL user 
    * `sudo su postgres`
2. Open the postgres console
    * `psql`
    * `CREATE ROLE catalog WITH PASSWORD 'catalog';`
    * `CREATE DATABASE catalog WITH OWNER catalog;` 
### Install git
`sudo apt-get install git`  

## Deploy the Item Catalog project
I used this [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
for reference.

###  Clone and setup your Item Catalog
1. create the directory catalog
`sudo mkdir /var/www/catalog`
2. `cd /var/www/catalog`
3. Clone the catalog repo here `sudo git clone https://github.com/YOUR/REPO.git`
4. create wsgi file `sudo nano flaskapp.wsgi` with the following
    ```coffeescript
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'Add your secret key'

    ```  
5. install dependency libraries for the catalog app
    ```coffeescript
    sudo apt-get install python-pip
    sudo pip install httplib2
    sudo pip install requests
    sudo pip install --upgrade oauth2client
    sudo pip install sqlalchemy
    sudo pip install flask
    sudo apt-get install libpq-dev
    sudo pip install psycopg2
    ```

Changes I had to make to the code in my repo are as follows:
1. change the engine to use the database we just created 
    * `engine=create_engine(postgresql://catalog:catalog@localhost/catalog...`
    
#### set up virtualenv
sudo pip install virtualenv 
sudo virtualenv venv
source venv/bin/activate 

### create VirtualHost
1. create file `sudo touch /etc/apache2/sites-available/catalog.conf`
2. add the following:
    ```    
    <VirtualHost *:80>
                    ServerName 54.167.166.145
                    ServerAdmin ubuntu@54.167.166.145
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
3. enable virtule host `sudo a2ensite catalog`
4. restart apache

### Modify google oauth
1. Add 'xip.io' to you Authorized Domains

2. In Authorized Javascript add `http://54.167.166.145.xip.io`

3. In Authorized Redirect Urls add:
    ```
      http://54.167.166.145.xip.io/gconnect	
      http://54.167.166.145.xip.io/oauth2callback	
      http://54.167.166.145.xip.io/login
    ```
---
##### tips
If you are unfamiliar with vi/vim:
* :i starts insert
* :w save file
* :q close file