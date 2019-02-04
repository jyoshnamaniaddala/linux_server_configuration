# linux_server_configuration

- The web application is my [Item Catalog project](https://github.com/jyoshnamaniaddala/catalog) created earlier in this Nanodegree program.
- The virtual private server is [Amazon EC2](https://console.aws.amazon.com/).
- The database server is [PostgreSQL](https://www.postgresql.org/).

## Connecting to server
- Open [Amazon EC2](https://console.aws.amazon.com/) and login.
- change location of server to N.Virginia.
- Launch Instance , Choose an Amazon Machine Image (AMI) **Ubuntu Server 18.04 LTS (HVM), SSD Volume Type** ,Choose an Instance      Type **t2.micro**
- Then click on review and launch and again launch.
- Now create new key-pair, download a .pem file.
 click on **Inbound**.
- In Inbound, click on edit and add 3 rules i.e,add port SSh 2200, Http port 80, NTP port 123 and save. 

**To use privatekey** [link]()
 #### Public IP Address : 18.232.99.30
 #### Accessable Port : 2200
 
## Configure Linux Server
 #### step 1: 
 
     ```ssh -i filename(linux_server_31_01_2019).pem ubuntu @ IP address```
  
 #### Step 2: Update and upgrade installed packages

      ```
      sudo apt-get update
      sudo apt-get upgrade
      ```
 #### step 3: Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo vi /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Save and exit using esc and confirm with :wq.
#### step 4: Restart ssh
      `sudo service ssh restart`
#### step 5:
To check port 2200 wether working or not by 
      ```
     ssh -i linux_server_31_31_2019.pem -p 2200 ubuntu@IP address
      ```
 #### step 6:Run following commands
     ```
     sudo ufw default deny incoming
     sudo ufw default allow outgoing
     sudo ufw default allow 2200/TCP
     sudo ufw default allow www
     sudo ufw default allow 123/NTP
     sudo ufw default deny 22
     sudo ufw default enable
     To check status:
     sudo ufw status
                      ```
 
### Creating Grader User
#### Step 1. Login to ubuntu and add user 
       
       ``` sudo adduser grader and password: grader```

#### Step 2. Now give the grader premissions 
              ```sudo visudo```
         Now add this line ```grader ALL= (ALL:ALL) ALL``` and save and exit(ctl+X)
         
#### Step 3.To verify the grader as sudo permission 
      
      ```su -grader and password: grader ```

#### Step 4. Now, give SSH key-pair for grader.
       Create .ssh folder by 
           ```mkdir /home/grader/.ssh```
     
#### Step 5. Change it to grader 
       
       ```su grader password:grader```.
           
#### step 6. Copy authorised key
      
      ``` sudo -cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys```
           
#### Step 6.  change ownership 
        
        ``` chown grader.grader /home/grader/.ssh```.

#### Step 7. adding grader to sudogroup.
    ```
             sudo su
             usermod -aG sudo grader
    ```
#### Step 8. Change permissions tp .ssh folder.
    ```
      chmod 700 /home/grader/.ssh
      chmod 644 /home/grader/.ssh/authorized_keys
      vi /etc/ssh/sshd_config
      There at authentication edit as permit root login no and pubkey authentication yessave and exit(ecs+:wq)
    ```
#### Step 9. Now restart the service
       
       ``` sudo service ssh restart```.

#### Step 10. After restart open the server with grader 
             
             ```ssh -i filename.pem -p 2200 grader@IPaddress```.
           
### Configure the local timezone to UTC
To,set time zone for grader the command as follows
     ```
     sudo dpkg-reconfigure tzdata
     ```
### Process of Installing Apache and postgresql software

#### Step 1. Now istall apache software at grader
```
          sudo apt-get install apache2
```
#### Step 2. Now again install library functions of apache 
         ```
           sudo apt-get install libapache2-mod-wsgi-py3
         ```
   Now, enable wsgi 
           ```sudo a2enmod wsgi```
#### Step 3. To check that apache has installed successfully or not, Open web browser and type your IP Address.

#### Step 4. After enable of wsgi install some libraries of python development 
        
        ```sudo apt-get install libpq-dev python-dev```.

#### Step 5. Now, install postgresql 
         
         ``` sudo apt-get install postgresql postgresql-contrib```.

#### Step 6. After installation of postgresql, change to postgresql from grader.
    ```
    sudo su - postgres
    psql
    ```
#### Step 7. Now create a user, 
         
         ```create user catlog with password 'catlog';```.

#### Step 8. Now alter the user, 
         
         ```alter user catlog createdb;```.

#### Step 9. Create a database, 
         
         ```create database catlog with owner catlog ;```.

#### Step 10.  Now change to catlog database 
           ```\c catlog```.

#### Step 11. Now revoke all the schemas 

     ```revoke all on schema public from public;```.

#### Step 12. Now grant all the public schemas to catlog 
     
     ``` grant all on schema public to catlog;```.

#### Step 13. Now exit from the database.


## Installing git

#### Step 1.  install git 
         
         ```sydo apt-get install git```.

#### Step 2.  change the directory to www 
               ```cd /var/www```.

#### Step 3. Now clone our git project here 
          
          ```sudo git clone url_link(https://github.com/jyoshnamaniaddala/catalog.git)```.

#### Step 4. Now change the owner permissions 
         
         ```sudo chown -R grader:grader catalog```.

#### Step 5. Now change the name of main file name 
        
        ```sudo mv project.py __init__.py```.

#### Step 6. Now in python files change the database sqllite engine to postgres engine.
```
    engine = create_engine('postgresql://catlog:catlog@localhost/catlog')
```

### google OAuth credentials:

1) Go to your app's page in the [Google APIs Console](https://console.developers.google.com/apis)
2) Choose Credentials from the menu on the left.
3) Create an OAuth Client ID.
4) This will require you to configure the consent screen, with the same choices as in the video.
5) choose Web application  in the list of application types, .
6) You can then set the authorized JavaScript origins, http://IPAddress.xip.io/login,http://IPAddress.xip.io/callback,http://IPAddress.xip.io/gconnect.
7) You will then be able to get the client ID and client secret.

