# **2420- Assignment-3 part 1**
## Automating System Information with Bash and Nginx

## **Introduction**

The purpose of this assignment is to write a Bash script that will generate an index.html file and configure it to run daily at 5:00 using a systemd system and timer. The HTML document generated by this script will be Hosted on a Nginx web server running on your Arch Linux droplet, with a ufw firewall configured to enhance server security

## **Table of Contents**
1. **Task 1:** Setting up a system user with directories
2. **Task 2:** Creating and configuring systemd service and scripts
3. **Task 3:** Configuring Nginx
4. **Task 4:** Installing and configuring ufw for ssh and HTTP
5. **Task 5:** Verifying that the System Information page is working properly


## **Task One:** Setting up user and directories

> What is the benefit of creating a system user for this task?


The reason we want to create a system user instead of using a regular user or root is to enhance security and keep thefunctions isolated from regular user activities. It decreases the risk of accidental or malicious system wide chages.


**Step 1:** Creating a new user

Type the following command:

```
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```
**Explanation:**
- `-r`: This will create a system account
- `-d`: This specifies the home directory
- `-s /usr/sbin/nologin`:  This specifies the non-login shell
  

**Step 2:** Create home directory

Next, since we don't have a home directory yet, we want to create one using this command

```
sudo mkdir -p /var/lib/webgen
```


**Step 3:** Create subdirectory /bin

This command is used to create subdirectories within the webgen directory

```
sudo mkdir -p /var/lib/webgen/bin
```


**Step 4:** Create subdirectory /HTML

This command is used to create subdirectories within the webgen directory

```
sudo mkdir -p /var/lib/webgen/HTML
```


**Step 5:** Cloning the generated index script

Our next step is setting up the generated index script.
This can be done by cloning it from the sourcehut repository with this command:

```
git clone https://git.sr.ht/~nathan_climbs/2420-as2-start
```


**Step 6:** Moving the generated script into the correct directory

Since we cloned the generated script into the home directory, we want to move it into the appropriate /var/lib/webgen/bin/ directory using:

```
sudo mv 2420-as2-start/generate_index /var/lib/webgen/bin/
```


**Step 7:** Give executable permissions to script

```
sudo chmod +x /var/lib/webgen/bin/generate_index
```



**Step 8:** Set Ownership

```
sudo chown -R webgen:webgen /var/lib/webgen
```


## **Task Two:** Creating and configuring systemd service and scripts



**Step 1:** Create the generate-index.service file

The first step of this task is to use the command below to create the generate-index.service file 

```
sudo nvim /etc/systemd/system/generate-index.service
```

Once you are inside the nvim text editor, you will want to type the following content inside of it:

```
[Unit]
Description=Generate Index Service File

[Service]
Type=simple
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

**Explanation:**
- **[Unit]:** Describes the service 
- **[Service]:** Defines how the service should behave when started
  

**Step 2:** Create the generate-index.timer file

The next step is to use the command below to create the generate-index.timer file

```
sudo nvim /etc/systemd/system/generate-index.timer
```

Once you are inside the nvim text editor, you will want to add the following content inside of it:

```
[Unit]
Description=Timer to run script daily

[Timer]
OnCalendar=*-*-* 05:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```


**Explanation:**
- **[Timer]:** This specifies the schedule of the timer 
- **[Install]:** This Specifies how timer works with systemd


**Step 3:**  Start Timer

This command starts the generate-index.timer right away.

```
sudo systemctl start generate-index.timer
```


**Step 4:** Enable timer

This command configures the timer to start automatically when the system is booted.

```
sudo systemctl enable generate-index.timer
```


**Step 5:** Timer status

Once the timer has been set up and started, we want to confirm its status to ensure that everything is working correctly.

to verify that the timer is active and that the service runs successfully,

Run this command:

```
systemctl status generate-index.timer
```

The output should look like this:

![Screenshot] <img src = "./imgone.png">


> [!NOTE]
> run `systemctl status generate-index.service` to check the status of the service


**Step 6:** View logs

To view the logs of the generated-index.service, run this command:

```
sudo journalctl -u generate-index.service
```


## **Task Three:** Configuring Nginx


**Step 1:** Install Nginx

Our first step is to install Nginx using this command:

```
sudo pacman -Syu nginx
```


**Step 2:** Open the file

Our next goal is to configure the Nginx file.
First, we have to open it with this command:

```
sudo nvim /etc/nginx/nginx.conf
```

It should look like this:

![Screenshot] <img src = "./imgtwo.png">


**Step 3:** Configure user

Our next step is to locate `user` and to change it to:

```
user webgen;
```

> [!NOTE]
> In the screenshot above, user is at the top of the script.
> you will not only want to change it to webgen, but also delete the `#` sign


