# Udacity Full Stack Nanodegree Project: Linux Server Configuration

### IP + SSH
**IP:** 172.104.19.53  
**SSH PORT:** 2200

### URL
**Portfolio Site:** http://terencefox.me  
**Item Catalog:** http://catalog.terencefox.me  
**Neighborhood Map:** http://map.terencefox.me

## Securing Your Server
Step 1.
Login as root:  
`ssh root@0.0.0.0`

Step 2.
Update and upgrade:  
`apt update && apt upgrade`

Step 3.
Name the machine with an `example hostname`:  

`echo "example_hostname" > /etc/hostname`  
`hostname -F /etc/hostname`

Change the loopback domain accordingly:  
`sudo nano /etc/hosts`  
Add the line:  
`127.0.1.1 example_hostname`

Step 4.  
Set the timezone for log readability:  
`dpkg-reconfigure tzdata`

Step 5.  
Add a new user:  
`adduser example_user`  
It will prompt you for a password - this can be anything easy to remember as password access will be disabled shortly.


Add it to the sudo group to give the user sudo access:  
`usermod -aG sudo username`


Step 6.
On your local machine, generate a new SSH key-pair:  
`ssh-keygen -b 4096`  
**A Note about Key Management:**  
At this point, the prompt will show you the default path and file name. SSH will look for a key in this location with this filename whenever you log in. If you've never made an SSH key-pair before, go ahead and press enter to use the default. If you have, you need to specify a new path and filename to avoid overwriting your existing key-pair. Make sure the path is still the .ssh folder in your home directory.


Step 7.
Log out of the root ssh session. Log back in as the user you just created and run a sudo command to ensure that the user was set up properly:
`ssh example_user@hostname`
In your ssh session, make a directory for the ssh key and give it the proper permissions:  
`mkdir -p ~/.ssh && sudo chmod -R 700 ~/.ssh/`

Step 8.
From your local machine, add the contents of your public key into example_user's authorized_keys file:
On Mac:  
`scp ~/.ssh/id_rsa.pub example_user@[203.0.113.10[IP_ADDRESS]:~/.ssh/authorized_keys`

Step 9.
Log out of the ssh session and log back in. If there was no password prompt, the ssh key is working and you can move on to disabling password access and root login.

`sudo nano /etc/ssh/sshd_config`

Change the following lines:

`PermitRootLogin` change `yes` to `no`  
`PasswordAuthentication` change `yes` to `no`

While you’re here, move SSH off the default port:  
`# What ports, IPs and protocols we listen for`  
`Port` should be `2200`

Restart the ssh service

`sudo service sshd restart`


Step 10.
Try to log in as root to ensure it’s been disabled. Login as the limited user to set up UFW -  note the `-p` flag to specify the port. If you created the SSH key with a different name or path than the default, specify it with the `-i` flag.
`ssh user@hostname -p 2200`

Confirm it’s inactive:  
`sudo ufw status`  
Close all ports to incoming:   
`sudo ufw default deny incoming`  
Open all ports to outgoing:
`sudo ufw default allow outgoing`  
Reopen the incoming ports you’ll need:   
`sudo ufw allow 2200` (For SSH)  
`sudo ufw allow www`  
`sudo ufw allow ntp`  
Enable the firewall:  
`sudo ufw enable`  

Step 11.
Now that we’ve moved off the default SSH port it’s handy add a config file to your local machine to create a nickname for a particular SSH profile.  
`sudo nano ~/.ssh/config`

Specify the following lines for your server:
```
host example_hostname (To make it easy to remember but could be anything)
Hostname 0.0.0.0 (your server's IP address)
Port 2200
User example_user
```

Now you can sign in simply with `ssh example_hostname`. If you're not using the default SSH key-pair you can specify the full path and filename with a line that starts with `IdentityFile`

## Deploying To Apache
### Installation and Deployment
Step 1.  
Install Apache2:  
`sudo apt update`  
`sudo apt apache2`

Change the default .conf file to point to your server’s ip address or associated domain:

`sudo nano /etc/apache2/sites-available/000-default.conf`  
`ServerName [IP OR DOMAIN]`

Restart apache:   
`sudo service apache2 restart`

