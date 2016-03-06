# Configuring Linux Web Servers

# Requirements
- [Vagrant][1]
- [VirtualBox][2]


# Environment Setup
1. Create a new folder on your computer where you’ll store your work for this course, then open that folder within your terminal.
1. Type `vagrant init ubuntu/trusty64` to tell Vagrant what kind of Linux virtual machine you would like to run.
1. Type `vagrant up` to download and start running the virtual machine
1. Install python2.7, pip, postgresql
`apt-get -qqy update
apt-get -qqy install postgresql python-psycopg2
apt-get -qqy install python-flask python-sqlalchemy
apt-get -qqy install python-pip
pip install bleach
pip install oauth2client
pip install requests
pip install httplib2`


[1]: https://www.vagrantup.com/
[2]: https://www.virtualbox.org/
