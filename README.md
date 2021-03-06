# The Enterprise System Administration Workshop

In this workshop we plan to 

1. Show how to set up and create a CI/CD environment with Jenkins

2. Deploy a testing/staging for our website with devops principles.

3. Deploy a production environment for our website.

Notice that while the website, and the api are very simple, something that would be trivial to deploy. These methods are
for websites people would continuously develop, these are mean't to be applied to start ups small enterprise or even
apps on a large or small scale. So apps that while they don't have to be massive apps, they are generally mean't to be
apps that teams of people would work on.

## Prerequisites

In terms of knowledge it would be helpful but not necessarily required to knowledge

- Basic unix/linux file structures and directory structures (not essential)
- Basic unix/linux commands (just look them up if you struggle)
- Vim or Vi text editing (can also use the nano text editor, but vscode is not gonna help)

In terms of what software would be required to follow along in case you want to follow along which is not required, can
just listen.

1. A github account connected to your university or instituition (see 2)

2. Would need the student developer pack from github, should take about three seconds and a university email to apply
   for one.

3. Namecheap domain (free with student developer pack and a .me extension) Look up instructions on github student pack
   and google.

4. Some vps (digital ocean droplet recommended) just the account needed setting them up will be covered, also doing
   these things are provider agnostic, i'll  be using digital ocean but these apply to all platforms, since they all
   provide similar experiences. 

5. You NEED some way of getting around, some sort of unix shell. Instructions
   below.

### Windows Unix Shell

In windows there are many options for getting unix shells. Putty is one, but not recommended for system administration
work. The two most recommended options are to either use a Virtual Machine, but they can be quite tedious to set up, or
you can use wsl. It is recommended that you use wsl.

WSL stands for windows subsytem for linux, and is essentially a linux kernel embedded in windows allowing windows users
access to the linux cli.

In order to get it follow this guide https://docs.microsoft.com/en-us/windows/wsl/install-win10

I would personally recommend downloading either Debian, Fedora or openSUSE for WSL.

Debian has the most available packages, but you will only be a few here. OpenSUSE Leap, is well regarded as one of the
best linux distros with good default tools which would probably be my own personal choice. Then Fedora is very similar
to CentOS which is what we will be using as the deployment environment, generally it is a good idea to have your
development environment be as close to your deployment environment as possible.

The default packages installed with wsl should be fine.

### MacOS

Make sure ssh is installed and enabled, just the client, not the server.

It is recommended to install vim or a cli text editor.

It is also recommended to install rsync for sysadmin work even if it isn't goint to be user here.

### Linux

See the macos for packages, but everything else should be fine.

## Introduction: What is devops and continuous delivery

From what I have read on the internet, most sys admins describe devops as a way of life.

Now devops is seperate from the agile methodology but they often are used together and I find that they are easier to
explain together.

Before the way software used to be developed is you would have a phase of development, then you would extensively test
the app, and then you would release (deploy) the app. These would happen in big increments of months or years and follow
a strict plan or schedule, then after this process you would discard and move onto the second version, often a complete
rewrite. They would often have a support period where they have hotfixes and patches but rarely big rewrites or new
features often resulting in overall outdated and less quality in software.

The agile methodology takes the same simple idea of having a development -> test -> deploy cycle but instead these
happen in smaller increments, and often have not quite extensive testing, then they wait for new issues to arise whether
it be from the development point of view, or the users point of view, and then repeat the cycle. It's agile because it
is a cycle of feedback and incremental versions.

While in theory agile works, in practice, having to redeploy the app every increment or version or redo testing is a
tedious and not worth while job. It makes the agile methodology fall short. Devops and continuous delivery is a process
used to automate the process of deployment and testing. These convert the testing and the release phases of agile into
what we would call a devops pipeline.

A devops pipeline looks like.

Push from source control -> Staging -> Deploy

Where the first step is the development process.

Staging is where you build the app, extensively test it, then you create what we call artifacts such as the actual
release binaries of the app or in our case digital ocean droplet images with our app installed.

