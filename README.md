# 2420 Assignment 3 - Part 1

This is step-by-step process to set up a web server using Arch Linux.

Requirements:

1. Arch linux droplet
2. Text editor (nvim, vim, nano)


## Step 1

Run the user script
The `user` script will create all the necessary base directories and files for the new `webgen` user. 

**Please note that you have to have root privileges in order to run the user script**

Run the following command:

    sudo ./user 


Copy the content from the file to your `generate-index` file from this repository `generate-index` file.

## Step 2

### Generate Index service and timer

In Step 2 you have to create and configure `generate-index.service` and `generate-index.timer` 

1. `generate-index.service` will run your generate-index file.
 
`generate-index.service` needs to be created at the location below

    /etc/systemd/system/

Create the file by running the command below.

    sudo nvim /etc/systemd/system/generate-index.service

Change permissions on your newly created file.

    sudo chmod +x generate-index.service

You need to configure your `generate-index.service`

Copy the code below into your `generate-index.service` file.
      

      [Unit]
      Description=generate-index html
      Wants=network-online.target
      After=network-online.target

      [Service]
      User=webgen
      Group=webgen
      ExecStart=/var/lib/webgen/bin/generate-index
      RemainAfterExit=yes
      Restart=on-failure

      [Install]
      WantedBy=multi-user.target 

2. `generate-index.timer` will start your service every day at 05:00

`generate-index.timer` needs to be created at the location below

    /etc/systemd/system/

Create the file by running the command below.

    sudo nvim /etc/systemd/system/generate-index.timer

Change permissions on your newly created file.

    sudo chmod +x generate-index.timer


You need to configure your `generate-index.timer`

Copy the code below to you `generate-index.timer` file.

    [Unit]
    Description=Timer for generate-index.service

    [Timer]
    OnCalendar=*-*-* 05:00:00
    Persistent=True

    [Install]
    WantedBy=timers.target

Run the following commands to start and enable the services:

    sudo systemctl start generate-index.service
    sudo systemctl enable generate-index.service

    sudo systemctl start generate-index.timer
    sudo systemctl enable generate-index.timer

Check the status of your services by running the commands below:

    sudo systemctl status generate-index.timer
    sudo systemctl status generate-index.service


   
If everything went well, your output when running the status for `generate-index.service` should look similar to the image below

