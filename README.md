# Configuring Linux Web Servers

Summary of steps

Server access:
ssh://52.24.235.146:2200

Application URL:
http://ec2-52-24-235-146.us-west-2.compute.amazonaws.com/

Software And Configuration Summary:

Changed  and only allowed SSH on port 2200 on ufw as well as NTC and HTTP on standard ports and turned on firewall logging
created user 'grader' and gave sudo privileges
updated repository sources
upgraded installed packages
removed old/unused packages
made a SSH key pair for the grader user and added it to ~/.ssh/authorized_keys
Set time zone to UTC from EST
installed git, python2.7, and python-pip
Installed and configured apache to use mod_wsgi (libapache2-mod-wsgi)
Installed and configured postgreSQL and added a role 'catalog' without superuser privileges, as well as system user with the same name.
Modified postgres pg_hba.conf file so catalog user could use password
Created database 'item_catalog' to which the catalog role was given access to.

Cloned repository into apache server root [https://github.com/Crewe/item-catalog](item-catalog) and changed to 'dev-site' branch
Added .htaccess file to server root to restrict serving of the .git folder.
installed python extensions: 
    python-psycopg2
    python-flask
    python-sqlalchemy
configured app in: 
    config.py
    itemcatalog.wsgi
installed pip packages:
    bleach
    oauth2client 
    requests 
    httplib2
    werkzeug==0.8.3
    flask==0.9
    Flask-Login==0.1.3
installed PyRSS2Gen RSS package
Configured apache2 site conf to use the WSGI alias to point to the repo directory
Updated client_secrets.json file to have the server host in the Google developer console
setup database with database_setup.py
seeded database with seed_database.py
enabled mod_rewrite on apache so that the IP redirects to the domain name

===== disabled remote login by root ====

restart ssh server
restarted apache server


Third Party Sources:

https://wiki.postgresql.org/wiki/PostgreSQL_For_Development_With_Vagrant
http://stackoverflow.com/a/17916515
http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/
http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2
http://stackoverflow.com/a/18664239
http://www.postgresql.org/docs/9.3/static/index.html
http://stackoverflow.com/a/11651783
http://stackoverflow.com/questions/869092/how-to-enable-mod-rewrite-for-apache-2-2
