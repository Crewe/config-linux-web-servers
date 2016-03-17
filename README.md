# Configuring Linux Web Servers

## Server Access
IP and Port for ssh: __52.24.235.146:2200__

## Application URL
http://ec2-52-24-235-146.us-west-2.compute.amazonaws.com/

## Software And Configuration Summary

* Updated repository sources, upgraded installed packages, and removed old/unused packages:
```
# apt-get update && sudo apt-get upgrade && sudo apt-get autoremove
```

* Created user `grader` and give sudo privileges
```
# apt-get install finger
# adduser grader
# touch /etc/sudoers.d/grader
```
* Edit sudoers file and add: `grader ALL=(ALL) PASSWD:ALL`

* Changed to grader user `# su grader`

* Made a SSH key pair with `ssh-keygen` for the grader user and added it to `/home/grader/.ssh/authorized_keys`

* Changed to grader user and only allowed SSH on port 2200 on ufw as well as NTC and HTTP on standard ports and turned on firewall logging
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw logging on
$ sudo ufw enable
```

* Disabled the root user access by setting __AllowRootLogin__ to __no__ in `/etc/ssh/sshd_config`, whitelisted grader user by adding __AllowUsers grader__ and then restarted the service and set __Port__ to __2200__. Then restarted the service.
```
$ sudo service ssh restart
```

* Set time zone to UTC from EST
```
$ sudo ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

* Installed required packages:
```
$ sudo apt-get install git python2.7 python-pip apache2 libapache2-mod-wsgi \
postgresql python-dev
```

* Configured postgreSQL and added a role `catalog` without superuser privileges.
```
$ sudo su - postgres
$ createuser catalog -PRSd
$ exit
```

* Added line `local   item_catalog    catalog                                 md5` to config file, and create a linux user `catalog`
```
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
$ sudo service postgresql restart
$ sudo adduser catalog 
```
* Then configure the database for the catalog role to use, as well as system user.
```
$ sudo su - postgres
$ createdb item_catalog
```
* Cloned repository into apache server root [item-catalog](https://github.com/Crewe/item-catalog) and changed to 'dev-site' branch
* Added .htaccess file to root directory to restrict serving of the .git folder.

_.htaccess_:
```
RedirectMatch 404 /\.git
```

* Installed python extensions: 
```$ sudo apt-get install python-psycopg2 python-flask python-sqlalchemy```
    
* Configured app: `/etc/var/www/html/item-catalog/config.py`
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
* Then and set path to application in _itemcatalog.wsgi_:
```
import sys

# Adds site app to python's path so imports can be used
sys.path.insert(0, '/var/www/html/item-catalog')
from project import app as application
```

Installed pip packages, and RSS:
```
$ sudo pip install bleach oauth2client requests httplib2 glances \
werkzeug==0.8.3 flask==0.9 Flask-login==0.1.3
$ cd PyRSS2Gen/
$ sudo python setup.py install
$ cd ..
```

* Updated client_secrets.json file to have the server host in the Google developer console setup database with `python database_setup.py`, and seeded database with `python seed_database.py`.

* Configured apache2 site conf to use the WSGI alias to point to the repo directory
and enabled mod_rewrite on apache so that the IP redirects to the domain name.
_000-default.conf_:
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

* Enable Apache modules and restart apache
```
$ sudo a2enmod rewrite
$ sudo a2enmod wsgi
$ sudo service apache2 restart
```

* Added `127.0.1.1 ip-10-20-2-228` to */etc/hosts* file.
* Restarted ssh server
* Updated repository sources, upgraded installed packages, and removed old/unused packages again.

#### Post app config setup of extras:
```
$ sudo apt-get install glances fail2ban unattended-upgrades \
update-notifier-common apt-listchanges
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

* Edit _jail.local_ sections and add new ones with the following settings:
```
[ssh]

enabled   = true
port      = 2200
...

[apache]

enabled   = true
maxretry  = 3
findtime  = 600
...

# Used to ban clients that are searching for scripts on the website to execute and exploit
[apache-noscript]

enabled  = true
...

# Block clients who are attempting to request unusually long and suspicious URLs. These 
# are often signs of attempts to exploit Apache by trying to trigger a buffer overflow
[apache-overflows]

enabled  = true
...

# To stop some known malicious bot request patterns
[apache-badbots]

enabled  = true
port     = http,https
filter   = apache-badbots
logpath  = /var/log/apache*/*error.log
maxretry = 2

# No files are being hosted from the users home directory
[apache-nohome]

enabled  = true
port     = http,https
filter   = apache-nohome
logpath  = /var/log/apache*/*error.log
maxretry = 2
```

* Retsart the service with: `$ sudo service fail2ban restart`

* Edit _/etc/apt/apt.conf.d/50unattended-upgrades_
```
// Automatically upgrade packages from these (origin:archive) pairs
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}-security";
        "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};

...

// Do automatic removal of new unused dependencies after the upgrade
// (equivalent to apt-get autoremove)
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Automatically reboot *WITHOUT CONFIRMATION*
//  if the file /var/run/reboot-required is found after the upgrade 
Unattended-Upgrade::Automatic-Reboot "true";

// If automatic reboot is enabled and needed, reboot at the specific
// time instead of immediately
//  Default: "now"
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
```
* Run `$ sudo dpkg-reconfigure -plow unattended-upgrades`. Or create the 
`/etc/apt/apt.conf.d/20auto-upgrades` file and add the following to it to
enable daily checks for upgrades:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

* Edit _/etc/apt/listchanges.conf_:
```
[apt]
frontend=pager
email_address=root
confirm=1
save_seen=/var/lib/apt/listchanges.db
which=both
```

* Reboot system: `$ sudo reboot`

* Run `$ glances` for current system and process statuses

### Third Party Sources:

1. https://wiki.postgresql.org/wiki/PostgreSQL_For_Development_With_Vagrant
1. http://stackoverflow.com/a/17916515
1. http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/
1. http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
1. http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2
1. http://stackoverflow.com/a/18664239
1. http://www.postgresql.org/docs/9.3/static/index.html
1. http://stackoverflow.com/a/11651783
1. http://stackoverflow.com/questions/869092/how-to-enable-mod-rewrite-for-apache-2-2
1. http://askubuntu.com/a/235264
1. https://wiki.debian.org/UnattendedUpgrades
1. http://blog.mafr.de/2015/02/26/ubuntu-unattended-upgrades/
1. https://pypi.python.org/pypi/Glances
1. http://www.fail2ban.org/wiki/index.php/Main_Page
1. https://www.digitalocean.com/community/tutorials/how-to-protect-an-apache-server-with-fail2ban-on-ubuntu-14-04
1. https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