* update this client id in loginpage,html file
* Also, update client_secrets.json file

### Configure and enable new virtual host
```
<VirtualHost *:80>
    ServerName 18.232.99.30.xip.io
    ServerAlias ec2-18-232-99-30.compute-1.amazonaws.com
    ServerAdmin ubuntu@54.210.140.47
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python3.6/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Now enable virtual host ```sudo a2ensite catalog```
### Setup the flask application
*  create a file at /var/www/catalog/ with name **catalog.wsgi**.
```
import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  from catalog import app as application
  application.secret_key = 'supersecretkey'
```

* Now reload and restart apache.
```
      sudo service apache2 reload
      sudo service apache2 restart
```

* From var/www/catalog/catalog create a virtual environment.
```
sudo virtualenv -p python venv3
```

* Now change the ownership permissions ```sudo chown -R grader:grader venv3```.

* Now activate virtual environment ```. venv3/bin/activate```.

### Installations in virtual environment
```
    sudo apt-get install pip3
    pip3 install flask
    pip3 install sqlalchemy
    pip3 install httplib2
    pip3 install requests
    pip3 install --upgrade oauth2client
    pip3 install psycopg2-binary
    sudo apt-get install libpq-dev
```

* Now enable site 
      ```sudo a2ensite catalog``` 
          

* Deactivate the virtual environment: 
           `deactivate`.

*Disable the default Apache site

        `sudo a2dissite 000-default.conf`. 
     
  The following prompt will be returned:

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

* Reload Apache: 
      
      `sudo service apache2 reload`.

* Security Updates and package updates 
```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get dist-upgrade
```

#### useful commands

* To check errors in our file,

       ``` sudo tail -f /var/log/apache2/error.log```
  
  
## Visit Application
 open [web-browser](http://18.232.99.30.xip.io) or as (http://ec2-18-232-99-30.compute-1.amazonaws.com)

