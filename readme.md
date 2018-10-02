# Linux Server Configuration
## Udacity FSDN Final Project

This repository is a brief description of the final project for Udacity's Full Stack Nanodegree Program. It's a Linux Server configuration using AWS Lightsail, Apache and other technologies (see more below).

**VM specs:** AWS Lightsail instance (Ubuntu 512 MB RAM, 1 vCPU, 20 GB SSD)
<br>
**IP address:** http://18.235.3.186/

### Technologies used & third-party resources
* Amazon Lightsail
* Ubuntu
* Uncomplicated Firewall (UFW)
* PostgreSQL
* Apache2


### Packages installed in VM
* finger
* python-pip
* flask
* mod_wsgi (Apache)
* oauth2client
* redis
* passlib
* sqlalchemy
* flask-sqlalchemy
* psycopg2-binary
* requests

----

## Configurations made


* Created new Ubuntu instance on Amazon Lightsail


* On Lightsail console, configured Firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).


* On new Ubuntu VM, configured UFW to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123), like so:
    ```
    sudo ufw default deny incoming
    ```
    ```
    sudo ufw default allow outgoing
    ```
    ```
    sudo ufw allow 2200/tcp
    ```
    ```
    sudo ufw allow www
    ```
    ```
    sudo ufw allow 123/udp
    ```


* Added a new user named 'grader'
    ```
    sudo adduser grader
    ```


* Gave 'grader' permission to sudo
    1. On terminal, typed:
    ```
    sudo visudo
    ```
    2. Copied line `root   ALL=(ALL:ALL) ALL` from file and pasted below it
    3. Changed the second line to `grader   ALL=(ALL:ALL) ALL`


* Created SSH key-pair for grader (private key available on "Notes to Reviewer", posted upon submission), and added key to `~/.ssh/authorized_keys`.


* Configured local timezone to UTC
    ```
    sudo timedatectl set-timezone
    ```
* Installed Apache
    ```
    sudo apt-get install apache2
    ```


* Installed mod_wsgi
    ```
    sudo apt-get install libapache2-mod-wsgi
    ```


* Installed PostgreSQL
    ```
    sudo apt-get install postgresql
    ```


* Configured PostgreSQL to disable remote connections, by commenting IPv6 line on *pg_hba.conf* file
    ```
    sudo nano /etc/postgresql/9.5/main/pg_hba.conf
    ```


* Created a database user named 'catalog', and a database named 'catalogdatabase':
    ```
    $ sudo su postgres
    ```
    ```
    postgres@ip-172-26-4-50:/home/ubuntu$ psql
    psql (9.5.14)
    Type "help" for help.

    postgres=# CREATE USER catalog WITH PASSWORD '******';
    CREATE ROLE
    postgres=# CREATE DATABASE catalogdatabase OWNER catalog
    postgres=# CREATE DATABASE catalogdatabase OWNER catalog;
    CREATE DATABASE
    ```

* Restarted Postgresql service
    ```
    sudo service postgresql restart
    ```

* Cloned catalog project into `/var/www/html`


* made modifications to *database_setup.py* and *lotsofitems.py* to setup and populate PostgreSQL database, change the lines that go like this:

    ```
    engine = create_engine('sqlite:///catalogdatabase.db')
    ```
    to this:
    ```
    engine = create_engine('postgresql://catalog:******@localhost/catalogdatabase')
    ```


* made modifications to *app.py* file
    1. `create_engine()` now points to postgresql database
    2. app.secrect_key is set outside `if __name__ == '__main__'` code block
    3. app.run() has no arguments


* created *myapp.wsgi* file to handle application, like so:
    ```
    import sys
    sys.path.insert(0, '/var/www/myapp')

    from app import app as application
    ```

* configured `/etc/apache2/sites-enabled/000-default.conf ` file to contain these lines:
    ```
    WSGIDaemonProcess myapp user=ubuntu group=ubuntu threads=5 home=/var/www/myapp/
        WSGIScriptAlias / /var/www/myapp/myapp.wsgi

         <directory /var/www/myapp>
                 WSGIProcessGroup myapp
                 WSGIApplicationGroup %{GLOBAL}
                 WSGIScriptReloading On
                 Order deny,allow
                 Allow from all
        </directory>
    ```
    right before closing `</VirtualHost>` tag.

* restarted apache2 service
    ```
    sudo service apache2 restart
    ```
* **DONE!**
