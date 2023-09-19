deploy django app in aws
How To Set Up Django with Nginx, and Gunicorn on Ubuntu
last veryfied date 26-10-2022
 

https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04

This blog is written based on this digitalocean post. In this digitalocean blog, they do not mention the permission concept if we do not give permission, then the static file will not work.We have a solution

reference  

1) https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04

2) https://www.youtube.com/watch?v=M06YHZ9YUdI

 

how to give user static file access permission 

USER Permission 

ps aux | grep nginx
sudo usermod -a -G ubuntu www-data

Now we need to move to the project root folder after type this command

sudo chown -R :www-data static

 

 

 

 

 

Step 1 — Installing Packages from the Ubuntu Repositories
sudo apt update

sudo apt install python3-pip python3-dev libpq-dev  nginx curl

sudo apt install python-pip python-dev libpq-dev postgresql postgresql-contrib nginx curl

 

Step 2 — Creating a Python Virtual Environment for your Project
sudo -H pip3 install --upgrade pip

sudo -H pip3 install virtualenv

sudo -H pip install --upgrade pip

sudo -H pip install virtualenv

mkdir ~/myprojectdir

cd ~/myprojectdir

virtualenv myprojectenv

source myprojectenv/bin/activate

pip install django gunicorn 

 

Step 3 — Creating and Configuring a New Django Project
django-admin startproject myproject ~/myprojectdir

nano ~/myprojectdir/myproject/settings.py


# ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP', '*']
ex ALLOWED_HOSTS = ['.example.com', '203.0.113.5']

 

~/myprojectdir/myproject/settings.py

. . .

STATIC_URL = '/static/'
import os
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
 

Step 4 — Completing Initial Project Setup
~/myprojectdir/manage.py makemigrations
~/myprojectdir/manage.py migrate

~/myprojectdir/manage.py createsuperuser

~/myprojectdir/manage.py collectstatic

sudo ufw allow 8000

~/myprojectdir/manage.py runserver 0.0.0.0:8000

cd ~/myprojectdir

gunicorn --bind 0.0.0.0:8000 myproject.wsgi

deactivate

Step 6 — Creating systemd Socket and Service Files for Gunicorn
sudo nano /etc/systemd/system/gunicorn.socket

inside, create a [Unit] section to describe the socket, a [Socket] section to define the socket location, and an [Install] section to make sure the socket is created at the right time:

/etc/systemd/system/gunicorn.socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
 

sudo nano /etc/systemd/system/gunicorn.service

 

 

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application

[Install]
WantedBy=multi-user.target
 

sudo systemctl start gunicorn.socket

sudo systemctl enable gunicorn.socket

sudo systemctl status gunicorn.socket

file /run/gunicorn.sock

sudo journalctl -u gunicorn.socket

 

Step 8 — Testing Socket Activation
sudo systemctl status gunicorn

curl --unix-socket /run/gunicorn.sock localhost

sudo systemctl status gunicorn

sudo journalctl -u gunicorn

sudo systemctl daemon-reload

sudo systemctl restart gunicorn

 
Step-9 Configure Nginx to Proxy Pass to Gunicorn
sudo nano /etc/nginx/sites-available/myproject

 

server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/sammy/myprojectdir;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
 

 

 

sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled

sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 8000

sudo ufw allow 'Nginx Full'

 

 

USER Permission 

ps aux | grep nginx
sudo usermod -a -G ubuntu www-data

Now we need to move to the project root folder after typing this command

sudo chown -R:www-data static

Reference

https://www.youtube.com/watch?v=M06YHZ9YUdI

 

 

If you update your Django application, you can restart the Gunicorn process to pick up the changes by running the following:

sudo systemctl restart gunicorn

sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service

sudo nginx -t && sudo systemctl restart nginx

 

 

 

#django #deployment  #aws

 

esay way to getting ssl
https://www.youtube.com/watch?v=0IaHq7JmfIU&t=1157s

https://sleeksoft.in/deploy-django-website-in-ubuntu-server-using-nginx-gunicorn-and-wsgi-with-postgres-db/

 

Final Step : Certbot installtion for enabling SSL
Install snapd

sudo apt install snapdCopy
Execute the following instructions on the command line on the machine to ensure that you have the latest version of snapd

sudo snap install core; sudo snap refresh coreCopy
Run this command on the command line on the machine to install Certbot.

sudo snap install --classic certbotCopy
Execute the following instruction on the command line on the machine to ensure that the certbot command can be run.

sudo ln -s /snap/bin/certbot /usr/bin/certbotCopy
Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

sudo certbot --nginxCopy
When prompted enter the desired value. This will automatically create SSL certificates for the pointed domain.

Now reboot the server.

rebootCopy
Credits:

Digital Ocean
Barath Prabu S
