## Linux Server configuration project
### by Yanyan Wu @ *June 2018*

## Overview 
Use  Amazon Lightsail to set up  a baseline Ubuntu Linux server instance and  to host the product catalog application that we developed earlier in this course (Here is the github link to the product catalog application I developed earlier in the full-stack class: https://github.com/yanyanwu0358/catalog ). 

## Project requirements 
1. Change the SSH port from  **22**  to  **2200**. Make sure to configure the Lightsail firewall to allow it. 
2. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
3. Create a new user account named `grader`. Its account need to enforce Key-based `SSH` authentication.
4. All system packages have been updated to most recent versions.
5. `SSH` is hosted on non-default port.
6. The web server responds on port 80.
7. A README file is included in the GitHub repo containing the following information: 
## Final deliverable information:
1. IP address: http://18.220.11.125/
2. URL: http://ec2-18-220-11-125.us-east-2.compute.amazonaws.com
3. Linux command to log into `grader` account:  `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.220.11.125`
4. Software installed: Python3, Python3-Pip, Postgresql, and other Python libraries (details in set up process below)
5. Major configurations made: see details in set up process below.

## Set up process
### 1. Create a Lightsail instance on AWS

-   Go to https://aws.amazon.com/. Click on the “Services” link in the top navbar and find “Compute section", select Lightsail
-   On the Lightsail page, click on “Create instance” page
-   Under “Select Blueprint” section click on “OS Only”
-   Select “Ubuntu”
-   Select a plan. Usually the first basic plan, free for the first month then $5 afterwards
-   Under “Name your instance”, rename your instance with any name you wanted
-   Click the “Create” button. The instance will take few minutes to appear on the instances page and it will turn to "running" status which means it's ready for you to use.
### 2.  Download the Lightsail default private key to local computer
-   In the Lightsail home page, click on the instance you just created and stay on the "connect" tab.
-   scroll all the way down to the bottom of the page and click on the blue “Account page” link
-   On the "SSH key pair page", Click on “download” at the bottom of the page, name and save the file on your local computer. In my case, the key pair file I downloaded is named as "LightsailDefaultPrivateKey-us-east-2.pem"
### 3. Copy the key pair file .pem to .ssh folder
- Paste the .pem file in the folder where VagrantFile is located.
- Open the same folder in cmd, and use the following command: vagrant reload
- Then after ssh-ing into vagrant using vagrant ssh, try the following command:  cd /vagrant/. Now you can see that the .pem file is located under /vagrant/
- Now copy the .pem to the .ssh folder using: `scp –i LightsailDefaultPrivateKey-us-east-2.pem ~/.ssh/`
- Then, delete the file from your folder where VagrantFile is located. 
- Make our public key usable : `chmod 600 ~/.ssh/ LightsailDefaultPrivateKey-us-east-2.pem`
- Now test  if we can ssh into Lightsail server using .pem file: `ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@18.220.11.125`

### 4. Add ports 2200/tcp and 123/udp into Lightsail instance

-   On the instance page click on the “Networking” tab,
-   Under “Firewall” section, click on “+Add another” at the bottom of the ports list,
-   Add ports as follow: Custom: TCP:2200, and Custom:UDP:123. (after adding these two ports the list should include at least four ports, 22, 80, 2200, and 123)
-   Click “Save”

### 5. Change ssh port from 22 to 2200
-   Open the sshd_config file: `$ sudo nano /etc/ssh/sshd_config`
-   Change the ssh port from 22 to 2200
-   Save the change
-   Restart the ssh service: `$ sudo service ssh restart`
### 6. Update UFW (uncomplicated firewall)ports and status
-   Allow the new port 2200: `$ sudo ufw allow 2200/tcp`
-   Allow existing port 22: `$ sudo ufw allow 80/tcp`
-   All UDP port 123: `$ sudo ufw allow 123/udp`
-   Check if the ufw is active. If not, do so using the command: `$ sudo ufw enable`
-   Restart the ssh service: $ sudo service ssh restart
### 7. Configure the local timezone to UTC
-   Configure the time zone: `$ sudo dpkg-reconfigure tzdata`
### 8. Create the grader user
-   Switch role to root urser using the command: `$sudo su -`
-   Create the grader user: `$sudo adduser grader` 
-   Specify the grader passcode
-   Create a new grader file under the sudoers directory and assign sudo status to grader by creating the file using: `$ sudo nano /etc/sudoers.d/grader`, and type below into the file `grader ALL=(ALL:ALL) ALL`.
### 9. Create a keypair for grader user
-   Stay on grader user account, or ssh into the grader user account: `$ ssh grader@18.220.11.125 -p 2200 `
-   Create a directory .ssh: `$ sudo mkdir .ssh`
-   Create a file with the name authorized_keys inside the .ssh directory: `$ sudo touch /.ssh/authorized_keys.`
-   Change the permission for the .ssh folder: `$ sudo chmod 700 .ssh`
-   Change the permissions for the authorized_keys file: `$ sudo chmod 644 /.ssh/authorized_keys`
-   Open a new cmd window on your device, cd into the folder where vagrant file sits.
- call `vagrant ssh`. don't log into Lightsail account
-  type the command: `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`
-   Stay on the same Terminal window, input $ cat ~/.ssh/udacity_key.rsa.pub to read the public key. Then Copy the public key.
-  Going back to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by `$` cd /home/grader
-  Open the authorized_keys file created for the grader user: `$ sudo nano /.ssh/authorized_keys`
-   Paste the content
-   Reopen the sshd_config file (sudo nano /etc/ssh/sshd_config) and change password authentication from “yes” to “no”.
-  Change the PermitRootLogin property to "no" in the sshd_config file: `$ sudo /etc/ssh/sshd_config`
- Change the owner from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`
- Restart the ssh service: `$ sudo service ssh restart`
- Disconnect the server by `$ ~.` and then log back through port 2200: `$ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.220.11.125`
### 10.  Install required packages including Python 3.
-   You can also install the user identifier package “finger” : `$ sudo apt-get install finger`
- Install git: `$ sudo apt-get install git`
- `sudo apt-get update`
- `sudo apt-get install apache2`
- `sudo apt-get install libapache2-mod-wsgi-py3`
- `sudo apt-get install python3-pip`
- `sudo apt-get install python3-flask`
- `sudo apt-get install python3-httplib2`    
-  `sudo apt-get install python3-sqlalchemy`  
-  `sudo apt-get install python3-oauth2client`
-  `sudo apt-get install python3-requests`
-  can use pip3 install to install the packages for python3
- check python3 version: `python3 --version`

### 11. Update Linux server
-   Connect to the linux server via either the Lightsail ssh connect or local ssh explained above step 10 with Linux command to log into  `grader`  account:  `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.220.11.125`
-   Update server packages using the following commands in sequence to prevent updates shown as required even after running both : `$ sudo apt-get update` and ` $ sudo apt-get upgrade`:
```
sudo apt-get dist-upgrade
sudo apt-get update
sudo apt-get upgrade
```
### 12. Clone the catalog app 
-   Create a new folder under the /www directory: `$ cd /var/www` then `$ sudo mkdir catalog`
- change the owner to grader: `$ sudo chown -R grader:grader catalog`
- Now go into catalog folder `$ cd catalog`
- Git clone the project : `git clone https://github.com/yanyanwu0358/catalog catalog`
- Now, the path to the catalog app should be:`/var/www/catalog/catalog`
### 13. Update the paths in the main python files:
- Rename the cloned application.py python file from its current name to __init__py
- In places.py file, update the path to the "client_secrets.json" file by adding the code below:
 `current_file_path = __file__    
  current_file_dir = os.path.dirname(__file__)   
  other_file_path = os.path.join(current_file_dir, "client_secrets.json")  `
### 14. Set up the database
- Install PostgreSQL : `$ sudo apt-get install postgresql`
- Login as user "postgres": `$ sudo su - postgres`
- Get into postgreSQL shell: `$ psql`
- Create a new database named " categoryitem" and create a new user named " catalog" in postgreSQL shell: postgres=# `CREATE DATABASE categoryitem;` postgres=# `CREATE USER catalog;`
- Set a password for user catalog: postgres=# `ALTER ROLE catalog WITH PASSWORD 'password';`
- Give user "catalog" permission to "categoryitem" application database: postgres=# `GRANT ALL PRIVILEGES ON DATABASE categoryitem TO catalog;`
- Quit postgreSQL: postgres=# `\q` or "Ctrl"+D
- Exit from user "postgres": exit
- Change the path to the database in the all of .py files and the database_setup.py files to : `engine = create_engine('postgresql://catalog:password@localhost/categoryitem')`
- Install psycopg2: `sudo apt-get -qqy install postgresql python3-psycopg2`
- Create catalog database by running the command: `cd /var/www/catalog/catalog` then run `python3 lotsofitems.py`
### 15. Create a new Virtual Host
- Create the catalog.conf file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste the text below inside the catalog.conf file:
```
<VirtualHost *:80>  
        ServerName 18.220.11.125  
        ServerAdmin [ha@mail.com](mail.com)  
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog>  
                Order allow,deny  
                Allow from all  
        </Directory>  
        Alias /static/var/www/catalog/catalog/static 
        <Directory /var/www/catalog/static/>  
                Order allow,deny  
                Allow from all  
        </Directory>    
       Alias /static/images /var/www/catalog/catalog/static/images  
        <Directory /var/www/catalog/catalog/static/images>  
                Order allow,deny  
                Allow from all  
        </Directory>  
      Alias /templates /var/www/catalog/catalog/templates  
        <Directory /var/www/catalog/catalog/templates>  
                Order allow,deny  
                Allow from all  
        </Directory>  
          ErrorLog ${APACHE_LOG_DIR}/error.log  
        LogLevel warn  
        CustomLog ${APACHE_LOG_DIR}/access.log combined  
 </VirtualHost>
```
- Disable the default virtual host: `$ sudo a2dissite 000-default.conf`
- Enable the new virtual host: `$ sudo a2ensite catalog.conf`
### 16.	Create the wsgi file for the app.
The wsgi file sits inside the parent catalog directory: `$ sudo nano /var/www/catalog/catalog.wsgi`
Paste the text below inside the catalog.wsgi file (in my case, inside my main python file __init__.py under catalog folder, the name of the application is application):
```
import sys    
import logging    
logging.basicConfig(stream=sys.stderr)   
sys.path.insert(0,"/var/www/catalog/")   `

from catalog import application as application     
application.secret_key = 'Add your secret key'    
```

### 17. Change the secret key credentials for Google sign in
- Get the Host Name for the public IP address (e.g., 18.220.11.125) from site: http://www.hcidata.info/host2ip.htm . In my case it is: ec2-18-220-11-125.us-east-2.compute.amazonaws.com
- Update the oauth2 credentials for the app in the Google Console at https://console.cloud.google.com/apis/credentials/oauthclient . Add http://18.220.11.125 and “http://ec2-18-220-11-125.us-east-2.compute.amazonaws.com”  to the field of “Authorised Javascript origins” and “Authorised redirect URIs” 
- Start Apache2 service with the command: $ sudo service apache2 restart
### 18. Change the secret key credentials for Facebook sign in
- Log into: https://developers.facebook.com/
- Add http://18.220.11.125 and “http://ec2-18-220-11-125.us-east-2.compute.amazonaws.com”  to the field of "App Domains"
### 19. Launch the app in the browser
-  Start Apache2 service with the command: `$ sudo service apache2 restart`
- Use the Host Name address http://ec2-18-220-11-125.us-east-2.compute.amazonaws.com (not just the public IP e.g., 18.220.11.125).
### 20. Get private key to be included in the final submission:
- At your vagrant file folder, after you vagrant up and vagrant ssh, go into the .ssh folder by `$cd .ssh`
- Then open and copy the file content to include it in your project submission. `$nano udacity_key.rsa`. This is the private key and looks like this:
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEA6dPH0yD0t/z2sk52SeOaCZXLcWngStHv/mGbts03r+eFPC4e
...........
...........
gpyuPc/KaursPRFkitxOYqM2gfV2WDv9Wln4S1RTdxux8SIKqPd/7w2JWPmsyYPV
-----END RSA PRIVATE KEY-----
```
## Useful tips for this project
- Discard local changes for a specific file by running `git checkout filename` before pull the updates from github `$ git pull origin master`
- If there were errors running the site, used the command below to see what it is:
`sudo tail -f /var/log/apache2/error.log`
- To import the other python file in the folder you need to include the parent folder name. Say your __init__.py and your python file both under parent folder catalog, inside __init__.py if you wanted to import places.py file, you would need to write:
`from catalog.places import app as application. `See details here: [http://mikegrouchy.com/blog/2012/05/be-pythonic-__init__py.html](http://mikegrouchy.com/blog/2012/05/be-pythonic-__init__py.html)
- For the image folder, we need to give all permissions so users can save images files to the folder.
`sudo chmod -R 777 /var/www/catalog/catalog/static/images`

- Helpful command in psql:
	-	`\c dbname;` (will connect to the db)
	-	`Delete from tablename;` (will delete all the rows in a table)
	-	`\d`  (describe)
	-	`dropdb dbname;` (will delete the database)

## List of third-party resources used to complete this project
 1. A recent submission on the same project. It's for python 2. Changes would required to make it work for pthon 3: https://github.com/hicham-alaoui/ha-linux-server-config 
 2. An older submission on the same project. Not everything works here : https://github.com/callforsky/udacity-linux-configuration 
 3. How to deploy a flask application on Lightsail: 
https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9
4. How to add path of a python file within another python file: https://blender.stackexchange.com/questions/31964/problem-with-paths-in-my-script-relative-to-local-python-file](https://blender.stackexchange.com/questions/31964/problem-with-paths-in-my-script-relative-to-local-python-file)
5. Use dist-upgrade when updates still show as required even after running  sudo apt-get update and sudo apt-get upgrade: https://askubuntu.com/questions/449032/29-packages-can-be-updated-how?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa

