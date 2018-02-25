# Getting Started with Contributing to NextCloud

## Introduction

I recently began contributing to the NextCloud Server on Github. As I was setting up my development environment, I noticed that the process took significantly longer than I expected. I also ran into many pitfalls where NextCloud's own documentation provided little assistance. I decided to create this guide with some basic guidelines, and a huge collection of troubleshooting advice. This is not an exhaustive list of all options for working on NextCloud, simply a fast way for developers to get started. With luck, this will help new developers contribute to an otherwise novice-friendly code base with exceptional community support.


## Getting Started

NextCloud runs PHP and Javascript code hosted on an Apache Server. This server must be running on Linux. However, a virtual machine can be used on Window to achieve the same result.


### Extra Steps for Windows 

If you are not using Windows, you can skip ahead to the next section. Many of these steps are taken from [nextcloud-scripts](https://github.com/jancborchardt/nextcloud-scripts/blob/master/Nextcloud%20Windows%20development%20environment.md) by jancborchardt.

Since NextCloud does not run on Windows (this includes native Apache servers like XAMPP), they recommend that you use the system through a virtual machine:


1. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads (click ***Windows hosts***)
2. Download Ubuntu Desktop https://www.ubuntu.com/download/desktop (Server if your laptop is slower) https://www.ubuntu.com/download/server
3. Set up the virtual machine
  * Standard settings
  * Choose the Ubuntu file you downloaded as image
  * At the ***Erase disk and install Ubuntu*** it’s safe to do it because the virtual machine does not have access to your system


Now enter the Virtual Machine and follow the steps for Linux.


### In Linux

Since we will be running NextCloud on an Apache server, we will need to install that first.
`sudo apt-get install apache2`

This should create a folder called `/vars/www/html/`
This is where apache will source its code from. In fact, if you type
`ls /vars/www/html/` you should see `index.html`, which will load when you type `localhost` into your browser:



![Apache index.html](http://yasoob.github.io/gci/images/default_apache.png)



You will also need the required PHP dependencies to run the project. First run:

`sudo apt-get update`

Then install the dependencies:

`sudo apt-get install php7.0 libapache2-mod-php7.0 php7.0-mbstring php7.0-curl php7.0-zip php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-bcmath php7.0-xml php7.0-json php7.0-tidy`

After this you should restart the apache webserver:

`sudo /etc/init.d/apache2 restart`



Every program you host on this Apache server will be another folder of the form `\vars\www\html\project\`. Let's add one now.

As with any Github project, you need to get a copy of the source code. 

If you don't already have it, install git with `sudo apt-get install git` and configure your [username](https://help.github.com/articles/setting-your-username-in-git/) and [email](https://help.github.com/articles/setting-your-commit-email-address-in-git/). (You will need these to connect to the repo using ssh and to sign your code later)

For cloning the project you have two options:
Using https (recommended for novices): `https://github.com/nextcloud/server.git`
Using ssh (recommended in general): `git clone git@github.com:nextcloud/server.git`

You should now have a folder called `server` at the path `/vars/www/html/server/`. Change directories into it with: 

 `cd server`


Then initialize all submodules in the repository:

`git submodule update --init --recursive`



NextCloud requires the system's httpd user (in this case ***www-data***) to own the repository. You can set this with:

` sudo chown www-data:www-data -R /var/www/html/server/`

If you run into issues here, you may wish to consult the official [NextCloud Documentation](https://docs.nextcloud.com/server/9/developer_manual/general/devenv.html#).




### Setting Up Your Database On Heroku

NextCloud supports many database options including MySQL, MariaDB, and PostgreSQL. I find that Heroku's built in Postgres database system is easy to use and works seamlessly for NextCloud. Note that if you plan on working heavily on NextCloud's database interface, you may want to choose a [local database solution](https://docs.nextcloud.com/server/12/admin_manual/configuration_database/linux_database_configuration.html). You will need to [create a Heroku account](https://signup.heroku.com/?c=70130000001x9jFAAQ) if you do not have one.



First you will need to install Postgres locally:

``sudo apt-get install postgresql postgresql-contrib php-pgsql`



To create a Postgres database hosted on Heroku:

1. Type `heroku login` to log into heroku
2. Type `heroku create project-name` to create a Heroku project
3. Go to https://www.heroku.com/postgres and click ***Get Started for Free***
4. If you aren't logged in, hit ***Login to Install***. Then hit ***Install Heroku Postgres*** and select your project
5. This should take you to the Resources page of your project. Click on `Heroku Postgres :: Database` and save that tab, you will need to come back to it in a moment.



### Creating Your Admin Account

Now it is time to open NextCloud. Open a browser and enter the url `localhost/server`. The first time that you open NextCloud you will see a page like this:



![NextCloud](https://community.time4vps.eu/uploads/editor/46/d95rm5utnwip.png)



You will have to manually add the database to NextCloud. Go back to Heroku, go to the ***Settings*** tab, and click ***View Credentials*** next to ***Database Credentials***.

Copy the User, Password, Database, and Host fields into the Database User, Database Password, Database Name, and Database Host fields on the NextCloud setup screen. You can choose your own admin account credentials.

If all goes well you should see the files gallery in NextCloud! Before you begin contributing, make sure to read the [contributing guidelines](https://github.com/nextcloud/server/blob/master/CONTRIBUTING.md) and to read about the [development process](https://docs.nextcloud.com/server/10/developer_manual/bugtracker/kanban.html).




### Troubleshooting

I ran into many issues while attempting to set up my NextCloud development environment. I hope that by addressing them here, future developers will be able to start working much faster. I will only be addressing NextCloud specific issues here, or relevant issues without readily available solutions.



#### I found the [nextcloud-scripts](https://github.com/jancborchardt/nextcloud-scripts/blob/master/Nextcloud%20Windows%20development%20environment.md) and am running into issues trying to set it up...

While I greatly appreciate this repository when setting up my development environment, I would not recommend trying to mount the NextCloud repo from Windows to a Ubuntu Virtual Machine. While it would be nice to be able to work on Windows and have a server running in a local Virtual Machine, this requires a very intimate knowledge of Virtual Machines, Linux permissions, and Windows permissions. For instance, the instructions ask you to change the permissions of the server folder using chmod. However, Windows does not necessarily respect these changes, and NextCloud will refuse to load if your permissions are incorrect. If you are looking to get started coding as quickly as possible, this option is probably not for you.



#### "Data directory is readable for other users. Please change the permissions to 0770 so that the directory cannot be listed by other users."

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

You may wish to try a manual installation using [occ](https://docs.nextcloud.com/server/9/admin_manual/installation/command_line_installation.html) (filling out the necessary strings):

```bash
sudo -u www-data php occ maintenance:install --database "pgsql" --database-name "" --database-user "" --database-pass "" --database-host "" --admin-user "admin" --admin-pass "password2"
```