**Step 4:** Creating two new directories

next, we will want to create two separate server block files instead of modifying the main nginx.conf file. The reason we want to do this is so that it is easier to manage configurations, and if changes are made it wouldn't affect the main nginx.conf file. 


First, we will create the sites-available directory:

 ```
sudo mkdir -p /etc/nginx/sites-available
```


Next, we will create the sites-enabled directory:

```
sudo mkdir -p /etc/nginx/sites-enabled
```

**Step 5:** Create a new configuration file

use this command to create a new server block:

```
sudo nvim /etc/nginx/sites-available/webgen.conf
```


**Step 6:** Add content

once inside the new script, add the following contents:

```
server {
   listen 80;
   server_name localhost-webgen;

   root /var/lib/webgen/HTML;
   index index.html;

   location / {
      try_files $uri $uri/ =404;
   }
}
```

**Explanation:**

The server is set up to listen on port 80 and serve files from the /var/lib/webgen/HTML directory in response to requests to localhost-webgen by this Nginx server block. It ensures that a 404 Not Found error will be displayed for any wrong requests and that only existing files or directories are provided.

**Step 7:** Enable Server Block

Next, we want to create a symbolic link in ordeer to enable our new configuration.

This can be done by using this command:

```
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/
```

**Step 8:** Open nginx.conf file and add the new directory

First we need to open the file using: `sudo nvim /etc/nginx/nginx.conf`

After it is opened, locate the http section and add the following line:

```
include /etc/nginx/sites-enabled/*.conf;
```

it should look like this:

![Screenshot] <img src = "./imgthree.png">


**Step 9:** Check for errors

To confirm that the Nginx file has no errors, run this command:
```
sudo nginx -t
```


**Step 10:** Restart Nginx


use this command to restart Nginx to apply changes:

```
sudo systemctl restart nginx
```

**Step 11:** Start Nginx:

We want to restart Nginx using:

```
sudo systemctl start nginx
```


**Step 12:** Check status

To confirm Nginx is up and running, use:

```
sudo systemctl status nginx
```


## **Task Four:** Installing and configuring ufw for ssh and HTTP


**Step 1:** Install UFW

install ufw using this command:

```
sudo pacman -Syu ufw
```

> [!IMPORTANT]
> Do `NOT` Enable ufw immeditaley after installation


**Step 2:** Allow SSH and HTTP from anywhere

Use these two commands:

```
sudo ufw allow ssh
```

```
sudo ufw allow http
```


**Step 3:** Enable rate limiting

To enhance security and protect against unauthorized access attempts, enable SSH rate limiting to prevent brute-force attacks by using the following command:

```
sudo ufw limit ssh
```


**Step 4:** Enable UFW

enter the following command to enable UFW:

```
sudo ufw enable
```

**Step 5:** Check UFW status

to check the status of the UFW, and to check whether the rules are applied use this command:

```
sudo ufw status 
```

The output should look like this:

```
Status: active

To                         Action      From
--                         ------      ----
22                         LIMIT       Anywhere
80                         ALLOW       Anywhere
22 (v6)                    LIMIT       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
```



## **Task Five:** Verification


**Step 1:** Reload and Restart

Use these three commands to reload the system and start it up again:

```
sudo systemctl daemon-reload
```

```
sudo systemctl start nginx.service
```

```
sudo systemctl enable nginx.service
```


**Step 2:** Get IP address

Our next step is to log into our Digital Ocean account.

Go to the droplet section, and there you should be able to copy your droplets IP adress


**Step 3:** Visit the IP address

Once we have copied our IP address, we will want to open a new tab, and type `http://` along with our droplets IP address.

example:

```
http://146.190.36.50/
```


the output should look something like this:

![Screenshot] <img src = "./imgfive.png">



**Congratulations!!** You have successfully setup a Bash script that generates a static index.html file that contains some system information.The script will be configured to run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script willbe served with an nginx web server that will run on your Arch Linux droplet along with a firewall setup using ufw to help secure your server.



## **Refrences**

1. https://wiki.archlinux.org/title/Users_and_groups#Example_adding_a_system_user.

2. https://wiki.archlinux.org/title/Nginx

3. https://wiki.archlinux.org/title/Uncomplicated_Firewall 

4. https://gitlab.com/cit2420/2420-notes-f24/-/tree/main



