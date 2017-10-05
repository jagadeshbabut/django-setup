## Setup Django, Nginx and uWSGI on a Ubuntu server - Two User setup

[![sayThanks](https://img.shields.io/badge/Say%20Thanks!-🦉-1EAEDB.svg)](https://saythanks.io/to/jagadeshbabu)

### Login to UBUNTU user

ssh ubuntu@ec2-54-88-129-151.compute-1.amazonaws.com

#### CREATE apps USER
```
sudo adduser apps
sudo su apps #Login to apps user and add ssh keys
mkdir ~/.ssh
echo -e "<SSH Key from your local machine>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

#### REQUIRED PACKAGES/DEPENDENCIES
```
sudo apt-get update && sudo apt-get -y upgrade
sudo apt-get -y install python-dev python-pip nginx vim uwsgi uwsgi-plugin-python
sudo pip install virtualenvwrapper
```

#### DATABASE

##### SQLITE
```
sudo apt-get -y install sqlite3 libsqlite3-dev
```

##### MYSQL
```
wget http://dev.mysql.com/get/mysql-apt-config_0.8.3-1_all.deb (Choose the latest one from https://dev.mysql.com/downloads/repo/apt/)
sudo dpkg -i mysql-apt-config_0.8.3-1_all.deb
sudo apt-get update
sudo apt-get install mysql-server
```

##### POSTGRESQL
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

### SETUP

#### NGNIX
```
sudo rm /etc/nginx/sites-enabled/default
sudo vim /etc/nginx/sites-available/parsel_tongue_nginx.conf 
```
Copy the [parsel_tongue_nginx.conf](https://github.com/jagadeshbabu/django-ubuntu-setup/blob/master/configs/parsel_tongue_nginx.conf) file
```
sudo ln -s /etc/nginx/sites-available/parsel_tongue_nginx.conf /etc/nginx/sites-enabled/parsel_tongue_nginx.conf
sudo service nginx configtest
sudo service nginx restart
```

#### APPLICATION DEPENDENCIES
```
sudo chown apps:www-data -R /var/log/uwsgi
sudo vim /etc/uwsgi/apps-available/parsel_tongue_uwsgi.ini
```
Copy the [parsel_tongue_uwsgi.ini](https://github.com/jagadeshbabu/django-ubuntu-setup/blob/master/configs/parsel_tongue_uwsgi.ini) file
```
sudo ln -s /etc/uwsgi/apps-available/parsel_tongue_uwsgi.ini /etc/uwsgi/apps-enabled/parsel_tongue_uwsgi.ini
```

#### PERMISSIONS FOR UWSGI RESTART
```
sudo visudo (Add the following line)
apps ALL=(ALL) NOPASSWD: /usr/sbin/service uwsgi status, /usr/sbin/service uwsgi start, /usr/sbin/service uwsgi stop, /usr/sbin/service uwsgi restart
ctrl + o
ENTER
ctrl + x
```

exit

### Login to APPS user

ssh apps@ec2-54-88-129-151.compute-1.amazonaws.com

#### APPLICATION DEPENDENCIES
```
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
source ~/.bashrc
```

##### Django Application Setup
```
mkvirtualenv parsel_tongue
pip install django
django-admin startproject parsel_tongue
cd parsel_tongue
echo -e 'STATIC_ROOT = os.path.join(BASE_DIR, "static/")' >> parsel_tongue/settings.py
python manage.py collectstatic
python manage.py migrate
python manage.py createsuperuser
deactivate
```

##### Restart Services
```
sudo service uwsgi start
```
exit