Deployment is the task of configuring the server for production, and making it accessible on the internet, taking the
artifacts from the build process.

Dev ops really is anything with a automation pipeline from source code to deployed, that means that this can be
something like github actions, gitlab ci/cd, travis ci or whatever your hearts desire. Jenkins is a middle ground CI
for more professional uses that allows you to self host a CI/CD instance and is what we will be using. Larger
enterprises like google might use kubernetes and docker, using a pipeline to build and test on docker, then deploying
with kubernetes on multiple servers.

## Steps to setup postgres

1. Install postgres
2. switch users with `sudo -iu postgres`
3. Create a new user `exampleuser123` with `createuser --interactive` and set
   all permissions to none.
4. Create a new database with `createdatabase` called `todo_db`
5. Go into psql with `psql` and type `ALTER ROLE exampleuser123 WITH PASSWORD 'password123';`
6. Now run `GRANT ALL PERMISSIONS ON DATABASE todo_db TO exampleuser123;` and quit with ctrl+D


## PART 1: Setting up a VPS CentOS Instance for Jenkins

For the actual deployment and testing I will use a instance of jenkins deployed on my personal computer. If you want to
work in a team based environment you would need to setup a seperate server for CI/CD. These steps will show you how to
do that, however since hosting costs $$$ we will delete the container as soon as its made.

### A quick note on choosing linux distributions

So for devops servers it does not matter what we choose as our operating system, as long as it fullfills our needs, if
anyone has been watching the news on Linux CentOS you would have heard of the controversy of CentOS a typical downstream
of rhel (RHEL is a paid linux distribution for running web services and centos copies the source code) to an upstream
distro (basically RHEL beta). This means some of the security benefits CentOS provided are no longer there, but most
importantly to sysadmins is that CentOS used to have a support cycle of around 10 years, and that has now shifted to 3
years, therefore CentOS is no longer a recommended distro for deploying high availability web applications. On the other
hand, using CentOS 8 (or stream) for deploying CI/CD instances is a great idea since there is no need for predictability
or high stakes security management. For deploying the actual web app we plan on using Rocky Linux, a community based
RHEL fork, but you can choose whatever Long Term Support distro you want (recommend debian, opensuse or freebsd).

### Create some digital ocean accounts

First create some account of a vps service, some recommendations

- Digital Ocean Droplet
- Google CentOS Instance
- Microsoft Azure
- Amazon AWS

### Create a VPS Instance

#### Digital Ocean

1. Get started with a droplet after you log in

2. Select CentOS 8.3 as the distribution

3. Select a relevant storage size, cpu and memory, as long as they are standard shared cpu droplets, it doesn't matter
   which one you pick.

4. Choose any region, I shall be choosing singapore because it is closest to me in sydney.

5. Choose Password instead of ssh, and don't select User Data, although if you are familiar with these options then you
   can go ahead and select them. Select a root password and continue.

6. One droplet with any available hostname, shall be using the default hostname

7. Make sure to select the backups option.

8. Wait for the droplet to be created.

### Setting up your website url

We will be using namecheap for three reasons, 1. It's free, 2. It has whois guard protection
meaining your private information stays private, and 3. It really doesn't matter which
provider you use since they are all pretty much the same, and we won't be using
namecheaps dns anyway.

1. Log into namecheap.com and navigate to Dashboard.

2. Navigate to your domain, and select manage

3. Under name servers, change to custom dns, and set the nameservers to the following

```
ns1.digitalocean.com
ns2.digitalocean.com
ns3.digitalocean.com
```

4. Log back into digital ocean and on the `...` next to your droplets name, select add a domain

5. Add these records

```
CNAME Record:
hostname: www 
is an alias of: @

A Record:
hostname: api
will direct to: your droplet

A Record:
hostname: jenkins
will direct to: your droplet
``` 

### Setting up SSH

Setting up SSH is pretty easy

1. Run the command `ssh-keygen` and simply press enter through all the options. Note that if you would like to set a
   password that is fine.

