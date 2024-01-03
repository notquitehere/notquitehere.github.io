---
title: "Deploying a React and Flask project on Ubuntu 18.04 using Nginx and uWSGI"
date: 2022-04-20
published: true
excerpt: A walkthrough of the process of setting up a React and Flask project on Ubuntu 18.04 using Nginx and uWSGI
cover:
tags: python react flask ubuntu
---

Prerequities:

- A server running Ubuntu 18.04 with a non-root user and firewall enabled. DigitalOcean have a good [setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).
- A registered domain name with the correct DNS settings
- An existing React App


## Nginx

### Install

```shell
sudo apt install nginx
```

### Adjust firewall to allow traffic

```shell
sudo ufw app list
```

Output:

```shell
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

**Nginx Full** opens both port 80 (HTTP) and 443 (HTTPS) if you don't need both of these you should open only the one you do need using either **Nginx HTTP** or **Nginx HTTPS**. As we're going to add an SSL certificate later we're going to set it to full for the moment.

```shell
sudo ufw allow 'Nginx Full'
```

You can revoke unnecessary access later using `sudo ufw delete allow <app>`.

Verify the change:

```shell
sudo ufw status
```

Output:

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

### Check Nginx is running

```shell
systemctl status nginx
```

Output:

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-04-20 11:01:26 UTC; 44min ago
       Docs: man:nginx(8)
    Process: 2300 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2312 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2313 (nginx)
      Tasks: 2 (limit: 1131)
     Memory: 3.0M
     CGroup: /system.slice/nginx.service
             ├─2313 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             └─2314 nginx: worker process
```

Navigating to `http://<server_ip>` at this point should return the default Nginx landing page.

### Set up a server block for your site:

Create a directory for your domain using the `-p` flag to create any necessary parent directories:

```shell
sudo mkdir -p /var/www/<domain>/
```

Make sure the ownership is correct:

```shell
sudo chown -R $USER:$USER /var/www/<domain>/
```

Create a sample `index.html` in the folder, you might want to use `nano` rather than vim here, if you've not used vim before and you want to [this cheat sheet](https://vim.rtorr.com/) is rather useful:

```shell
vim /var/www/<domain>/index.html
```

You don't have to get fancy with the contents, even this is more complicated than you need:

```html
<html>
    <head>
        <title>It works!</title>
    </head>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>
```

In order for Nginx to serve this it needs to know where it is, so we need to create a server block for it. This is done in `/etc/nginx/sites-available/` rather than `/ect/nginx/sites-enabled/` so that the site can be disabled later without losing the configuration:

```shell
sudo vim /etc/nginx/sites-available/<domain>
```

Add the following, don't forget the [tld](https://kinsta.com/knowledgebase/what-is-a-tld/) (.com, .co.uk) at the end of your url:

```
server {
    listen 80;
    listen [::]:80;

    root /var/www/<domain>;
    index index.html index.htm index.nginx-debian.html;

    server_name <domain> www.<domain>;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

NOTE: if you are using React Router you will need to include `/index.html` in the location block otherwise refresh won't work.

```
location / {
    try_files $uri $uri/ /index.html =404;
}
```

Now you need to enable it, by creating a sim link to `sites-enabled`:

```shell
sudo ln -s /etc/nginx/sites-available/<domain> /etc/nginx/sites-enabled/
```

To avoid possible hash bucket memory problems you need to adjust a single line in the `nginx.conf` file:

```shell
sudo vim /etc/nginx/nginx.conf
```

Find `server_names_hash_bucket_size` and uncomment that line (remove the #). Once you've done that test your Nginx files to check for errors:

```shell
sudo nginx -t
```

(If you accidentally run this without the sudo it **will** fail as it doesn't have the correct permissions to access the files it needs.)

If there aren't any errors restart nginx:

```shell
sudo systemctl restart nginx
```

### Set up SSL

Install Certbot and its Nginx plugin:

```shell
sudo apt install certbot python3-certbot-nginx
```

Obtain an SSL Certificate:

```shell
sudo certbot --nginx -d <domain> -d www.<domain>
```

If you haven't used `certbot` before you will be prompted to enter an email address and agree to the terms of service. Once you've done that `certbot` will attempt to set up your certificate, if it's successful you will be asked whether you want to redirect all traffic to HTTPS (you do).

Check that certbot's auto-renewal timer is working:

```shell
sudo systemctl status certbot.timer
```

Output:

```shell
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Wed 2022-04-20 08:27:42 UTC; 3h 59min ago
    Trigger: Wed 2022-04-20 14:11:07 UTC; 1h 44min left
   Triggers: ● certbot.service

