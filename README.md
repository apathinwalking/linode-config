# linode-config
Setting up a linode server

## Getting Started

Follow the guide [here](https://www.linode.com/docs/getting-started)

I chose Ubuntu 14.04

## Adding a User

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
createdb <otherdb>
```

I created another db called watermap for that project specifically

Connect to the db using new user

```
sudo -i -u <usrname>
psql
```

### PostGIS

Install postgis

```
sudo apt-get install postgresql-9.5-postgis-2.2
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

You need NVM to do that.

Do this:

```
git clone https://github.com/creationix/nvm.git ~/.nvm
cd ~/.nvm
git checkout (git describe --abbrev=0 --tags)
```

Get a wrapper for the fish shell

```

cd ~/.nvm
wget https://raw.githubusercontent.com/passcod/nvm-fish-wrapper/master/nvm.fish
echo "set -x NVM_DIR ~/.nvm"
source $NVM_DIR/nvm.fish
```

That should "install" nvm.

Exit the server, and relog in.

Now do this:

```
nvm install 5.5.0
```

Delete these lines from ~/.profile, and add them to ~/.bashrc:

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```