2. Next add ssh to your github account, click your profile icon in the top right, select `settings`, then under SSH and
   GPG keys click New SSh key on the top right. Copy the your key with the command `cat .ssh/id_rsa.pub` this will print
   your PUBLIC key to the terminal, as `cat` prints files to the terminal, and `.ssh/id_rsa.pub` is the location of
   where your key is stored. ssh keys have a private and a public component. The private is the key and only you can
   access it, the public key is like providing the lock that they key fits so never give your private key away.

3. now in your server instance, find what the servers public ip is (remember to provide instructions for this) and run
   the command `ssh-copy-id root@yourdomain` where the server ip is the public ip address to your server. (Note:
   sometimes it takes a while for domains to get registered when switching nameservers, in that case use the ip)

We done for now.

### Note on systemctl and services

All operating systems have software that runs in the background, they do things like make sure that the time is
synchronised with the rest of the world, they provide resources for apps that run in the foreground. These are commonly
called Daemon services.

In windows it is kind of difficult from my experience (not a windows user tho) to create and manage services. If we want
to make an app that runs when we start up our computers and runs in the background it can be quite tedious. Linux on the
other hand has a tool that allows us to do this, it is called systemd and can be managed using systemd service files
usually with the extensions of `.socket` and `.service` and managed with `systemctl`. Read more on the
[archwiki page](https://wiki.archlinux.org/index.php/systemd) 

Systemctl has the commands

- `systemctl start servicename` to start a service called servicename
- `systemctl stop servicename` to stop a service called servicename
- `systemctl restart servicename` to stop then start a service called servicename
- `systemctl status servicename` to see if the service activation actually worked
- `systemctl enable servicename` to make sure that the service starts on system start up e.g. when
  you turn on your computer or when you restart your computer.

You can view the systemd logs with the `journalctl` command.

`.socket` files often are services that start when they are needed, like a printing service only starts when you need to
print something, and `.service` files are for services that run all the time.

You can also specify services for specific users only but read the archwiki for that.

### A note on the linux filestructure

The linux filesystem structure is a standard created by the linux foundation and specified under the Filesystem
Hierarchy Standard, more information can be found on
[the wikipedia page](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

This isn't too important here except when we get to setting up nginx, and is a bit of a point of contention among
system administrators. To understand why there are some important directories which we have to explain.

The root `/` is where everything goes into, equivalent of the `C:` drive in windows.

The `/etc` directory is where applications put there global configurations, that when they start all configuration
options and files are mean't to be put here.

The `/usr` directory specifies read only files used by apps, a common example would be icons, music or stuff in a game,
they are read only because they never need to be modified.

The `/var` directory is kind of like the read and write counter part of the `/usr` directory, things that do need to be
modified but aren't configuration files often go here.

The `/srv` directory is what causes the controversy, according to the linux standard, files which are mean't to be used
for servers such as directories which are used as remote file storage like google drive kind of things go here,
and static website files.

In nginx it has become standard to put your website in the `/var/www` directory, this to me personally isn't the right
place to put them, some system admins share this opinion while others feel that it is better to follow the standards
of nginx for consistancy. Nearly all guides on the internet will use `/var/www` while we will be using `/srv/www`
instead.

### Users and permissions

In linux we have users and groups

Linux contains a simple permissions model, each service has permission information, they can be read, written, or
executed (like opening up an app in windows). You can view permission with the `ls -a` command, it appears with
something of the form of `drwxrw-r-- user group`, the first letter `d` tells us whether it is a directory or not. The
next three characters are in the form of `rwx` where `r` is the read permission, `w` is the write permission, `x` is the
execute permission, and `-` says they do not have the given permission. A file often belongs to a user and a group, the
permissions tells us what permissions the user who owns the file has, what permissions the group who owns the file has
and what permissions everyone else (others) has.

Example:

`rw-` - Read and Write permissions but no execute.

`r-x` - Read and execute but no write permission.

Note that `rwx` is repeated three times, the first time is the read, write and execute permissions for a user, the
second is for a group, and the third is for the entire system commonly called others.

In linux we want to restrict access as much as possible to prevent things from having access to other things they
shouldn't.

In linux a user is like a user of an account like in windows or mac, and a group is just a group of users. When we add
permissions to a specific user, only that user can access that resource, but if we need some users to access a resource
but others not be able to access a given resource, then we can create a group. 

Note on `gpasswd` vs `usermod`

There are two ways to add a user to a group, `usermod -aG somegroup username` and `gpasswd -a username somegroup`

The difference is subtle, but `usermod` modifies the users configuration with options such as changing there default
terminal shell, or adding and removing groups.

`usermod` has two options that we will look at `-G` which gives a user certain groups, and `-a`, which adds the
pre-existing groups that a user had to the user. Now the problem is if you accidentally forget the `-a` flag, you end up
removing the groups a user alread had, and so this can be considered more risky, in practice this rarely happens. On the
otherhand `gpasswd` can either add or remove groups so it is therefore a safer option.

1. run the command `adduser exampleuser` where the user is your username in my case emendoza.

2. set a passwd for your user with the command `passwd exampleuser`

3. run the command `gpasswd -a exampleuser wheel` this adds our user to the wheel group. In Linux, the wheel group is
   like the admin role in windows.

4. Logout of ssh with Ctrl+D or type `exit` into the cli.

5. Now repeat the command `ssh-copy-id exampleuser@yourdomain` notice we are now using the example user instead of root.

### Install required software

1. Run command `sudo dnf upgrade` to make sure we are downloading the latest versions.

2. Run command `sudo dnf install vim-enhanced`

3. Run command `sudo dnf install epel-release` to install the red hat extra packages.

### Change ssh settings

We will use vim for this, if you are unfamiliar use replace `vim` with `nano`, but remember, as described in the Unix
and Linux System Administration Handbook System administrators will judge you so do it descretely if you plan to deploy
infront of other system admins.

1. enter the command `sudo vim /etc/ssh/sshd_config`

2. change the line `PermitRootLogin yes` to `PermitRootLogin no`

3. change the line `PasswordAuthentication yes` to `PasswordAuthentication no`

4. quit vim with `:wq`

5. reload ssh daemon with `sudo systemctl reload sshd`

6. Test it works before you exit by entering a new terminal and typing `ssh exampleuser@yourdomain`

### Firewall

A server works by exposing all ports available to the outside world. As system administrators one of our primary jobs is
to restrict the access of the outside world to be able to only access what they need. So we can directly access our
server, but nobody else should be able to, similarly when we serve our website, we want people only to be able to access
what they need to see the website, we don't want them to be able to access anything else. A firewall makes it so that
people can only access ports that we specify, with all other ports still being accessible internally.

#### About firewalld

For this workshop we will use firewalld, it works by having default zones, a zone is an area that the network runs in,
for example, you might have a zone for local computers in an office, and a zone for computers outside in the public, or
a zone for just the computer specifically. Each zone might have certain services enabled such as you might have http
enabled publically for everyone, but only have ssh enabled for users within the local network.

More information about firewalld can be found by googling archwiki firewalld, and you can see what commands are
available by reading the manual with `man firewall-cmd`.

1. Run command `sudo dnf install firewalld`

2. start firewall service with `sudo systemctl start firewalld`

3. add ssh to the firewall permissions list with `sudo firewall-cmd --permanent --add-service=ssh`

4. type `sudo firewall-cmd --relaod` to enable changes

5. similar to before open up a new terminal and see if you can still access the server.

6. if it all works enable changes with `sudo systemctl enable firewalld`

### Timezones

It is worthwhile to change the timezone of the server to the local timezone you are in especially dealing with databases
and such. As a side note I often change the local timezone of my databases and api's to UTC and let region specific time
information be handled on the client side.

1. Find which timezone you want by running the command `sudo timedatectl list-timezones`

2. Navigate with the j and the k keys `j` for up and `k` for down. You can also search with the `/` command. for example
   `/Australia` for all regions in australia.

3. Next set your region as default with `sudo timedatectl set-timezone region/timezone`, for example,
   `sudo timedatectl set-timezone Australia/Sydney`

4. Confirm with `sudo timedatectl`

### Jenkins

We shall be installing jenkins, jenkins is what enables devops in our server. Generally speaking
when creating a dev ops pipeline you need some sort of software to automate tasks. There are
different kinds of devops tools some focusing on continuous integration, and some focusing on
continuous delivery. Continuous integration is where you run automation on the testing and
building phases of app building. It is more important for projects like desktop and mobile applications
where you only need to release the binaries, so deployment isn't taken into consideration, examples of
this are travis ci, and github actions. Notice these all are seperate services that integrate with github,
on the other hand continuous delivery is the process of automating the deployment stage, quite often they
have elements of continuous integration, but it is more important that every push you have a functioning
web server. Examples of this are heroku, gitlab/github pages. The other important thing is that continuous
delivery is pretty much always always part of the server in which you want to deploy on rather then being
a seperate service as compared to github actions and travis. The most common application today used for
continuous delivery is Jenkins because of it's free and open source nature, meaning that you can modify,
extend and use it free of charge. Gitlab CI/CD is quickly taking off as well, and others incluce Atlassian
Bamboo and Jetbrains Teamcity.

We shall be using Jenkins because it's free and it's a transferrable skill, with a lot of development jobs
asking specifically for experience using Jenkins.

For a comprehensive guide of jenkins, check out it's extensive documentation which we will be using for the
instructions to set up nginx and jenkins. https://www.jenkins.io/doc/book/

Jenkins runs on Java 8 at this point in time only sadly so don't try and use a more recent jdk.

1. Enable the jenkins repository (tell centos where to download jenkins from) with 
   `sudo curl -o /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo`

2. Import the hash keys for jenkins repository (make sure that we are really downloading jenkins rather then virus')
   with `sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key`

3. Upgrade all the repositories to make sure we are downloading the latest jenkins with `sudo dnf upgrade`

4. Finally install jenkins with `sudo dnf install jenkins java-11-openjdk-devel`

5. reload systemctl `sudo systemctl daemon-reload`

6. Now start the jenkins service with `sudo systemctl start jenkins`

7. Check if it worked with `sudo systemctl status jenkins`

8. If everything worked correctly enable on startup with `sudo systemctl enable jenkins`

We are done installing jenkins but we will revisit later for some configuration in the nginx section.

### A quick note on SELinux

We will have our first experiences dealing with the pain in the ass that is selinux. SELinux stands for Security
Enhanced Linux and is like an extra more intense linux permissions layer. Instead of files having just read, write and
execute permissions they also have policies about what they are allowed to do on the system and what they are used for.
SELinux on the scale we are using here isn't too bad, and since it is so common for linux files and folders to be used
for what they are, most of CentOS already has default SELinux policies that we need to enable or restore manually. As a
website scales, SELinux becomes harder and harder. Learning SELinux is kind of a thing on it's own and so I wouldn't
stress about it.

SELinux has three modes

- Enabled: if a selinux check fails the service cannot run
- Permissive: if a selinux check fails an error is logged but service still runs
- Disabled: selinux policies aren't checked at all.

Generally it is a good idea to turn selinux to permissive, while deploying a server and change it to enabled later.
[Changing Selinux state guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/changing-selinux-states-and-modes_using-selinux)

### Nginx

Nginx is kind of like the server part of our server, we use it to manage routes and http access to our server. Most
backend web frameworks like flask, django, actix, and others don't require it since they can serve the html files 
themselves. On the other hand I often use it anyway because it can be used to easily add advanced features like
compression, https and http2 to our web pages, and in the example of jenkins, we can use jenkins by itself now, but we
would need to access an ugly route like https://www.howgood.me:8080, nginx allows us to instead use the route in the
form of https://jenkins.howgood.me

#### Installing nginx

Nginx is in the extras repo for centos which we already installed before.

1. Simply type `sudo dnf install nginx`

2. Start nginx with `sudo systemctl start nginx`

3. Check if nginx is working with `sudo systemctl status nginx`

4. If everything works enable with `sudo systemctl enable nginx`

5. Now add nginx to firewall with `sudo firewall-cmd --add-service=http`

6. Also add https for later `sudo firewall-cmd --add-service=https`

7. Save changes with `sudo firewall-cmd --runtime-to-permanent`

#### Setting up blocks in nginx

In nginx you can set up 'blocks' which point to specific domains, that means you can have multiple different domains
that point to the same server!

We will only be using a single block so it's kind of pointless but it's handy for using later if for example you wanted
to deploy a static website and an api on the same server.

1. Make a server staging configurations directory with `sudo mkdir /etc/nginx/sites-available` this is where we write
   our server configurations.

2. Make a server deployment configurations directory with `sudo mkdir /etc/nginx/sites-enabled` this is where our
   finished configurations go.

3. Now we need to modify the `/etc/nginx/nginx.conf` file, I will use the command `sudo vim /etc/nginx/nginx.conf`
replace vim with nano if you aren't comfortable with vim.

Go to where is says `http {` and scroll to below the matching `}` bracket and add the following lines after the bracket.

```
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```

4. last step is to restart nginx to enable changes with `sudo systemctl restart nginx`

#### Setting up jenkins with nginx

Up until now we have been using vim and manually writing the files, but if you look at the file
`nginx_configs/jenkins.yourdomain.me.conf` downloaded with the project it is quite big!

Of course you are welcome to manually rewrite it all into the file
`/etc/nginx/sites-available/jenkins.yourdomain.me.conf` but that is a pain so we are gonna cheat a little.

First step is to modify the file in whatever text editor you like edit the file
`nginx_configs/jenkins.yourdomain.me.conf`

1. Rename the file to have your domain name (eg `jenkins.howgood.me.conf`).

2. Edit the file where it says `server_name` to your actual domain (eg `jenkins.howgood.me`).

We are gonna use rsync to copy the file to our server with

`rsync nginx_configs/jenkins.yourdomain.me.conf yourusername@youdomain:~`

Next we are gonna move the file with the `mv` command to  `/etc/nginx/sites-available/jenkins.yourdomain.conf` note that
mv is the same as cut in windows.

`sudo mv jenkins.yourdomain.me.conf /etc/nginx/sites-available/jenkins.yourdomain.conf`

Now we have to make an error log directory

`sudo mkdir /var/log/nginx/jenkins`

Add a symbolic link (look up symbolic link if you don't know what it means) with the sites-enabled folder.

`sudo ln -s /etc/nginx/sites-available/jenkins.yourdomain.conf /etc/nginx/sites-enabled/jenkins.yourdomain.conf`

Test with nginx test

`sudo nginx -t`

Now we gotta mess with permissions and users

1. First change the user to root with `sudo chown root /etc/nginx/sites-available/jenkins.yourdomain.conf`

2. Now change the group to root with `sudo chgrp root /etc/nginx/sites-available/jenkins.yourdomain.conf`

3. Now we have to give the jenkins user the group nginx with `sudo gpasswd -a jenkins nginx`

4. Run the command `sudo restorecon -Rv /etc/nginx/sites-available` to add the proper selinux policies for all the files
   in that directory.

5. Add selinux permissions for http services to forward other http services with
   `sudo setsebool -P httpd_can_network_relay 1`

6. Restart nginx with `sudo systemctl restart nginx` and open your browser and navigate to http://jenkins.yourdomain

### Certbot and https

Since we will be using passwords and transferring secure information our next step must be to add http over tls support
to our jenkins (CI) server. For this there are multiple ways including buying a expensive certificate, another way is to
use a free lets encrypt certificate. The benefit of buying a certificate is that they provide higher validation levels
are generally more trusted and provide warrantys. The biggest benefit is probably that they are renewed less, free
certificates are alright and generally way more secure then not having one, although, since they are provided free they
need to be renewed every three months. 

We will use a free tool created by lets encrypt in order to obtain there certificates called certbot.

1. install certbot with `sudo dnf install certbot python3-certbot-nginx`

2. obtain a certificate by typing `sudo certbot --nginx -d jenkins.yourdomain`

3. type your email address used for renewal and security purposes (for me it's 'admin@effectfree.dev')

4. Read through the options and select which ones you want (I just put yes for everything)

5. Once this is complete you can test your configuration with the [SSL Labs Server Test](https://www.ssllabs.com/ssltest/)

6. Now setting up auto renewal requires cron.

#### A Quick Note on cron

In linux system administration we often want to automate tasks needed to be performed every day, or every
few days. As such there are multiple ways to do this. One of them is cronjobs. Cron is a service that will
run a service or a script or something after a set period of time. Another alternative is to use systemd,
systemd has multiple file types, some common ones are timers which run at a set period in the day, sockets
which listen and activate software when needed, and services which run through the duration from startup to
shutdown. Creating cron jobs is easy, we can use the crontab command to quickly create cronjobs.

First test nginx renewel with `sudo certbot renew --dry-run`

`sudo crontab -e`

add this line

```cron
...
0 3 * * * certbot renew --quiet
```

this command makes cron run certbot renew at 3am every day.

### Final Jenkins Setup

1. Navigate to your website in your web browser now that you have set up http over tls (e.g. https://jenkins.howgood.me)

2. Run the command `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

3. Copy the string printed from the terminal to the jenkins site

4. For plugins deselect everything then select
   - Dashboard View
   - Folders
   - Build Name and Description Setter
   - Build Timeout
   - Credentials Binding
   - Timestamper
   - Workspace Cleanup
   - Pipeline
   - Pipeline: Stage View
   - Git
   - GitHub
   - SSH Build Agents
   - Matrix Authorization Strategy
   - PAM Authentication
   - LDAP
   - Email Extension
   - Mailer
   - Publish Over SSH
   - SSH

5. Fill in the user details for your jenkins instance.

### Saving Jenkins as a backup for future use

We might want to save what we have just built as a digital ocean image for reuse later on.

1. Navigate to digital ocean and select your droplet

2. Turn off your droplet

3. Navigate to snapshots and select take snapshot

4. Congratulations you have finished setting up a CI Server

### Last installing dependencies

In order to build our project in jenkins we need to install some last dependencies in our devops server.

1. Installl elixir with `sudo dnf install elixir erlang`

2. Install nodejs with `sudo dnf install nodejs`

## Part 2: Configure building, testing and deployment for api with jenkins

### Part 2.1: Checking out from source control and unit testing

#### Setting up our first build pipeline

Our first step is to setup the initial pipeline project

1. First of all fork the project from the macquarie dsc github to your own personal github at
   https://github.com/Macquarie-University-DSC/tasks_api

2. Clone your project into your own computer
   1. Click Code in the top right

   2. Copy the clone address either https or ssh if you have ssh setup

   3. Run the command `git clone your_repo_that_you_copied.git`

3. Go to your jenkins server that you had just setup and click Create Job on the dashboard

4. Enter the name for your DevOps Pipeline, I am just gonna call mine TasksAPI

5. Select Pipeline, and then click OK

6. Give our pipeline a description, like "DevOps Pipeline for the Tasks API"

7. Select Discard old builds, put how long you want to keep your build (empty for me), and Max number of builds (20 for me).

8. Select Github project and add the url to your forked github repo

9. Under Build triggers, click GitHub hook trigger for GITScm polling

10. Under Pipeline click Pipeline script from SCM, select the SCM as Git, and enter the clone url you used before.

11. Under Script Path type `pipeline/Jenkinsfile`

12. Now click Save

### Service Daemons, and more SELinux

## Part 3: Deploy a static website with nginx, http2 and https with ansible+jenkins

## Nginx statis site stuff

1. First make the directory we will put our html files in with `sudo mkdir /srv/www`

2. Give this directory read write and execute permissions for sudo user, read
   and execute permissions for the group, and others with 
   `sudo chmod 755 /srv/www` and check with `ls -l /srv`

3. Run the command `sudo restorecon -v /srv/www` to add proper selinux policies
   to this directory.


### Setting up Jenkins and github integration

### More Users, Groups and SELinux
