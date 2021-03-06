---
published: true
title: 'WordPress with WP-CLI on Bash on Ubuntu on Windows 10'
cover_image: ''
description: 'Using wp-cli on Bash on Windows 10'
tags: php, wordpress, wp-cli, windows
series:
canonical_url: 'https://worldwildweb.dev/wordpress-with-wp-cli-on-bash-on-ubuntu-on-windows-10/'
---

At first sight this sounds ridiculous. In fact it sounds absurd. But it’s not, if you heard of the new Microsoft feature — Bash on Ubuntu on Windows.

#### Windows 10’s Anniversary Update offered a big new feature for developers: A full, Ubuntu-based Bash shell that can run Linux software directly on Windows. This is made possible by the new “Windows Subsystem for Linux” Microsoft is adding to Windows 10.

In this post, I will setup fully functional WordPress installation with WP-CLI on top of LAMP server which will be installed on my Linux Subsystem through Windows 10.

Let’s do this!

## First we need to install Bash on Ubuntu on Windows

Tutorial how to do it, can be found [here](http://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/). After the installation is completed, run the app as administrator. If you would like to go with different user than root, you can read [this](http://www.howtogeek.com/261417/how-to-change-your-user-account-in-windows-10s-ubuntu-bash-shell/).

## LAMP stack is the next task

WordPress requires PHP, MySQL and HTTP server. We can go with Apache, so LAMP toolset is all we need. This command will install PHP, MySQL and Apache in a moment:
`$ sudo apt-get install lamp-server^`

## When we have our server ready we can start with the fun part — installing our command-line interface for WordPress

_Note that you must have PHP 5.3.29 or later and the WordPress version 3.7 or later._

Download the wp-cli.phar file via curl:
`$ curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar`

Now lets check, if it is running:
`$ php wp-cli.phar --info`

To use WP-CLI from the command line by typing wp, make the file executable and move it somewhere in your PATH. For example:
`$ chmod +x wp-cli.phar`
`$ mv wp-cli.phar /usr/local/bin/wp`

Check, if it is working:
`$ wp --info`

If everything went okay, you should see something like this:

```
$ wp --info
PHP binary: /usr/bin/php5 PHP version: 5.5.9-1ubuntu4.14
php.ini used: /etc/php5/cli/php.ini
WP-CLI root dir: /home/wp-cli/.wp-cli
WP-CLI packages dir: /home/wp-cli/.wp-cli/packages/
WP-CLI global config: /home/wp-cli/.wp-cli/config.yml
WP-CLI project config: WP-CLI version: 0.23.0$
```

## Run our local WordPress installation

Аfter we have our environment and tools ready, playing with the WP-CLI makes the new local WP installation a few commands away.

Navigate to:
`$ cd ../var/www/html`

This is where our WP will live. Remove the default `index.php` file.
Also, now is a good time to start our Apache and MySQL services:
`$ rm index.html`
`$ service apache2 start`
`$ service mysql start`

Let’s download our WP core files:
`$ wp core download`

This will download the latest version of WordPress in English (`en_US`). If you want to download another version or language, use the `--version` and `--locale` parameters. For example, to use the Bulgarian localization and 4.2.2 version, you would type:
`$ wp core download --version=4.2.2 --locale=bg_BG`

Once the download is completed, you can create the `wp-config.php` file using the `core config` command and passing your arguments for the database access here:
`$ wp core config --dbname=databasename --dbuser=databaseuser --dbpass=databasepassword --dbhost=localhost --dbprefix=prfx_`

This command will use the arguments and create a `wp-config.php` file.
Using the db command, we can now create our datebase:
`$ wp db create`

This command will use the arguments from `wp-config.php` and do the job for us.
Finally, to install WordPress, use the `core install` command:
`$ wp core install --url=example.com --title=WordPress Website Title --admin_user=admin_user --admin_password=admin_password --admin_email=admin@example.com`

If you’ve got your success message, restart Apache:
`$ service apache2 restart`

After the restart is completed, open your browser and type [http://localhost/](http://localhost/) and enjoy your brand new WP installation.

_Note that every time you start the application you must first start up MySQL and Apache services:_

`$ service mysql start|restart|stop`
`$ service apache2 start|restart|stop`

# Final words

I like WP-CLI a lot for saving me time in doing boring stuff, but this may seems as a workaround and it probably is. Still, nothing compares to a real Linux environment.  
But let’s face the fact we are running native Linux software on Windows. That by itself is fantastic. The benefits are unlimited.

Keep in mind that the Bash on Ubuntu on Windows feature is still in beta, so let’s hope that Microsoft will keep up with the good news.

Please, share and comment in the section below for your opinion.

# Further readings

[How to access your Ubuntu Bash files in windows and your Windows system drive in Bash](http://www.howtogeek.com/261383/how-to-access-your-ubuntu-bash-files-in-windows-and-your-windows-system-drive-in-bash/)

[How to create and run Bash shell scripts on Windows 10](http://www.howtogeek.com/261591/how-to-create-and-run-bash-shell-scripts-on-windows-10/)

[http://wp-cli.org/docs/tools/](http://wp-cli.org/docs/tools/)
