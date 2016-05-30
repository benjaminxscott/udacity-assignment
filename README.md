> public URL: `ec2-52-10-122-233.us-west-2.compute.amazonaws.com`

# access the deployment image
`ssh -p 2200 -i ~/.ssh/udacity_key.rsa root@52.10.122.233`

# deployment as root
- create grading user 
> useradd -U -mgrader -s/bin/bash grader 

- add `grader` to wheel using `visudo`
- Configure the local timezone to UTC 0 
 > dpkg-reconfiguretzdata and set timezone to 'other / UTC' 

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

- install app dependencies
> apt-get install git python pip sqlite3 

- install python dependencies 
> pip install -r requirements.txt 

- deploy and set perms on secret keys 
> scp $destination client_secrets.json && chmod 705 client_secrets.json

- Install the app 
> git clone https://github.com/bschmoker/goldlist.git /var/www/deploy_goldlist 

- hide git repo from public view
> rm -rf /var/www/deploy_goldlist/.git/

- [Install webserver dependencies](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
> apt-get install apache2 
> apt-get install libapache2-mod-wsgi

- create a dedicated user to run the app  
 > useradd -U -m catalog 
 > chown -R catalog:catalog /var/www/deploy_goldlist

- create a WSGI config for the app
 >echo -e "import sys\n sys.path.insert(0,"/var/www/deploy_goldlist")\nfrom goldlist import goldlist application=goldlist">/var/www/deploy_goldlist/goldlist.wsgi

- create an apache vhost with
```
<VirtualHost*>ServerNameec2­52­10­122­233.us­west­2.compute.amazonaws.com
WSGIDaemonProcessgoldlistuser=cataloggroup=catalogthreads=5WSGIScriptAlias/
/var/www/deploy_goldlist/goldlist.wsgi<Directory/var/www/goldlist>WSGIProcessGroupgoldlist
WSGIApplicationGroup%{GLOBAL}Orderdeny,allowAllowfromall</Directory></VirtualHost> under
/etc/apache2/sites­available/000­default.conf
```




