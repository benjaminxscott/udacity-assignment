
## Access the app
`ec2-52-10-122-233.us-west-2.compute.amazonaws.com`

- access the server as `root`:
> ssh -p 2200 -i ~/.ssh/udacity_key.rsa root@52.10.122.233

- access the server as `grader`:
> ssh -i ~/.ssh/grader_rsa -p 2200 grader@52.10.122.233

## Deploy app server

- build Ubuntu 14.04 image
> on AWS, heroku, etc

- create `grader` user
> useradd -U -m grader -s/bin/bash grader 

- create a complex password for `grader` user
> passwd `grader` 
> <input password>

- generate SSH keypair for `grader` user
> su grader
> ssh-keygen -t rsa -b 4096 -C "grader" 
> save as `grader-id_rsa`
> mkdir ~/.ssh chmod 700 ~/.ssh
> cat grader-id_rsa.pub >> ~/.ssh/authorized_keys  
> chmod 600 ~/.ssh/authorized_keys 

- add `grader` user public key to SSH daemon
> cat grader-id_rsa.pub >> ~/.ssh/authorized_keys

- add `grader` to sudoer group as in [this guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/ch02s03.html)
> `visudo`

- Configure the local timezone to UTC 0 
 > dpkg-reconfiguretzdata -> set timezone to 'other / UTC' 

- Change default SSH port from 22 to 2200 
> Edit 'Port' setting under /etc/ssh/sshd_config 

- Restart SSH service
> service ssh restart

- upgrade *nix libraries
 > apt-get update && upgrade

- bounce the box
> shutdown -r now

- configure firewall to deny incoming connections besides SSH, HTTP and NTP

```
ufw allow 2200
ufw allow 80
ufw allow 123
ufw enable
```

## Install the app

- Get the latest app (without git history)
> git clone --depth 1 --branch=master https://github.com/bschmoker/goldlist.git /var/www/deploy_goldlist 

- Copy app secrets
> mkdir /etc/goldlist
> cp client_secrets.json /etc/goldlist
> chmod 705 /etc/goldlist/client_secrets.json

- create a dedicated user to run the app  
 > useradd -U -m catalog 
 > chown -R catalog:catalog /var/www/deploy_goldlist


## Install dependencies
- install app dependencies
> apt-get install git python pip postgresql 

- install python dependencies 
> pip install -r requirements.txt 

## Install the app database

- start database daemon (if not already running)
> service postgresl start

- create postgres user `catalog` in DB
> su postgres
> createuser --pwprompt --createdb catalog
> <enter password>

- create database and assign to `catalog` user
> createdb listings --owner catalog

- generate database tables
> python db_setup.py

- (optional) generate sample data
> python db_seed.py

## Running the App
- [Install webserver dependencies](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
> apt-get install apache2 
> apt-get install libapache2-mod-wsgi

- create a WSGI config for the app
 >echo -e "import sys\n sys.path.insert(0,"/var/www/deploy_goldlist")\nfrom goldlist import goldlist application=goldlist">/var/www/deploy_goldlist/goldlist.wsgi


- create an apache vhost to handle incoming requests
```
<VirtualHost*>ServerNameec2­52­10­122­233.us­west­2.compute.amazonaws.com
WSGIDaemonProcessgoldlistuser=cataloggroup=catalogthreads=5WSGIScriptAlias/
/var/www/deploy_goldlist/goldlist.wsgi<Directory/var/www/goldlist>WSGIProcessGroupgoldlist
WSGIApplicationGroup%{GLOBAL}Orderdeny,allowAllowfromall</Directory></VirtualHost> under
/etc/apache2/sites­available/000­default.conf
```