Open your domain or IP in a browser and the default Apache page should appear.

Apache accesses files in a few different locations on the system. Default locations for Ubuntu (these will be referred to later):

```
ServerRoot              ::      /etc/apache2
DocumentRoot            ::      /var/www
Apache Config Files     ::      /etc/apache2/apache2.conf
                        ::      /etc/apache2/ports.conf
Default VHost Config    ::      /etc/apache2/sites-available/000-default.conf, /etc/apache2/sites-enabled/000-default
Module Locations        ::      /etc/apache2/mods-available, /etc/apache2/mods-enabled
ErrorLog                ::      /var/log/apache2/error.log
AccessLog               ::      /var/log/apache2/access.log
cgi-bin                 ::      /usr/lib/cgi-bin
binaries (apachectl)    ::      /usr/sbin
start/stop              ::      /etc/init.d/apache2 (start|stop|restart|reload|force-reload|start-htcacheclean|stop-htcacheclean)
```

Step 2.  
We're going to use our standard admin user to deploy our projects to the server using git. Apache has a user and a group both named `www-data` that accesses the files in the above directories. Thus the next step is to change the permissions for the DocumentRoot so that only our admin user and Apache can access it. First we add our user to the www-data group:  
`sudo usermod -a -G www-data example_user`  
Change the owner and the group of the root directory:  
`sudo chown -R www-data:www-data /var/www`  
Deny any user outside the group access:  
`sudo chmod -R o-rwx /var/www`  
Give the www-data group full permissions:  
`sudo chmod -R g+rwx /var/www`  

Step 3.  
If you just changed your current user's group, you need to log out and log in again for the change to take effect, so do so now. Use the `id` command to confirm your user is in the `www-data` group. Next, install git:  
`sudo apt install git`  
Navigate into the DocumentRoot directory:  
`cd /var/www`  
Now clone your repo into it:  
`git clone url`  
Git will prompt you to enter your login info.

See below for next steps depending on what kind of project you're deploying.

### Configuration - Static Site
Step 1.  
**Note:** The placeholder `project` should be the name of the directory you just cloned for organization and convenience.

Create .conf file for the project:
`sudo nano /etc/apache2/sites-available/project.conf`

Refer to the [documentation](https://httpd.apache.org/docs/2.4/vhosts/examples.html) but a
typical simple configuration looks like the following:
```
<VirtualHost *:80>
    ServerAdmin you@example.com
    ServerName [IP OR DOMAIN]
    DocumentRoot /var/www/project (this is the directory we cloned from git)
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Step 2.  
Apache keeps track of which sites are enabled and disabled with symlinks in `/etc/apache2/sites-enabled`. Don't modify this directory directly. Use the following command instead:  
`sudo a2ensite project`  
Then reload Apache:  
`sudo service apache2 reload`

If you need to make changes, disable the site and then follow the above steps to re-enable it and reload Apache:  
`sudo a2dissite project`

### Configuration - Web Application
Step 1.
**Note:** The placeholder `project` should be the name of the directory you just cloned for organization and convenience.  
This configuration is going to be for a Python application developed with the Flask framework. Flask uses WSGI so we need to install that as an Apache module first:  
`sudo apt-get install libapache2-mod-wsgi`  
Once it's installed, restart Apache:  
`sudo service apache2 restart`

Step 2.  
Next we need to set up a slightly more involved configuration file than for a static site. The following is what I used. It has three options that tell WSGI to run the application as a daemon process, sets the thread-count to 5, and tells it to watch my wsgi file (which we will create next) and reload the site if it changes. Note that the directory directive points to the app package itself, not the root directory.

`sudo nano /etc/apache2/sites-available/project.conf`  
```
<VirtualHost *:80>
                ServerName project.domain.com
                ServerAdmin terence.fo@gmail.com
                WSGIDaemonProcess catalog threads=5
                WSGIScriptAlias / /var/www/project/project.wsgi
                <Directory /var/www/project/project/>
                        WSGIProcessGroup project
                        WSGIApplicationGroup %{GLOBAL}
                        WSGIScriptReloading On
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Step 3.
Next we create the wsgi file at the root directory of the project. This is going 
