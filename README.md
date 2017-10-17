### Setup Nginx, MySQL5.7 and multiple PHP versions using Homebrew on macOS High Sierra

In this guide, we covered how to installing multiple PHP versions (specifically, 5.6 and 7.0), Mysql & configure Nginx on macOS High Sierra with Homebrew, step by step

## Table of Contents
  1. Install Xcode
  2. Install Homebraw (Homebrew is package manager for OS X.)
  3. Install Nginx
  4. Install multiple PHP versions (PHP5.6 & PHP7.1)
  5. Install MySQL5.7

#### 1. 	Install Xcode
Download it from developer.apple.com or iTunes (App Store), install and agree the license.

Open Terminal and install the Xcode Command Line tools.
```
$ xcode-select --install
```

#### 2.	Install Homebraw
To install homebrew, type this in terminal:
```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Remove outdated installed package versions:
```
$ brew cleanup
```

Running brew doctor will inform you if you have any problems with your install:

```
$ brew doctor
```

Updates all packages (formulas) with"

```
$ brew update
$ brew upgrade
```

#### 3.  Install Nginx
Install default nginx with:
```
brew install nginx
```

You can start nginx manually and check if running in browser: http://localhost:8080 or open http://localhost:80

Start, Stop and Restart php-fpm

```
brew services reload nginx
brew services restart nginx
brew services start nginx
brew services stop nginx
```

Or makes it autostart:

```
sudo cp /usr/local/opt/nginx/*.plist /Library/LaunchDaemons
sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

Nginx configuration can be found here

```
/usr/local/etc/nginx/nginx.conf
```

Serving multiple local website, first step is to copy any configuration file before you begin making changes to it, like so:

```
cp /usr/local/etc/nginx/nginx.conf /usr/local/etc/nginx/nginx.conf.bak
```

Setup virtuals, prepare the following file directory:

```
mkdir /usr/local/etc/nginx/sites-available
mkdir /usr/local/etc/nginx/sites-enabled
```

Modify nginx.conf

```
$ vi /usr/local/etc/nginx/nginx.conf
```

At the bottom of your nginx.conf file you should include the following line:

```
http {
  # ... ...
  # ... ... default nginx stuff
  # ... ...

  # Add this to the bottom of your http block
  include sites-enabled/*.conf;
}
```

adding sites put a specific server config, just place them into your sites-available directory:

```
vim /usr/local/etc/nginx/sites-available/php.conf
```

Example PHP Ngnix Config:

```
		server {
		        ## Your website name goes here.
		        #server_name     php56.local.com;
				server_name     php71.local.com;
		        ## Listen Port
		        listen          *:80;
		        ## Your only path reference.
		        #root /usr/local/var/www/php56/public;
		        root /usr/local/var/www/php71/public;
		        ## This should be in your http block and if it is, it's not needed here.
		        index index.php;

		        location = /favicon.ico {
		                log_not_found off;
		                access_log off;
		        }

		        location = /robots.txt {
		                allow all;
		                log_not_found off;
		                access_log off;
		        }

		        location / {
		                # This is cool because no php is touched for static content.
		                # include the "?$args" part so non-default permalinks doesn't break when using query string
		                try_files $uri $uri/ /index.php?$args;
		        }

		        location ~ \.php$ {
		                # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
		                include fastcgi.conf;
		                fastcgi_intercept_errors on;
		                # fastcgi_pass   127.0.0.1:9056;
		                fastcgi_pass   127.0.0.1:9071;
		        }

		        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
		                expires max;
		                log_not_found off;
		        }
		}
```

Uncomment to specify to use which php version
```
# fastcgi_pass   127.0.0.1:9056;
# fastcgi_pass   127.0.0.1:9071;
```

Additionally there are examples and tutorials can be found on the following:

```
https://www.nginx.com/resources/wiki/start/#pre-canned-configurations
```

Next to enable the site, create symlink to sites-enabled:

```
sudo ln -s /usr/local/etc/nginx/sites-available/php.conf /usr/local/etc/nginx/sites-enabled/php.conf
```

To disable the site you can simply delete the conf from sites-enabled folder.

```
rm /usr/local/etc/nginx/sites-available/php.conf
```

Finally you must register your servername to hosts file located in /private/etc/hosts

```
127.0.0.1 php56.local.com
127.0.0.1 php71.local.com
```

### 4. Install multiple PHP versions
In order to install PHP you have to install the homebrew/php tap first (this is needed only once):

```
$ brew tap homebrew/php
```

See which versions of PHP we can have first:

```
$ brew search php
```

Installing PHP 5.6 with php-fpm & some extensions:

```
$ brew install php56 --with-fpm
$ brew install php56-mcrypt
$ brew install php56-xdebug
$ brew unlink php56
```

The unlink command allows you to install multiple PHP versions. The extensions mcrypt and xdebug are optional, so only the first and last lines are required.

After running these commands, first open /usr/local/etc/php/5.6/php-fpm.conf and find this line:

```
listen = 127.0.0.1:9000
```

Change to

```
listen = 127.0.0.1:9056
```

Setup auto start:

```
$ ln -sfv /usr/local/opt/php56/*.plist ~/Library/LaunchAgents
```

And to load php-fpm 5.6 now:

```
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.php56.plist
```

Installing PHP 7.1 with php-fpm & some extensions:

```
$ brew install php71
$ brew install php71-mcrypt
$ brew install php71-xdebug
$ brew unlink php71
```

First, open /usr/local/etc/php/7.1/php-fpm.d/www.conf and find this line:

```
listen = 127.0.0.1:9000
```

Change to

```
listen = 127.0.0.1:9071
```

Setup auto start:

```
$ ln -sfv /usr/local/opt/php71/*.plist ~/Library/LaunchAgents
```

And to load php-fpm 7.1 now:

```
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.php71.plist
```

The php.ini file is located:

```
/usr/local/etc/php/5.6/php.ini
/usr/local/etc/php/7.1/php.ini
```

Test PHP Processing on your Web Server

Create folder:
```
mkdir /usr/local/var/www/php71/
mkdir /usr/local/var/www/php71/public/

mkdir /usr/local/var/www/php56/
mkdir /usr/local/var/www/php56/public/
```

We can create the file at that location by typing:

```
sudo vi /usr/local/var/www/php71/public/index.php
sudo vi /usr/local/var/www/php56/public/index.php
```

This will open a blank file. 
 
```
<?php phpinfo(); ?>
```

### 5. Install MySQL (Latest Version)
Run the following in your command-line:

```
$ brew install mysql
```

Start mysql now
```
brew services start mysql
```

By default MySQL server installs with no root password but locks it to localhost (so only local users can access it anyway)

To secure our MySQL server, we need to change the root password, remove anonymous users and disable remote root logins:

```
mysql_secure_installation
```

> Enter current password for root (enter for none):

Press ENTER since you don’t have one set.

> Change the root password? [Y/n]
Confirm using ENTER to accept the suggested default answer (Y) and enter it here in the prompt.

> Remove anonymous users? [Y/n]

Press again ENTER. They are not necessary.

> Disallow root login remotely? [Y/n]

ENTER — No need to log in as root from any other IP than 127.0.0.1.

> Remove test database and access to it? [Y/n]

ENTER — You don’t need the testing tables.

> Reload privilege tables now? [Y/n]

ENTER — Reload the privilege tables to ensure all of the changes made so far will take effect immediately.

Setup auto start

```
$ ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
```

And start the database server:

```
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

Test connection
To ensure that all works fine we can connect to mysql with following

```
mysql -u root -p
```

Enter your root password and you should see the MySQL console:

Type ‘help;’ or ‘\h’ for help. Type ‘\c’ to clear the current input statement.

mysql>


Download SequelPro and install it.

```
http://www.sequelpro.com/
```