![Service Status](https://github.com/yovitsa/2420_Assignment_3_Part_1/blob/main/assets/services_image.png "Status running")



In the case that your services are not starting run the command below to reload the services. 
Also if you are making any changes to your service files. run the command below. 
      
    sudo systemctl daemon-reload

After running this command run the start / enable command services again.


## Step 3


### Install and configure  nginx

1. **Install nginx**

Run the command below to update your Arch linux distribution, you want to have most recent version.

    sudo pacman -Syu

Run the command below to install the nginx package

    sudo pacman --S nginx



2. **Modify the Main `nginx.conf` File**:

Update the `nginx.conf` file, so nginx server runs as the `webgen` user.

At the top of the `nginx.conf` file write the following statement 
     
     user webgen;
     

Save the file and exit. 
Do not make any other changes to the `nginx.conf`.

2. **Create a Separate Server Block File**:

We will create a separate server block file.

We need to create two new directories.

The new directories need to be created in the directory below.

    cd /etc/nginx

Create two new directories by running the command below.

    sudo mkdir -p /etc/nginx/sites-available
    sudo mkdir -p /etc/nginx/sites-enabled


Create a new server block file in `/etc/nginx/sites-available/`

Create a new the file called `webgen.conf`.

Change permissions on your newly created file.

    sudo chmod +x `webgen.conf`

Copy the code below
     
   
    server {
    
          listen 80;
          listen [::]:80;

          server_name <your droplet ip address>;

          root /var/lib/webgen/HTML
          index index.html;cat 

          location / {
            try_files $uri $uri/ =404;
    }

Save and exit the file.

Append include sites-enabled/*; to the end of the http block in the `nginx.conf` file:

/etc/nginx/nginx.conf
http {
    ...
    include /etc/nginx/sites-enabled/*;
}

Save the `nginx.conf` file and exit, do not change anything else.

}
     
To enable a site, create a symlink by running the command below:

    ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/webgen.conf



4. **Troubleshoot nginx configuration**:

  Check the `nginx.conf` file for any syntax errors:

Run the command below:
     
     sudo nginx -t
     
Run the command below to start and enable nginx services

    sudo systemctl daemon-reload
    sudo systemctl start nginx.service
    sudo systemctl enable nginx.service  

Test your web page by typing your ip address in your browser.
Your output should look similar to the below.

![ufw](https://github.com/yovitsa/2420_Assignment_3_Part_1/blob/main/assets/It%20Works.png)

### Step 4:

## Configure uncomplicated Firewall(ufw)

1. **Install and Configure UFW**:

Run the command below to update your Arch linux distirbution, you want to have most recent version.

    sudo pacman -Syu

Run the command below to install the nginx package

    sudo pacman -S ufw

Run the command below to enable and start the service.

    sudo systemctl enable --now ufw.service

Run the command below to check the status of ufw with the command

    sudo ufw status verbose

It should return inactive if you have just installed ufw for the first time.

You want to allow ssh connection to your server
Run any of the commands below to allow ssh from anywhere.

    sudo ufw allow ssh
    sudo ufw allow 22 # designated ssh port

You want to limit ssh connections. This will deny an incoming address if they attempt 6 initiations in 30 seconds.

    sudo ufw limit ssh

You need to allow `http` for you web server.
Run the command below.

    sudo ufw allow http

After defining the rules for your firewall, turn on your firewall by running the command below

**Do not run this command before applying rules to your firewall, especially for ssh**

    sudo ufw enable

Check the status again, to confirm that everything is working.

    sudo ufw status verbose

Your output should look something similar to the below.

![ports](https://github.com/yovitsa/2420_Assignment_3_Part_1/blob/main/assets/ports.png)

### Step 5

Test your web server by entering your ip addres into your browser

Your output should be similar to the image below

![It works](https://github.com/yovitsa/2420_Assignment_3_Part_1/blob/main/assets/It%20Works.png)

## Troubleshoot and possible issues


Issue 1: Apache Conflicts with nginx
When setting up nginx, I ran into a problem where Apache server was running on my system was using port 80. 
This caused a conflict because nginx also needed port 80.
Solution to this situation is to stop all apache servers and disabled them so that port 80 is free for nginx.

To figure out which processes were using port 80, I installed the `lsof` tool. 
This tool showed me all the running services and their port usage.

Run the command below to stop and after disable the apache server or some other server

    sudo systemctl stop httpd

    sudo systemctl disable httpd



Run the command below to check that apache is no longer using the port 80.

    lsof -i :80


Issue 2: iptables  Error
While trying to set up UFW (Uncomplicated Firewall), I got this warning:
iptables v1.8.10 (legacy): can't initialize iptables table 'filter': Table does not exist (do you need to insmod?).

The problem was caused by an outdated kernel or missing modules needed for iptables. I also found that the iptables.service was inactive. To fix this, I updated my Arch Linux server and rebooted it. After the update and reboot, iptables was working, and I was able to configure UFW without any issues

To update the arch server run the command below.

    sudo pacman -Syu

To reboot you server run the command below

    sudo systemctl reboot



# Reference
  
  
  1. `nginx` https://wiki.archlinux.org/title/Nginx 3.2.3.1 

  2. `lsof`
  https://www.liquidweb.com/blog/how-to-locate-open-ports-in-linux/#:~:text=The%20six%20primary%20methods%20for,re%20correctly%20configured%20and%20secure.

  3. `ufw`
  https://wiki.archlinux.org/title/Uncomplicated_Firewall

  4. `timer`
  https://wiki.archlinux.org/title/Systemd/Timers

