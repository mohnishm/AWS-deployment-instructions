# AWS-deployment-instructions
Instructions for deploying to AWS

# get requirements.txt
$ pip freeze > requirements.txt

$ touch .gitignore
`add these lines to gitignore files`
    *.pyc
    venv/

# then push your code to github

# create EC2 instance
    - edit security groups:
        - HTTP, HTTPS, 80, 22, 8000 under TCP protocol

    - download key-value pair
    - use it to ssh into instance


# connect to your instance

# server configuration
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get update

$ sudo apt-get install python3
$ sudo apt-get install python3-pip
$ sudo apt-get install python3-dev
$ sudo apt-get install nginx
$ sudo apt-get install git

$ sudo pip3 install virtualenv

# install redis
$ wget http://download.redis.io/redis-stable.tar.gz
$ tar xvzf redis-stable.tar.gz
$ cd redis-stable
$ make

`then copy redis-server and redis-cli executables under /usr/local/bin.`
sudo cp src/redis-server /usr/local/bin
sudo cp src/redis-cli /usr/local/bin


$ sudo mkdir /etc/redis
$ sudo mkdir /var/redis
$ sudo cp utils/redis_init_script /etc/init.d/redis_6379
$ sudo cp redis.conf /etc/redis/6379.conf
$ sudo mkdir /var/redis/6379

`in /etc/redis/6379.conf`
Edit the configuration file, making sure to perform the following changes:
    - Set daemonize to yes (by default it is set to no).
    - Set the logfile to /var/log/redis_6379.log
    - Set the dir to /var/redis/6379 (very important step!)

$ sudo update-rc.d redis_6379 defaults
$ sudo /etc/init.d/redis_6379 start


# install mysql
$ sudo apt-get update
$ sudo apt-get install mysql-server
`start mysql service`
$ sudo systemctl start mysql
`launch at reboot`
$ sudo systemctl enable mysql

$ sudo mysql
    USE mysql;
    CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost';
    UPDATE user SET plugin='auth_socket' WHERE User='username';
    FLUSH PRIVILEGES;
    exit;

$ sudo mysql
    USE mysql;
    UPDATE user SET plugin='mysql_native_password' WHERE    User='your_yourname_here';
    FLUSH PRIVILEGES;
    exit;

$ sudo service mysql restart

$ sudo mysql
    CREATE DATABASE ipl_dbname

# clone your project
$ git clone ~repo_name

$ cd repo_name
$ virtualenv venv

# activate your virtualenv
$ source venv/bin/activate
$ sudo apt-get install python3.6-dev 
$ sudo apt-get install libmysqlclient-dev
$ pip install -r requirements.txt
$ pip install bcrypt
$ pip install django-extensions
$ pip install gunicorn

# remeber your project name and repo name
project name = project_name
repo name = repo_name

# change your settings
$ sudo nano settings.py
`modify these lines`
Debug = False
ALLOWED_HOSTS = ['localhost','127.0.0.1', 'your_public_ip']
DATABASES = 'HOST'='', 'PORT'=''
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

$ python manage.py collectstatic

# gunicorn
$ gunicorn --bind 0.0.0.0:8000 project_name.wsgi:application
`then stop this process`
`then deactivate virtual env`

# setup gunicorn to run as a service
$ sudo nano /etc/systemd/system/gunicorn.service
`insert this text into the file`
[Unit]
Description=gunicorn daemon
After=network.target
[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/repo_name
ExecStart=/home/ubuntu/repo_name/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 project_name.wsgi:application
[Install]
WantedBy=multi-user.target


# enable and boot into these commands
$ sudo systemctl daemon-reload
$ sudo systemctl start gunicorn
$ sudo systemctl enable gunicorn

# setup Nginx
$ sudo nano /etc/nginx/sites-available/project_name
`add the following into this file`
server{
    listen 80;
    server_name your_public_ip;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/repo_name;
    }
    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:8000;
    }
}

`go to the root dir of linux`
$ sudo ln -s /etc/nginx/sites-available/project_name /etc/nginx/sites-enabled
$ sudo nginx -t



# finally
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo service nginx restart
$ sudo systemctl daemon-reload
$ sudo systemctl start gunicorn
$ sudo systemctl enable gunicorn

f your landing page is loading(If you have one) but not your json and charts that is you're getting a "Server error(500)",
then try this..
sudo nano /etc/systemd/system/gunicorn.service
and add this to ExecStart,
ExecStart=/home/ubuntu/repo_name/env/bin/gunicorn --error-logfile /var/log/gunicorn.log --access-logfile - --workers 3  --bind 0.0.0.0:8000 project_name.wsgi:application

sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn

