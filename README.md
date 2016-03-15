# Configuring Linux Web Servers

## Summary of steps

### Server Access
ssh://52.24.235.146:2200

### Application URL
http://ec2-52-24-235-146.us-west-2.compute.amazonaws.com/

### Software And Configuration Summary


1. Created user 'grader' and give sudo privileges
```
# apt-get install finger
# adduser grader
# cp /etc/sudoers.d grader.d
# cp /etc/sudoers.d /etc/sudoers/grader.d
```
Then edit the file to have the correct user name.

1. Made a SSH key pair for the grader user and added it to /home/grader/.ssh/authorized_keys

1. Changed to grader user and only allowed SSH on port 2200 on ufw as well as NTC and HTTP on standard ports and turned on firewall logging
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw status
$ sudo ufw enable
$ sudo ufw allow ntp
$ sudo ufw logging on
$ sudo ufw enable
```

* Disabled the root user access by setting *AllowRootLogin* to *no* in `/etc/ssh/sshd_config`, whitelisted grader user by adding *AllowUsers grader* and then restarted the service.
```
$ sudo /etc/init.d/ssh restart
```

1. Updated repository sources, upgraded installed packages, and removed old/unused packages:
```
$ sudo apt-get update && sudo apt-get upgrade && sudo apt-get autoremove
```

1. Set time zone to UTC from EST
```
$ sudo ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

Installed required packages:
```$ sudo apt-get install git python2.7 and python-pip libapache2-mod-wsgi \
postgresql```

Installed and configured apache to use mod_wsgi (libapache2-mod-wsgi)
Installed and configured postgreSQL and added a role 'catalog' without superuser privileges, as well as system user with the same name.

```
$ sudo su - postgres
$ createuser catalog -PRSd
$ exit
```
Add line `local   item_catalog    catalog                                 md5` to config file, and create a linux user `catalog`
```
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
$ sudo adduser catalog then configure the database for the catalog role to use.
$ sudo su - postgres
$ createdb item_catalog
$ psql
# \c item_catalog
# grant all on item_catalog to catalog;
# \q
# exit
```
Cloned repository into apache server root [https://github.com/Crewe/item-catalog](item-catalog) and changed to 'dev-site' branch
Added .htaccess file to root directory to restrict serving of the .git folder.
** .htaccess: **
```
RedirectMatch 404 /\.git
```

installed python extensions: 
    python-psycopg2
    python-flask
    python-sqlalchemy
    
configured app `/etc/var/www/html/item-catalog/config.py`
```
# config.py
# Configuation file for the database connection

database = 'item_catalog'
username = 'catalog'
password = 'password_here'
client_secrets_path = '/var/www/html/item-catalog/client_secrets.json'


def connectionString():
    return "postgresql+psycopg2://" + username + ":" + password + "@/" + database


def clientSecrets():
    return client_secrets_path
```

    ** itemcatalog.wsgi: **
```
import sys

# Adds site app to python's path so imports can be used
sys.path.insert(0, '/var/www/html/item-catalog')
from project import app as application
```

installed pip packages, and RSS:
```
$ sudo pip install bleach oauth2client requests httplib2 \
werkzeug==0.8.3 flask==0.9 Flask-login==0.1.3
$ cd PyRSS2Gen/
$ sudo python setup.py install
$ cd ..
```
Updated client_secrets.json file to have the server host in the Google developer console
setup database with database_setup.py
seeded database with seed_database.py
Configured apache2 site conf to use the WSGI alias to point to the repo directory
and enabled mod_rewrite on apache so that the IP redirects to the domain name.
```
$ sudo a2enmod rewrite
$ sudo 
```
** 000-default.conf: **
```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	ServerName ec2-52-24-235-146.us-west-2.compute.amazonaws.com 

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
    RewriteEngine On
    RewriteCond %{HTTP_HOST} !^ec2-52-24-235-146.us-west-2.compute.amazonaws.com$
    RewriteRule /.* http://ec2-52-24-235-146.us-west-2.compute.amazonaws.com/ [R]
    WSGIScriptAlias / /var/www/html/item-catalog/itemcatalog.wsgi
</VirtualHost>
```

Added `127.0.1.1 ip-10-20-2-228` to */etc/hosts* file.

restarted ssh server
restarted apache server


== Third Party Sources:

https://wiki.postgresql.org/wiki/PostgreSQL_For_Development_With_Vagrant
http://stackoverflow.com/a/17916515
http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/
http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2
http://stackoverflow.com/a/18664239
http://www.postgresql.org/docs/9.3/static/index.html
http://stackoverflow.com/a/11651783
http://stackoverflow.com/questions/869092/how-to-enable-mod-rewrite-for-apache-2-2
