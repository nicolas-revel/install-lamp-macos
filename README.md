# Prerequisite

Before installing any dependancy regarding your local LAMP stack, you need to be sure that you ave completely remove MAMP from your computer. If you have any trace of an old localhost or php version version or whatever, it might get in conflict with the one that you may try to install.

## Uninstall MAMP

1.  Go to the Applications folder → MAMP PRO.
2. Launch the MAMP PRO Uninstaller.app item.
3. In the MAMP PRO Uninstaller window, click the Uninstall button. It will remove all folders and files belonging to the MAMP PRO installation, including the MYSQL databases that were modified while using MAMP PRO.
4. Enter your administrator password to confirm the action.
5. Remove the MAMP folder from the Applications folder.
6. After this, I recommend that you check whether the app’s service files are still remaining on your Mac. By default, they should be stored in the hidden Library folder. In Finder press Command+Shift+G keys and type _~/Library_ in the search field. Click on **Go**.
7. In the Library find and remove all files associated with MAMP. Check the following folders for them:
	-   _~/Library/Application Support_
	-   _~/Library/Caches_
	-   _~/Library/Container_
	-   _~/Library/Preferences_
	-   _~/Library/Application Scrip
8. Once you remove all remaining files of MAMP, empty your Trash bin to entirely remove MAMP from your Mac.

## Updating XCode and Brew

Before going forward, remember to update your XCode version and then update homebrew by doing the following command :

```bash
brew update && brew upgrade
```

Then we can start installing the different modules.

# Installation

## Apache (or httpd)

To install Apache you need to run this command :

``` bash
brew install httpd
```

Once the installation is complete, you can restart you can check apache status

``` bash
brew services info httpd
```

You can also check wether apache 2 is working or not by requesting `localhost:8080` on favorite browser and you should see "It Works !". Though, you might have seen that we are on the port `8080` and that it might be more pleasant to be on the default apache port `80` . To do so we need to change this line in the httpd conf file :

```bash
# File location : /opt/homebrew/etc/httpd/httpd.conf

Listen 8080
# Change to :
Lister 80
```

## MariaDB

To install mariadb you need to run this command :

``` bash
brew install mariadb
```

Once the installation is complete you might want to change the root password or even set it to blank. To do so you will need to enter mariadb by doing  `mariadb` and then run the following commands :

``` mysql
# We switch to the mysql database
USE mysql;

# Change the password of the root user
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password_here';

# Apply changes to the database
FLUSH PRIVILEGES;

# Exit mysql
exit();
```

Once you have changed the password, you can restart the apache server by doing :

```bash
brew services restart httpd
```

## PHP & phpMyAdmin

Then we are going to install php :

```bash
brew install php
```

Then you can install phpMyAdmin :

```bash
brew install phpmyadmin
```

Once it is install, do not forget to add the following at the end of your apache2 config file :

```bash
# /opt/homebrew/etc/httpd/httpd.conf

Alias /phpmyadmin $HOMEBREW_PREFIX/share/phpmyadmin  
<Directory $HOMEBREW_PREFIX/share/phpmyadmin/>  
     Options Indexes FollowSymLinks MultiViews  
     AllowOverride All  
     <IfModule mod_authz_core.c>  
	     Require all granted  
     </IfModule>  
     <IfModule !mod_authz_core.c>  
	     Order allow,deny  
	     Allow from all
     </IfModule>  
</Directory>
```

Then you can edit your phpmyadmin configuration file to allow connection with no password for users by uncommenting this line :

```php
# /opt/homebrew/etc/phpmyadmin.config.inc.php

$cfg['Servers'][$i]['AllowNoPassword'] = true;
```

Once this is done you can restart your apache server to update your server

```bash
brew services restart httpd
```

## XDebug

Before installing XDebug, you need to check if XCode command line tools is installed (`xcode-select --install`) and that php is installed via homebrew.

When you are good to go, you can install XDebug by running :

```bash
pecl install xdebug
```

On Apple M1 hardware, programs can either be compiled for the native M1/ARM64 architecture, or for the emulated x86_64 architecure. Sometimes there is a mismatch with the default and PECL will fail, or Xdebug won't load with a message such as:

> PHP Warning:  Failed loading Zend extension 'xdebug.so' (tried: opt/homebrew/lib/php/pecl/20190902/xdebug.so (dlopen(/opt/homebrew/lib/php/pecl/20190902/xdebug.so, 9): no suitable image found.  Did find:
>         /opt/homebrew/lib/php/pecl/20190902/xdebug.so: mach-o, but wrong architecture
>         /opt/homebrew/lib/php/pecl/20190902/xdebug.so: stat() failed with errno=22), /opt/homebrew/lib/php/pecl/20190902/xdebug.so.so (dlopen(/opt/homebrew/lib/php/pecl/20190902/xdebug.so.so, 9): image not found)) in Unknown on line 0

You can verify what your PHP's architecture is with:

```bash
file `which php`
```

If that says `arm64e`, then you need to run:

```bash
arch -arm64 sudo pecl install xdebug
```

And if it's `x86_64`, then you need to run:

```bash
arch -x86_64 sudo pecl install xdebug
```

In some cases `pecl` will change the `php.ini` file to add a configuration line to load Xdebug. You can check whether it did by running `php -v`. If Xdebug shows up with a version number, than you're all set and you can configure Xdebug's other functions, such as [Step Debugging](https://xdebug.org/docs/step_debug), or [Profiling](https://xdebug.org/docs/profiler). Otherwise, follow these steps :
1. If `pecl` did not add the right line, you might need to run `php --ini`in the command line. If there is a file with `xdebug` in the name, such as `/opt/homebrew/etc/php/7.4/cli/conf.d/99-xdebug.ini`, then this is the file to use. If that file does not exist, but there are other files in a `conf.d` or similar directory, you can create a new file there too. Please name it `99-xdebug.ini` in that case.
2. Add the following line to this PHP ini file :`zend_extension=xdebug`
3. Restart your webserver via `brew services restart httpd`
4. Verify that Xdebug is now loaded :
	- Create a PHP page that calls [xdebug_info()](https://xdebug.org/docs/all_functions#xdebug_info). If you request the page through the browser, it should show you an overview of Xdebug's settings and log messages.
	- On the command line, you can also run `php -v`. Xdebug and its version number should be present as in:
> 	  PHP 7.4.10 (cli) (built: Aug 18 2020 09:37:14) ( NTS DEBUG )
> 	  Copyright (c) The PHP Group
> 	  Zend Engine v3.4.0, Copyright (c) Zend Technologies
> 			  with Zend OPcache v7.4.10-dev, Copyright (c), by Zend Technologies
> 			  with Xdebug v3.0.0-dev, Copyright (c) 2002-2020, by Derick Rethans 
