#P5: Linux Server Configuration

### IP Address
52.37.15.134


### SSH Port
2200


### Complete URL
[Link to the Completed Website](http://ec2-52-37-15-134.us-west-2.compute.amazonaws.com/)

Also here for copy and paste:
http://ec2-52-37-15-134.us-west-2.compute.amazonaws.com/


### Configuration Summary
OK, so I'm going to take this step by step, highlighting both the way I got everything to work, in addition to any errors or issues I got stuck on. There were quite a few.

Also, first off, I want to point out the following were excellent resource lists, which I used extensively to find documentation and search for solutions to common problems I had throughout the project:

1. [Allan's Markedly underwhelming and potentially wrong resource list for P5](https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587)  
Allan's list was a great resource for getting through the basic setup steps.  
2. [Kirk Brunson's Project 5 Resources](https://discussions.udacity.com/t/project-5-resources/28343)  
I think I used most of Kirk's links throughout the project. Most of my sources are from his list.  
3. [Aaron_M's P5 How I got through it](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342)  
A lot of great links to the Digital Ocean Community tutorials.  
4. [Norbert Stuken's FSND-P5_Linux-Server-Configuration](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)  
Norbert does mention that this is basically a walkthrough, and it is very easy to go through his steps piece by piece to do the project. However, what I found most helpful was organizing the thinking for how to approach the project, and reading each of the sources listed to understand what's happening when you're making each call/modifying files

Also, before we dive into this, hats off to Digital Ocean for some fantastic Community tutorials. I think most of the links in the Resource Lists, and some other ones I used, came from them. Also, thanks a bunch to Michael for the Configuring Linux Web Servers class. That was tremendously helpful to get up to speed. If I don't mention a source for a particular section, it's most likely because it was covered in his class. Alright, here we go:


#### Basic Configuration
###### Launch Virtual Machine
I set up the Virtual Machine, followed steps 1-5 in the Udacity Development environment guide, no brainer. For the next few steps, I did everything as root, since I hadn't set up the user "grader" yet. Everything that follows, up until the creation of the user "grader" was done as "root"

While working on the project, I realized that I had to use the `ssh` command a lot, and I remember setting up some sort of file to make it easier to do on a previous unrelated project, so all you had to type was  
```
# I'm going to use "$" to designate shell commands
$ ssh udacity
```  
instead of the full
```
$ ssh -i ~/.ssh/udacity_key.rsa root@52.37.15.134
```
command. So I found this great article at Nerderati called [Simplify your life with an ssh config file](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/), set up a config file at `~/.ssh/config`, and added the following to it
```bash
Host udacity  
   Hostname 52.37.15.134  
   Port 2200  
   User root  
   IdentityFile ~/.ssh/udacity_key.rsa
```
This was super helpful. Especially for quickly adding a bash environment for debugging (explained later)

---
###### Update currently installed packages
Thanks to Michael, and the Configuring Linux Web Servers class, this was simply the following:
```
$ apt-get update      # To update ubuntu's package lists.
$ apt-get upgrade     # To actually upgrade the packages.
$ apt-get autoremove  # To remove any unnecessary packages.
```

---
###### Configure local timezone to UTC
This was done automatically when the virtual machine was launched. I don't know why, but I double checked this after reading [Ubuntu's Time Management Docs](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29). In essence:
```
$ cat /etc/timezone # To show the current timezone setting
```

---
###### Add User "grader"
The Udacity course went over this very well. So did this Digital Ocean tutorial: [How to Add and Delete Users on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps). First, since we're still logged in as root:
```
# Create the user grader and use "grader" as the password,
# skipping the remaining boxes for additional information

$ adduser grader

# also typing "y" and hitting enter to finish.
```

---
###### Grant "grader" sudo permissions
Again, Michael did a great job of covering this in the class. I also want to note that a lot of the resource lists refer to [this Digital Ocean article on adding and deleting users](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps), which details how to use `visudo` for accomplishing this, in addition to methods for assigned users to the group `%sudo`, but I used Michael's method of creating a file in `/etc/sudoers.d/` folder instead.  
```
# Create 'grader' file in /etc/sudoers.d/ directory

$ cd /etc/sudoers.d/
$ touch grader
$ nano grader
```
And adding the following line
```
grader ALL=(ALL) NOPASSWD:ALL
```

#### Securing the Server
OK, so I had about two hours of fun on this step before figuring out the solution below. I'll explain my mistakes as I go.

###### Generating Keygen pair for 'grader'
This is about the time I thought it would be smart to start using the grader user instead of root for everything. Kirk posted two great articles in his resource list, [ArchLinux: SSH Keys](https://wiki.archlinux.org/index.php/SSH_keys) and [Digital Ocean: How to set up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2). In addition, Michael and Udacity. So, I set up a local keygen pair saved in the `~/.ssh/` directory as `./udacityGrader`.:
```
# Create the keygen pair
$ ssh-keygen

#which created the ./udacityGrader and ./udacityGrader.pub keys
```
I'm pretty sure I had an issue with passwords in the first attempt, so I created the keys without a password. I had difficulty moving these using the methods explained in the articles, so I just added them directly to grader's home directory as the root user:
```
# Find grader's home directory
$ cat /etc/passwd
grader:x:1000:1000:Udacity Grader,,,:/home/grader:/bin/bash

# Create ~/.ssh
$ cd /home/grader
$ mkdir .ssh
$ cd .ssh
$ nano authorized_keys
```
where I copied and pasted the `udacityGrader.pub` public key from my local machine. Then, I modified the access permissions of  `authorized_keys` using `chmod`:
```
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```
I attempted to ssh in to the virtual machine as the grader user a different shell environment... But it didn't work. This took a long time to figure out. What I realized, was that when I tried to log in as grader, the server was trying to open the authorized_keys file to get the public key. However! since I had created this file as 'root', and modified the access permissions so that only the owner could read them (root), I had effectively locked 'grader' out of the server. I went back in as root and modified the ownership like so:
```
# Change ownership
$ chown grader /home/grader/.ssh /home/grader/.ssh/authorized_keys

# Change group for good measure
$ chgrp grader /home/grader/.ssh /home/grader/.ssh/authorized_keys
```
Logging in as grader was a success!! So I changed the SSH config file on my local machine (as described above), logged out of my 'root' shell session, and did the rest of this as the user 'grader'

---
###### SSHD Config File
Moving on, I locked out the root user and disabled password authentication to the server. Opening the `/etc/ssh/sshd_config` file:
```
# Changing PermitRootLogin from without-password to:
PermitRootLogin no

# PubkeyAuthentication was already configured as yes
PubkeyAuthentication yes

# Disable password authentication
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```
followed by restarting the ssh service:
```
$ sudo service ssh restart
```
I double checked this by trying to ssh as 'root' and ssh as 'grader' without a key. Didn't work. Success!

I think I also need to mention here that I ran into trouble initially because I added `Authusers grader` to the end of the file. I found it in one of my sources. However, I hadn't configured the file permissions correctly for ~/.ssh/authorized_keys (as explained above), then I logged out of the 'root' shell session. Basically I locked myself out. SO, I had to delete the virtual machine and start over. I caught my mistake this time though.

---
###### Ports
OK, opened up the `/etc/ssh/sshd_config` file:
```
# What ports, IPs and protocols we listen for
# Port 22, comment this out, just in case
Port 2200
```
followed by restarting the ssh service:
```
$ sudo service ssh restart
```
done.

---
###### UFW
Issued the following commands:
```
# Base configuration
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing

# Configure access for SSH, HTTP, and NTP
$ sudo ufw allow 2222/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp

# DANGERZONE!! Enable UFW
$ sudo ufw enable

# Check UFW
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)

```
Good to go!

#### Installing the Application




### Third Party Resources
####Resource Lists
[Udacity: Allan's Markedly underwhelming and potentially wrong resource list for P5](https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587)  
[Udacity: Kirk Brunson's Project 5 Resources](https://discussions.udacity.com/t/project-5-resources/28343)  
[Udacity: Aaron_M's P5 How I got through it](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342)  
[Github: Norbert Stuken's FSND-P5_Linux-Server-Configuration](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)  

####SSH
[Nerderati: Simplify your life with an ssh config file](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)   
[ArchLinux: SSH Keys](https://wiki.archlinux.org/index.php/SSH_keys)  
[Digital Ocean: How to set up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

####Timezone
[Ubuntu: Time Management Docs](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

####Add User/sudo permissions
[Digital Ocean: How to Add and Delete Users on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

####Markdown
[Github: Adam Pritchard's Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)  

