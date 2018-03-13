# Udacity Full Stack Nanodegree Project: Linux Server Configuration

### IP + SSH
**IP:** 172.104.19.53 
**SSH PORT:** 2200 

### URL
**Portfolio Site:** http://terencefox.me 
**Item Catalog:** http://catalog.terencefox.me 
**Neighborhood Map:** http://map.terencefox.me

### Installation Process
I did this project on Linode. A brief outline of the steps:
1. Create an Ubuntu 16.04LTS image
2. Login as root
3. Update & Upgrade
4. Set hostname & update loopback domain
5. Set timezone
6. Added a new user with sudo privileges
7. New SSH session with the new user 'terence'
8. Generated a key-pair on my local machine and SCP'd the public key to the proper folder in 'terence' home directory
9. Confirmed key-pair worked and then edited `sshd_config` to deny root login, disable password authentication and set the port to 2200
10. Installed UFW. Disallowed all ports except the new SSH port 2200, port 80 and port 123. For testing, I also enabled the Flask default port 5000 but I turned this off after the Flask app was running.
11. Installed Apache2
12. Installed mod_wsgi
13. Installed git
14. Installed PostgreSQL
15. Created a limited-access user and associated database
16. Cloned my itemcatalog git project into /var/www
17. Created a virtual environment for the app and got it running with PostgreSQL.
18. Created a wsgi file that activates and uses the virtual environment and sets the database URL as an environment variable the app will look for.
