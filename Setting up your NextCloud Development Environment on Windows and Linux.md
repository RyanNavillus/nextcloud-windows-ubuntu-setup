# Quick Start Guide for Nextcloud Developers



## Introduction

This guide will walk developers through the process of setting up the proper development environment for contributing to Nextcloud, and installing the project with the right configuration settings. This is not an exhaustive list of all options for working on Nextcloud, simply a fast way to get started.




## Getting Started

Nextcloud runs PHP and Javascript code which can be hosted on an Apache Server. This server must be running on Linux. However, a virtual machine can be used on Window to achieve the same result. In this guide we will be using and Apache2 web server although [other alternatives](https://docs.nextcloud.com/server/12/admin_manual/installation/system_requirements.html) are supported.



##Setup for Windows

This Windows configuration will use VirtualBox to host Ubuntu 16.04



### Install the Virtual Machine with Ubuntu 16.04

Since Nextcloud's server does not run on Windows (this includes native Apache servers like XAMPP), they recommend that you run the system through a virtual machine:


1. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads (click ***Windows hosts***)
2. Download Ubuntu Desktop https://www.ubuntu.com/download/desktop (Server if your laptop is slower) https://www.ubuntu.com/download/server
3. Set up the virtual machine
  * Standard settings
  * Choose the Ubuntu file you downloaded as image
  * At the ***Erase disk and install Ubuntu*** it’s safe to do it because the virtual machine does not have access to your system
    ​


### Prepare your Ubuntu Virtual Machine Environment

Since we will be running Nextcloud on an Apache server, we will need to install that first.

* `sudo apt-get install apache2`

This should create a folder called `/vars/www/html/`
This is where apache will source its code from. In fact, if you type
`ls /vars/www/html/` you should see `index.html`, which will load when you type `localhost` into your browser:



![Apache index.html](http://yasoob.github.io/gci/images/default_apache.png)



Note that once you install Apache, the server will always run at startup. You can check the status of the Apache server with `systemctl status apache2` Check the [systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html) documentation for more information. 

You will also need the required PHP dependencies to run the project. First run:

* `sudo apt-get update`

Then install the dependencies:

* `sudo apt-get install php7.0 libapache2-mod-php7.0 php7.0-mbstring php7.0-curl php7.0-zip php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-bcmath php7.0-xml php7.0-json php7.0-tidy`

After this you should restart the apache webserver:

* `sudo /etc/init.d/apache2 restart`



### Installing the Nextcloud repository on Windows

We will be switching back to Windows for this step. To set up the Github project you should either use the [Windows Subsytem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) (WSL) or the [Git Bash](https://gitforwindows.org/) for Windows. To continue, depending on the system you chose, open either **Bash on Ubuntu on Windows** or **Git Bash**.

 As with any Github project, you need to get a copy of the source code. 

- If you are working in WSL and don't already have it, install git with `sudo apt-get install git`  
- In either WSL or Git Bash, configure your [username](https://help.github.com/articles/setting-your-username-in-git/) and [email](https://help.github.com/articles/setting-your-commit-email-address-in-git/). (You will need these to connect to the repo using ssh and to sign your code later)

Pick a location to install the repository, and navigate to it. For cloning the project you have two options:

1. Using https (recommended for novices): `git clone https://github.com/nextcloud/server.git`
2. Using ssh (recommended in general): `git clone git@github.com:nextcloud/server.git`

You should now have a folder called `server` at the path. Change directories into it with: 

- `cd server`

Then initialize all submodules in the repository:

- `git submodule update --init --recursive`

Every program you host on this Apache server will be another folder of the form `\vars\www\html\project\`. This is how we will be adding installing Nextcloud.



### Allow the Virtual Machine to Access your Local Nextcloud Repository

To access the repository on your Windows file system from the Ubuntu virtual machine, we need to share the folder with the virtual machine.

* First we need to [share the github repository in Windows with your Ubuntu file system](https://www.howtogeek.com/187703/how-to-access-folders-on-your-host-machine-from-an-ubuntu-virtual-machine-in-virtualbox/). 
* Make sure to follow all the steps when you do this. Also make sure to check ***Auto-Mount*** and ***Make Permanent*** when sharing the folder in VirtualBox Manager. **DO NOT** check ***Read Only***.

Now start up your Virtual Machine to make the following changes so that Apache can run the server.

1. We need to create a symbolic link to the mounted git repo folder so that Apache can access it. Virtualbox's Auto-Mount feature places the folder in ``/media/sf_foldername`. Then enter the command `ln -s /media/sf_server /var/www/html`
2. Nextcloud requires the system's httpd user (in this case ***www-data***) to own the repository. You can set this with: `sudo chown www-data:www-data -R /var/www/html/sf_server/`
   (If you run into issues here, you may wish to consult the more detailed [Nextcloud Documentation](https://docs.nextcloud.com/server/9/developer_manual/general/devenv.html#))
3. Only users in the ***vboxsf*** group are able to access the folders mounted by VirtualBox. To give Apache access to it, use the command `sudo usermod -a -G vboxsf www-data`



### Allow Windows to Connect to the Virtual Machine

In order to be able to connect to the server running in your virtual machine from Windows, you need to change some virtual machine network settings. Check [this link](https://dalanzg.github.io/tips-tutorials/ubuntu/2016/04/15/install-ubuntu-server-on-virtualbox/) for a guide on this process. Note that your web interfaces may be named differently than in this example.

You can use `ip addr show` in your virtual machine to find the IP address of your Nextcloud server (listed next to "inet")



### Setting up a Database

Nextcloud requires a database to function. Nextcloud [supports many database options](https://docs.nextcloud.com/server/12/admin_manual/installation/system_requirements.html) which you may wish to use. However, if you do not have experience with databases, a fast alternative to these is Heroku's default PostgreSQL configuration. This has the notable downside of not being usable offline, but it is convenient and easy to set up.

First you will need to install Postgres locally in your virtual machine:

* `sudo apt-get install postgresql postgresql-contrib php-pgsql`

To create a Postgres database hosted on Heroku:

1. Type `heroku login` to log into heroku

2. Type `heroku create project-name` to create a Heroku project

3. Go to https://www.heroku.com/postgres and click ***Get Started for Free***

4. If you aren't logged in, hit ***Login to Install***. Then hit ***Install Heroku Postgres*** and select your project

5. This should take you to the Resources page of your project. Click on `Heroku Postgres :: Database` and save that tab, you will need to come back to it in a moment.

   ​

### Creating Your Admin Account

Now it is time to open Nextcloud. Open a browser and enter the url `localhost/server`. The first time that you open Nextcloud you will see a page like this:



![NextCloud](https://community.time4vps.eu/uploads/editor/46/d95rm5utnwip.png)



You will have to manually add the database to Nextcloud. Go back to Heroku, go to the ***Settings*** tab, and click ***View Credentials*** next to ***Database Credentials***.

Copy the User, Password, Database, and Host fields into the Database User, Database Password, Database Name, and Database Host fields on the Nextcloud setup screen. You can choose your own admin account credentials.



### Conclusion 

You should now be able to open the server in Windows by entering the URL `localhost/server/` into the browser. If all goes well you should see the files gallery in Nextcloud! If not, look at the **Troubleshooting** section at the bottom of this page. Before you begin contributing, make sure to read the [contributing guidelines](https://github.com/nextcloud/server/blob/master/CONTRIBUTING.md) and to read about the [development process](https://docs.nextcloud.com/server/10/developer_manual/bugtracker/kanban.html). 



______



## Setup for Linux

This Linux setup was tested on Ubuntu and Debian. Make sure that you are running a supported Linux distribution.



### Prepare your Ubuntu Virtual Machine Environment

Since we will be running Nextcloud on an Apache server, we will need to install that first.

- `sudo apt-get install apache2`

This should create a folder called `/vars/www/html/`
This is where apache will source its code from. In fact, if you type
`ls /vars/www/html/` you should see `index.html`, which will load when you type `localhost` into your browser:



![Apache index.html](http://yasoob.github.io/gci/images/default_apache.png)



Note that once you install Apache, the server will always run at startup. You can check the status of the Apache server with `systemctl status apache2` Check the [systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html) documentation for more information. 

You will also need the required PHP dependencies to run the project. First run:

- `sudo apt-get update`

Then install the dependencies:

- `sudo apt-get install php7.0 libapache2-mod-php7.0 php7.0-mbstring php7.0-curl php7.0-zip php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-bcmath php7.0-xml php7.0-json php7.0-tidy`

After this you should restart the apache webserver:

- `sudo /etc/init.d/apache2 restart`



Every program you host on this Apache server will be another folder of the form `\vars\www\html\project\`. Let's add one now.

As with any Github project, you need to get a copy of the source code. 

* If you don't already have it, install git with `sudo apt-get install git` and configure your [username](https://help.github.com/articles/setting-your-username-in-git/) and [email](https://help.github.com/articles/setting-your-commit-email-address-in-git/). (You will need these to connect to the repo using ssh and to sign your code later)

For cloning the project you have two options:

* Using https (recommended for novices): `git clone https://github.com/nextcloud/server.git`
* Using ssh (recommended in general): `git clone git@github.com:nextcloud/server.git`

You should now have a folder called `server` at the path `/vars/www/html/server/`. Change directories into it with: 

 `cd server`


Then initialize all submodules in the repository:

`git submodule update --init --recursive`



Nextcloud requires the system's httpd user (in this case ***www-data***) to own the repository. You can set this with:

` sudo chown www-data:www-data -R /var/www/html/server/`

If you run into issues here, you may wish to consult the official [Nextcloud Documentation](https://docs.nextcloud.com/server/9/developer_manual/general/devenv.html#).



### Setting up a Database

Nextcloud requires a database to function. Nextcloud [supports many database options](https://docs.nextcloud.com/server/12/admin_manual/installation/system_requirements.html) which you may wish to use. However, if you do not have experience with databases, a fast alternative to these is Heroku's default PostgreSQL configuration. This has the notable downside of not being usable offline, but it is convenient and easy to set up.

First you will need to install Postgres locally:

- `sudo apt-get install postgresql postgresql-contrib php-pgsql`

To create a Postgres database hosted on Heroku:

1. Type `heroku login` to log into heroku

2. Type `heroku create project-name` to create a Heroku project

3. Go to https://www.heroku.com/postgres and click ***Get Started for Free***

4. If you aren't logged in, hit ***Login to Install***. Then hit ***Install Heroku Postgres*** and select your project

5. This should take you to the Resources page of your project. Click on `Heroku Postgres :: Database` and save that tab, you will need to come back to it in a moment.

   ​

### Creating Your Admin Account

Now it is time to open Nextcloud. Open a browser and enter the url `localhost/server`. The first time that you open Nextcloud you will see a page like this:



![NextCloud](https://community.time4vps.eu/uploads/editor/46/d95rm5utnwip.png)



You will have to manually add the database to Nextcloud. Go back to Heroku, go to the ***Settings*** tab, and click ***View Credentials*** next to ***Database Credentials***.

Copy the User, Password, Database, and Host fields into the Database User, Database Password, Database Name, and Database Host fields on the Nextcloud setup screen. You can choose your own admin account credentials.



### Conclusion

If all goes well you should see the files gallery in Nextcloud! Before you begin contributing, make sure to read the [contributing guidelines](https://github.com/nextcloud/server/blob/master/CONTRIBUTING.md) and to read about the [development process](https://docs.nextcloud.com/server/10/developer_manual/bugtracker/kanban.html).




### Troubleshooting

These are a few common issues you may run into while attempting to set up the Nextcloud development environment. This will only address Nextcloud specific issues, or relevant issues without readily available solutions.



#### Nextcloud Error - "Data directory is readable for other users. Please change the permissions to 0770 so that the directory cannot be listed by other users."

Your data folder may be in a few locations within the Apache hierarchy, but it is most likely at `\var\www\nextcloud-data\`

If this is the case, try to give it the correct permissions with:

`sudo chmod 770 /var/www/nextcloud-data`

If you cannot locate your data folder, you may wish to do a fresh install of the repository, this time also running [this script](https://doc.owncloud.org/server/9.0/admin_manual/installation/installation_wizard.html) before creating your admin account:

```bash
#!/bin/bash
ocpath='/var/www/owncloud'
htuser='www-data'
htgroup='www-data'
rootuser='root'

printf "Creating possible missing Directories\n"
mkdir -p $ocpath/data
mkdir -p $ocpath/assets
mkdir -p $ocpath/updater

printf "chmod Files and Directories\n"
find ${ocpath}/ -type f -print0 | xargs -0 chmod 0640
find ${ocpath}/ -type d -print0 | xargs -0 chmod 0750

printf "chown Directories\n"
chown -R ${rootuser}:${htgroup} ${ocpath}/
chown -R ${htuser}:${htgroup} ${ocpath}/apps/
chown -R ${htuser}:${htgroup} ${ocpath}/assets/
chown -R ${htuser}:${htgroup} ${ocpath}/config/
chown -R ${htuser}:${htgroup} ${ocpath}/data/
chown -R ${htuser}:${htgroup} ${ocpath}/themes/
chown -R ${htuser}:${htgroup} ${ocpath}/updater/

chmod +x ${ocpath}/occ

printf "chmod/chown .htaccess\n"
if [ -f ${ocpath}/.htaccess ]
 then
  chmod 0644 ${ocpath}/.htaccess
  chown ${rootuser}:${htgroup} ${ocpath}/.htaccess
fi
if [ -f ${ocpath}/data/.htaccess ]
 then
  chmod 0644 ${ocpath}/data/.htaccess
  chown ${rootuser}:${htgroup} ${ocpath}/data/.htaccess
fi
```



#### I received an error after attempting to create an admin account

This may occur if your initial installation attempt fails and creates an invalid config.php file. You can delete this file and try again with the command `rm /var/www/html/server/config/config.php`.  Note that you may need to be logged into the root user (`sudo -i`) to perform this action. You may wish to try a manual installation using [occ](https://docs.nextcloud.com/server/9/admin_manual/installation/command_line_installation.html) (filling out the necessary strings):

```bash
sudo -u www-data php occ maintenance:install --database "pgsql" --database-name "" --database-user "" --database-pass "" --database-host "" --admin-user "admin" --admin-pass "password2"
```



