Linux Server Configuration
-Ubuntu-apache2-wsgi-Flask

#requires:
1. Ubuntu and root user
2. Internet access

#IP and port:
52.10.87.76
ssh:2200 (limit repeat unsuccessful attempts)
http:80

#address
http://ec2-52-10-87-76.us-west-2.compute.amazonaws.com/
gplus login is not adapted to this site.
github login is adapted and working.
please only use this site address if you want to login with github account.

#to be installed
##apt-get:
apache2
libapache2-mod-wsgi
postgresql
python-pip
python-psycopg2
libpq-dev
git
##pip:
Flask-SQLAlchemy
Flask-Migrate

#cron jobs
/etc/cron.monthly/update (update sofetwares monthly)
/etc/cron.daily/ntpdate (correct time daily)

#Resources:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

#Steps:
1. connect to the server with root user.
2. >adduser grader
   password "123456".
3. >nano /etc/sudoers.d/grader
   type:
      grader ALL=(ALL) NOPASSWD:ALL
   save and exit. 
4. >passwd -e grader
   make the simple password expire.
5. >apt-get update
   >apt-get upgrade
   update all the installed softwares.
   >nano /etc/cron.monthly/update
    type:
      apt-get update
      apt-get upgrade
    save and exit
   setup a cron job to update the softwares routinely.
6. >nano /etc/ssh/sshd_config
   change the Port from 22 to 2200.
   >service ssh restart
7. configure firewall
   1) >ufw status
      check firewall status
   2) >ufw default deny imcoming
      close all incoming ports
   3) >ufw default allow outgoing
   4) >ufw allow 2200/tcp
      >ufw allow http
      >ufw allow ntp
      >ufw limit 2200/tcp (restrict repeat unsuccessful ssh logins)
   5) >ufw enable
      >ufw status
      start and check the firewall
8. set up UTC timezone
   >dpkg_reconfigure tzdata
     choose "None of Above"
     then choose "UTC"
   >nano /etc/cron.daily/ntpdate
     type:
       ntpdate ntp.ubuntu.com
     save and exit
9. start the cron jobs
   >service cron start
10. install apache2 and set up mod-wsgi
   >apt-get -y install apache2
   >apt-get -y install libapache2-mod-wsgi
11. install postgresql
   >apt-get -y install postgresql
    add user catalog and database catalog
   >su postgres
   >psql
    open the postgresql interface
   >CREATE DATABASE catalog;
   >CREATE USER catalog WITH PASSWORD "tiger";
12. install required python modules for flask-alchemy
   >apt-get -y install python-pip
   >pip install Flask-SQLAlchemy Flask-Migrate
   >apt-get -y install python-psycopg2
   >apt-get -y install libpq-dev
13. install git and clone project
   >apt-get install git
   >git config --global user.name "username"
   >git config --global user.email "email"
   >git clone https://github.com/baccan0/CatalogApp.git /var/www/html/Catalog/
   >chmod 700 /var/www/thml/Catalog/.git (restrict file permissions)
14. build the database
   1) change the create_engine in file "/var/www/html/Catalog/database_setup.py" and "/var/www/html/Catalog/populate.py":
      engine = create_engine("postgresql+psycopg2://postgres:/localhost:5432/catalog")
   2) su postgres
   3) execute those two files:
      >cd /var/www/html/Catalog/
      >python database_setyp.py
      >python populate.py (fill some entries into the database)
   4) >exit (back to root)
   5) change the create_engine in file "/var/www/html/Catalog/populate.py" and "/var/www/html/Catalog/main.py"
       engine = create_engine("postgresql+psycopg2://catalog:tiger/localhost:5432/catalog")
15. grant the database privilleges to user catalog:
   >su postgres
   >psql
   >\c catalog;
   >GRANT SELECT,DELETE,UPDATE,INSERT ON webuser TO catalog;
   >GRANT SELECT,DELETE,UPDATE,INSERT ON catalog TO catalog;
   >GRANT SELECT,DELETE,UPDATE,INSERT ON item TO catalog;
   >GRANT SELECT,UPDATE ON webuser_id_seq TO catalog;
   >GRANT SELECT,UPDATE ON catalog_id_seq TO catalog;
   >GRANT SELECT,UPDATE ON item_id_seq TO catalog;
16. configure wsgi module
   >nano /etc/apache2/sites-enabled/000-default.conf
   in <VirtualHost *.80> ... </VirtualHost>
   type in:
     ServerName 52.10.87.76
     DocumentRoot /var/www/html
     LogLevel info
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
     WSGIScriptAlias / /var/www/html/CatalogApp/myapp.wsgi
     <Directory /var/www/html/CatalogApp>
       Order allow,deny
       Allow from all
     </Directory>
     Alias /static /var/www/html/CatalogApp/static
     <Directory /var/www/html/CatalogApp/static/>
       Order allow,deny
       Allow from all
     </Directory>
    >nano /var/www/html/CatalogApp/myapp.wsgi
     type in:
      import sys
      import os
      os.chdir('/var/www/html/CatalogApp')
      sys.path.insert(0, '/var/www/html/CatalogApp')
      from main import app as application
     save and exit
17. restart apache2 server
    >service apache2 restart
18. close root login
   1) generate a key pair for grader
      >ssh-keygen -f grader
   2) put the grader.pub into /home/grader/.ssh/authorized_keys file on the server
   3) put the private key into folder ~/.ssh/ on local computer
   4) change the sshd_config
      type:
        PermitRootLogin no
        AllowUsers grader
   5) service ssh restart
19. exit and connect to server as grader.
