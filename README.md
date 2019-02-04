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
           