Apr 20 08:27:42 mny-invoices systemd[1]: Started Run certbot twice daily.
```

Do a dry run to make sure:

```shell
sudo certbot renew --dry-run
```

## React

Build your project then upload your build files to `/var/www/<domain>/`. Done.

## Flask

Set up your python environment:

```shell
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
```

### Create a Virtual Environment:

Install venv:

```shell
sudo apt install python3-venv
```

Create a directory for your project:

```shell
mkdir ~/<project>
cd ~/<project>
```

Upload your Flask project (including the requirements.txt) to the folder.

Create and activate your venv so you can install your required modules:

```shell
python3 -m venv <myprojectenv>
source <mnyprojectenv>/bin/activate
```

If `wheel` and `uwsgi` aren't in your requirments file you'll need to install them first (`pip install wheel uwsgi`).

### WSGI Entry Point:

I was already using `api.py` to load my Flask application so I just needed to remove `app = create_app()` from the if and I was good to go:

```python
from my_app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host='0.0.0.0', debug=True)
```

### uWSGI Configuration File:

```shell
vim ~/<project>/<project>.ini
```

This file is going to need a few things in it:

```
[uwsgi]
module = api:app

master = true
processes = 5

uid = www-data
gid = www-data

socket = api.sock
chmod-socket = 666
vacuum = true

die-on-term = true
```

- `[uwsgi]`: tells uWSGI to apply the settings
- `module`: this specifies the module itself as well as the callable within it
- the next block tells uWSGI to start up in master mode and spawn 5 worker processes to handle requests
- set the user id (uid) and group id (gid) to `www-data` so that the socket can be accessed
- set up the socket, change file permissions to ensure `www-data` has read-write access and set `vacuum` so that the socket is cleaned up when the process stops.
- `die-on-term` helps to ensure the init system and uWSGI agree on what each process signal means.


### Set up a service to automatically start Flask and uWSGI when the server starts

```shell
sudo vim /etc/systemd/system/<project>.service
```

Contents:

```
[Unit]
Description=uWSGI instance to serve <project>
After=network.target

[Service]
User=<username>
Group=www-data
WorkingDirectory=/home/<user>/<project>
Environment="PATH=/home/<user>/<project>/<myprojectenv>/bin"
ExecStart=/home/<user>/<project>/<myprojectenv>/bin/uwsgi --ini <project>.ini

[Install]
WantedBy=multi-user.target
```

- `[Unit]` metadata and dependencies - used here to describe the service and tell the init system to only start after the networking target has been reached
- `[Service]` This section specifies the user, group and file locations to run the service
- `[Install]` Tells systemd what to link to this service when you enable it to start at boot, in this instance it should start when the multi-user system is up and running.


Start the service and enable it so that it starts at boot:

```shell
sudo systemctl start <project>
sudo systemctl enable <project>
```

Check its status:

```shell
sudo systemctl status <project>
```

Output will be something like:

```shell
● api.service - uWSGI instance to serve api
     Loaded: loaded (/etc/systemd/system/api.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-04-20 10:05:50 UTC; 3h 15min ago
   Main PID: 1989 (uwsgi)
      Tasks: 6 (limit: 1131)
     Memory: 54.5M
     CGroup: /system.slice/api.service
             ├─1989 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
             ├─2001 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
             ├─2002 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
             ├─2003 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
             ├─2004 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
             └─2005 /home/notquitehere/api/apienv/bin/uwsgi --ini api.ini
```

### Check that the socket is accessible

Make sure the permissions are set correctly on the containing folders by running the following command, you need to use the whole file path otherwise it will only return the permissions for the exact file:

```shell
namei -nom /home/<user>/<project>/<project>.sock
```

Output:

```shell
f: /home/<user>/<project>/<project>.sock
 drwxr-xr-x root         root         /
 drwxr-xr-x root         root         home
 drwxr-xr-x <user>       <user>       <user>
 drwxr-xr-x <user>       <user>       <project>
 srw-rw-rw- <user>       www-data     <project>.sock
```

If any of these don't have `r-x` as the last group you're not going to be able to access the socket file. Chmod them to the correct settings (755) before moving on.

### Setup the proxy

To set up the proxy so that we can access our api we need to go back and edit `/etc/nginx/sites-available/<project>`. There is a lot more information in this file than we had originally due to the SSL stuff being added however the section we're interested in is the domain bit at the top. Specificially we want to add an `/api` block with details of our uWSGI socket in it.

```
server {
    root /var/www/<domain>;
    index index.html index.htm index.nginx-debian.html;

    server_name <domain> www.<domain>;

    location / {
            try_files $uri $uri/ =404;
    }

    location /api {
            include uwsgi_params;
            uwsgi_pass unix:/home/<user>/<project>/<project>.sock;
    }

    [...]
}
```

Once that's been added check for errrors:

```shell
sudo nginx -t
```

If there aren't any, restart nginx:

```shell
sudo systemctl restart nginx
```

Everything **should** now work.

## Resources

- Installing Nginx on Ubuntu: <https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04>
- Securing Nginx with Let's Encrypt: <https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04>
- How to Deploy a React Application: <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-react-application-with-nginx-on-ubuntu-20-04>
- How to Deploy a React + Flask Project: <https://blog.miguelgrinberg.com/post/how-to-deploy-a-react--flask-project>
- How to serve a Flask Application: <https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04>
- How to Fix Common Nginx Errors <https://www.linuxbabe.com/linux-server/how-to-fix-common-nginx-errors>
