# linode-config

Setting up a linode server

## Getting Started

Follow the guide [here](https://www.linode.com/docs/getting-started).
 However, note the following

- I chose Ubuntu 14.04
- I set up TWO servers, a WEB and APP server.
  - Web server: ~4GB
  - App server: ~20GB
- In order to do this, you have to deploy image twice.

In the following sections, everything is to be done on the app server
 unless otherwise specified.

## Adding a User

SSH into the App server.
Type this into the server you're now ssh'd into

```
adduser <user>
adduser <user> sudo
```

## Installing basic stuff

Install the basics:

```
sudo apt-get install git
```

### Fish

I installed fish shell, because I like it better.
Do this to install it:

```
sudo apt-add-repository ppa:fish-shell/release-2
sudo apt-get update
sudo apt-get install fish
```

Do this to change it to your default shell:
Note: this may break things...

```
chsh -s /usr/bin/fish
```

Make config directory and file:

```
mkdir -p ~/.config/fish
touch ~/.config/fish/config.fish
```

Open that file with nvim and add the following lines:

```
# add /usr/local/bin to path
set -g -x PATH /usr/local/bin $PATH
```

To theme the shell I did the following:

```
git clone https://github.com/chriskempson/base16-shell.git ~/.config/theme/base16-shell
```

Add these lines to your ~/.config/fish/config.fish
 (replacing with favorite color scheme):

```
eval sh $HOME/.config/theme/base16-shell/base16-flat.light.sh
```

Add these lines to color the prompt (taken from [here](https://gist.github.com/JoshAntBrown/bad905396f28446bb659))

```
set -g fish_color_normal      base0
set -g fish_color_command     base0
set -g fish_color_quote       cyan
set -g fish_color_redirection base0
set -g fish_color_end         base0
set -g fish_color_error       red
set -g fish_color_param       blue
set -g fish_color_comment     base01
set -g fish_color_match       cyan
set -g fish_color_search_match "--background=$base02"
set -g fish_color_operator    orange
set -g fish_color_escape      cyan
set -g fish_color_hostname    cyan
set -g fish_color_cwd         yellow
set -g fish_color_git         green
```

### Tmux

do this:

```
cd ~/.config/theme
wget https://raw.githubusercontent.com/gchiam/gchiam-dotfiles/master/.tmux-theme-base16.conf
mv .tmux-theme-base16.conf tmux-theme-base16.conf
```

Add this to tmux.conf:

```
source ~/.config/theme/tmux-theme-base16.conf
```

Also add these lines:

```fish
# remove greeting
set fish_greeting ""

# set fish as default shell
set -g default-shell "/usr/bin/fish"

# set vim keys for copy-mode
setw -g mode-keys vi

# for vim immediate return to normal mode
set -s escape-time 0
```

### Neovim

```sh
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt-get update
sudo apt-get install neovim
```

Create a config file:

```sh
touch ~/.config/nvim/init.vim
```

## Postgresql

Using [this](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)

Do these commands

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ \
 trusty-pgdg main" >> /etc/apt/sources.list.d/postgresql.list'
sudo apt-get update
sudo apt-get install postgresql-9.5 postgresql-contrib-9.5
```

Log into newly created postgres account:

```sh
sudo -i -u postgres
```

Get the postgres prompt:

```sh
psql
```

and quit

```sql
\q
```

Creat user (use same name as your username):

```sh
creatuser <username> -d -s -P
```

Create the database for the new user:

```sh
createdb <username>
```

Connect to the db using new user

```
sudo -i -u <usrname>
psql
```

Exit psql with `\q`.

I created another db for a project I'm hosting. I made my user the owner,
 and user to connect as.

```
createdb green-pattern-map -p 5433
```

Now make sure that postgres starts on reboot:

```
sudo update-rc.d postgresql enable
```

### PostGIS

Install postgis

```
sudo apt-get install postgresql-9.5-postgis-2.2
sudo apt-get install postgis
```

login to psql and enter the following:

```sql
CREATE EXTENSION postgis;
```

login to each db and do this

### Enabling remote access to postgres server

I do this to enable gui tools to interact with it

Login to postgres user: `sudo -i -u postgres`

First find your ip address on your own computer:

```
ip route get 8.8.8.8 | awk '{print $NF; exit}'
```

You have to make sure that your IP Address is static for this.

edit this file:

```
nvim /etc/postgresql/9.5/main/pg_hba.conf
```

Add this line, and replace with your ip address (followed by something like /24):

```
host all all <ip-address> trust
```

Edit this file:

```
nvim /etc/postgresql/9.5/main/postgresql.conf
```

Edit the line with `listen_addresses='localhost'`:

```
listen_addresses='*'
```

Make sure to uncomment it!

Now restart postgres:

```sh
/etc/init.d/postgresql restart
```

Make sure iptables doesn't block postgres (do as root):

```
iptables -A INPUT -p tcp -s 0/0 --sport 1024:65535 -d 10.10.29.50 \
--dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -s 10.10.29.50 \
--sport 5432 -d 0/0 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
```

Restart the firewall:

```
sudo service ufw stop
sudo service ufw start
```

Log in from YOUR computer with the following
 (make sure that you also have psql on your computer)

```sh
psql -h <server ip> -U <user> -d <dbname> -p 5432
```

## Nodejs

We are installing Nodejs 5.5.0.
Do the following:

```
sudo apt-get install nodejs npm nodejs-legacy
sudo npm install n -g
sudo n 5.5.0
```

## Setting up domain

Login to the domain provider (I used namecheap)
 for your domain and add Name servers from linode:

- ns1.linode.com
- ns2.linode.com
- ns3.linode.com
- ns4.linode.com
- ns5.linode.com

Now you have to configure your domain.

Go to the DNS manager on the linode dashboard and add a zone.
Add your domain name here.
Do that and click "Add a Master Zone"

Now you have to can add DNS record.

Or you can stick with the stuff they automatically fill in (www, mail)

In the DNS Manager tab add a new A record...

Fill in Hostname with 'testing' (or whatever you desire...)

Fill in the IP Address with the servers ip

## Setting up basic nodejs server

I followed [this guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-14-04)
 but I didn't set up a seperate reverse proxy server,
 so use the localhost IP address for app server's
 private IP.

Install PM2:

```
sudo npm install -g pm2
```

Make a directory in your home directory for code. I call it Apps.
Also make a directory for our test app.

```
mkdir -p ~/Apps
mkdir -p ~/Apps/hello
```

Open ~/Apps/hello/hello.js with nvim and add these lines,
 replacing APP_PRIVATE_IP_ADDRESS with the private IP of the server.
 Or 127.0.0.1 if no seperate server. You don't need the /17

```
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(8080, '127.0.0.1');
console.log('Server running at http://127.0.0.1:8080/');
```

Now test the code (start a tmux session, so you can do both of these)

```
node ~/Apps/hello/hello.js
```

In a seperate window:

```
curl http://127.0.0.1:8080
```

If you get "hello world" it worked!

Now we have to get the app working as a background process with pm2.

Do this:

```
pm2 start ~/Apps/hello/hello.js
```

Now do this to kill it:

```
pm2 stop hello
```

To get the app to start on system startup.

```
pm2 start ~/Apps/hello/hello.js
bash
sudo env PATH=$PATH:/usr/local/bin pm2 startup ubuntu -u <user>
```

Then `sudo reboot` to make sure it worked.
SSH back in and then `pm2 ls` to make sure the
 app is running.

Note: if you screw up, do this: `sudo update-rc.d -f pm2-init.sh remove`

### Setting up reverse proxy

```
sudo apt-get install nginx
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup
sudo nvim /etc/nginx/sites-available/default
```

Add these lines, replacing server_name with your domain.
 Then `sudo reboot`

```
server {
    listen 80;

    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    }
```

This lets any access to example.com redirect to 127.0.0.1:8080
 i.e. your nodejs hello app.

Great! First app set up.

### Adding Other Apps

In order to add any extra apps, you would add another block to
 /etc/nginx/sites-available/default like the following. Make sure
 that any other apps have different ports. Anything
 8080+

```
    location /app2 {
        proxy_pass http://127.0.0.1:8081;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```

Put your app in the ~/Apps directory, and make sure to run:

```
pm2 start ~/Apps/<newapp>/<newapp.js>
pm2 save
```

If you make any changes to an app, make sure to `pm2 restart <appname>`

Personally, I changed hello.js to be an index of the various projects I'm hosting.


