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
instead of the full command
```
$ ssh -i ~/.ssh/udacity_key.rsa root@52.37.15.134
```
So I found this great article at Nerderati called [Simplify your life with an ssh config file](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/), set up a config file at `~/.ssh/config`, and added the following to it
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
Again, Michael did a great job of covering this in the class. I also want to note that a lot of the resource lists refer to [this Digital Ocean article on adding and deleting users](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps), which details how to use `visudo` for accomplishing this, in addition to methods for assigned users to the group `%sudo`, but I used Michael's method of creating a file in the `/etc/sudoers.d/` directory instead.  
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

#### Final Configuration
The next few sections were covered by the Udacity course, but there were some great docs referred to by Allan and Kirk. I also took this in steps, setting up a couple test apps/files to make sure everything was working before cloning the project to the server. I'll talk about that as I go.
###### Apache
Install Apache:
```
$ sudo apt-get install apache2
```
Confirm that it's working by going to http://52.37.15.134. It works!

---
###### Apache mod_wsgi
Install mod_wsgi:
```
$ sudo apt-get install libapache2-mod-wsgi
```

I edited the `/etc/apache2/sites-enabled/000-default.conf` file by adding `WSGIScriptAlias / /var/www/html/myapp.wsgi` before the closing `</VirtualHost>` line:
```
# Slightly redacted to save space
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost>
```
and restart Apache:
```
$ sudo apache2ctl restart
```

Of course, now that apache is looking for a file at `/var/www/html/myapp.wsgi`, I set one up:
```
def application(environ, start_response):
   status = '200 OK'
   output = 'Hello World!'

   response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
   start_response(status, response_headers)

   return [output]
```
and checked it by going to my site link. It worked! At this point, I know I have apache working, and I know I have mod_wsgi working with a sample app.

---
###### Flask app
At this point, I basically just used an article from [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) to configure and test a sample Flask application with the current infrastructure. It worked, but for the sake of the size of this README, I'll just go over the flask dependencies. This was pretty simple. I basically ran all the commands in the `pg_config.sh` for P3. I jumped the gun a little bit, because I hadn't installed or configured the postgresql database, but the functionality was there:
```
$ sudo apt-get -qqy update
$ sudo apt-get -qqy install python-flask python-sqlalchemy
$ sudo apt-get -qqy install python-pip
$ sudo pip install bleach
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
```

---
###### The `su -` command
Before I get to postgresql, I'd like to digress. I found this article on [the su Command](http://www.linfo.org/su.html) to be incredibly useful when having to move between the user 'grader' and the user 'postgres' during the database configuration.
```
$ sudo su - grader
# Do stuff as grader
$ exit
logout
```
I especially thought that the use of the dash '-' character in the command, whereby the new session of the user being switched to is opened with that user's shell and environment variables, was totally cool. It was also great to find out that you could simply `exit` or `logout` or `ctrl-d` to get back to your original shell session. Anyway, this was very useful for configuring the Postgresql database, as explained below.

---
###### POSTGRESQL - install
Alright, I'm going to just refer you down to the Third Party Resources section below to check out the articles I used for this one. I basically got super confused about postgresql roles vs users for a long time, before it clicked. And reading a bunch of different articles, even the docs, helped to explain everything. So, here goes

Install Postgresql:
```
$ sudo apt-get install postgresql

# Digital Ocean recommends also installing the contrib package with some additional utilities
$ sudo apt-get install postgresql-contrib

# Also install psycopg2 while we are at it, so we can use postgresql with Flask
$ sudo apt-get install pyscopg2
```

---
###### POSTGRESQL - Do Not Allow Remote Connections
This [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) article was the basis for this part. Rather than copy and paste, let me just say that we can check the `/etc/postgresql/9.1/main/pg_hba.conf` file to see the postgresql network configuration. By default postgresql doesn't allow remote connections, which is awesome for this project. 

---
###### POSTGRESQL - configure
At this point, we switch over to the user 'postgres' to set configure the postresql database:
```
$ sudo su - postgres
```
This should have opened a "new" shell environment. You can check this quickly by looking at the command line itself, it should say 'postgres' where it was 'grader'. If it does, excellent!

Next we create a new postgresql user called 'catalog'. As the user 'postgres' at the normal command line, type:
```
$ createuser --interactive

# Which prompts this message:
Enter name of role to add: catalog
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
```
So basically, the prompt then sets up a postgres user called catalog with the attribute `Create DB` only. They have the ability to login, but they cannot create new roles or do any admin work. We can double check this by the following:
```
# Log into postgresql as postgres
$ psql

# Logged in

postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 catalog   | Create DB                                      | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}
 
```
Finally, while still inside the postgresql command prompt, we add a password for the postgresql user 'catalog' as follows:
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```
where 'password' stands for any password (simply 'catalog' in this case)

Lastly, we create the 'catalog' database:
```
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```
Revoke all rights from public to edit the database, and grant those rights to 'catalog', exit, and exit from the 'postgres' user:
```
postgres=# REVOKE ALL ON SCHEMA public FROM public;
postgres=# GRANT ALL ON SCHEMA public TO catalog;
# Quit postgresql
postgres=# \q

# Back to shell

$ exit
```

---
###### POSTGRESQL - create linux user 'catalog'
Great! So now that we have configured PostgreSQL, we also actually have to create a linux user called 'catalog'. This [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04) does a great job of explaining why. In short, Postgresql assumes that the current linux user will also have a matching role and database.

Anyway, so we do the same commands as we did for 'grader', but for 'catalog':
```
# Create the user catalog and use "catalog" as the password,
# skipping the remaining boxes for additional information

$ adduser catalog
```
Confirm that the user 'catalog' exists, and we're all done with Postgresql!! For now...

#### Installing the Application
Alright, so at this point we've configured the server, installed and verified that Apache is working, installed and verified that Postgresql is working. The next step is to actually download the Application!

---
###### IMPORTANT NOTE ON DESIGN DECISIONS - VIRTUALENV - VAGRANT
By this point, a couple of the tutorials and some of the forum posts recommended installing virtualenv for the P5. However, by the time I got to those recommendations, I had already installed a bunch of the dependencies globally on the server. Also, I don't have much experience with virtualenv. So for the sake of simplicity, I bypassed all those steps. In addition, P3 as it was, came with the vagrant "wrapper" if you will. So I cleaned up the directory structure of P3 by removing the files related to vagrant, and created a new github repository that would be used for this project with the code from P3.

---
###### Git
Install and set global configurations:
```
$ sudo apt-get install git
$ git config --global user.name "me"
$ git config --global user.email "me@gmail.com"
```

---
###### Clone Github Repository with P3
So, I was a bit anxious here, so I cloned the repository into the home directory (`~/`) for 'grader':
```
$ git clone https://github.com/kellenproctor/Catalog-for-P5.git
```
I then moved everything to the directory `/var/www/catalog/catalog/` which I had created earlier while configuring the test flask application.
```
$ sudo mv * /var/www/catalog/catalog/
```
Unfortunately, I decided to delete the remaining files, by `rm -rf *`, which included the `.git` directory, so that wasn't smart. Regardless, everything works perfectly anyway (end result), so I'm taking this as a warning for the future.

---
###### .htaccess and inaccessible .git
I created a `.htaccess` file in `/var/www/catalog/` and put in the following:
```
$ sudo nano .htaccess

# Inside nano
RedirectMatch 404 /\.git
```
Saved and exited.

---
###### Configuring Postgresql with App
Alright! So we have the app on the server, we have everything configured. The last step here, is to change every reference in `application.py`, `database_setup.py`, and `lotsofitems.py` from:
```
engine = create_engine('sqlite:///catalog.db')
```
to
```
engine = create_engine("postgresql://catalog:catalog@localhost/catalog")
```
Now the app can connect to the Postgresql database!!

---
###### Initialize Database and Data
Alright, now to actually create the tables, and initialize the database with some data!

First, move to the `/catalog/` directory holding the app files:
```
$ mv /var/www/catalog/catalog/

# Initialize the database
$ python database_setup.py

# Add some data
$ python lotsofitems.py
```
No errors. Score!!

---
###### Rename application.py
Simply, rename `application.py` to `__init__.py`
```
$ sudo mv application.py __init__.py
```

---
###### Reconfigure Apache Files
I changed `/var/www/catalog.wsgi` to this:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
application.debug = True
```

And created a `catalog.conf` file in the directory `/etc/apache2/sites-available/` with this:
```
<VirtualHost *:80>
      ServerName 52.37.15.134

      ServerAdmin admin@52.37.15.134

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
      LogLevel debug
      CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Configured and reset apache with the following commands:
```
# Make all existing sites-enabled disabled
$ sudo a2dissite [sites-enabled.conf file]

# Enable our catalog site
$ sudo a2ensite /etc/apache2/sites-available/catalog.conf

# Reset apache
$ sudo service apache2 restart
```
And we're off the races!!! Next step, say a little prayer, yell "I am awaited in Valhalla!" tell my coworker to "Witness Me!!" then check out the link...

---
###### Check out the link
And it works!!! move around a bit, and everything is excellent!! Game On, Wayne! Witness! (I just watched Mad Max again)

#### Exceeds Expectations
###### Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
I used [Fail2ban](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04) on the recommendation of Norbert's resource list.
Install Fail2ban:
```
$ sudo apt-get install fail2ban
```
Copy the default config file:
```
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
Check and change the default parameters by opening the local config file. Norbert does recommend changing a couple parameters, but after reading the documentation, I opted instead to just make sure the ssh port sections were changed correctly from 22 to 2200.:
```
$ sudo vim /etc/fail2ban/jail.local
```
Finally, stop and restart the fail2ban service to apply changes:
```
$ sudo service fail2ban stop
$ sudo service fail2ban start
```

---
###### Include cron scripts to automatically manage package updates
Great link from Norbert to the Ubuntu Documentation for the [unattended-upgrades](https://help.ubuntu.com/community/AutomaticSecurityUpdates) package.

Install and enable the unattended-upgrades package:
```
$ sudo apt-get install unattended-upgrades
$ sudo dpkg-reconfigure -plow unattended-upgrades
```

---
###### Includes monitoring applications that provide automated feedback on application availability status and/or system security alerts.
A lot of people recommended using [Glaces](http://glances.readthedocs.org/en/latest/glances-doc.html#introduction), so I did.

Install and run glances:
```
$ sudo pip install Glances
$ Glances
```

####Debugging
###### The `tail -f` command
Allan and Kirk both mention using some variation of the `tail -20` command to view the log files in `/var/log/apache2/error.log`. This basically echo the last 20 lines of that log file to the shell. However, I recall a better method of doing this in a different project I worked on. So I googled it, and sure enough, there was a great explanation on Superuser (a part of StackExchange) for the [tail -f command](http://superuser.com/questions/229627/linux-command-line-utility-for-watching-log-files-live). Bascially, by running
```
$ sudo tail -f /var/log/apache2/error.log
```
You get a 'live' view of the log file. So I could set up a shell session to run this while I tried to debug my app. Basically, the `-f` option enable monitoring, which sets up a process to update the shell if there are any changes made to the log file.


### Minor Issues
###### Hostname issue
So, at some point, and I'm not sure when it started, there was this really annoying thing where every time I ran `sudo`, it would generate this message
```
sudo: unable to resolve host ip-10-20-11-238
```
Luckily Norbert ran into the same issue, and post an article from [Askubuntu](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none) about this. Basically, I had to add a line to `/etc/hosts/`:
```
127.0.0.1 localhost

# Addition here
127.0.1.1 ip-10-20-11-238
```
And that did the trick!

---
###### Issue with showItems function
Alright, so I spent a day fixing this problem. I was super giddy about the site working in the first place, then my Dad tried to open up the description of a specific item... and it crashed. I couldn't figure out what the problem was. I eventually discovered, by looking at the logs and some judicious googling (that's really fun to say out loud) that my original code was throwing a `Keyerror: user_id blah blah`, so I modified my `__init__.py` file from this:
```python
# Show the specifics of an item
@app.route('/catalog/<category_name>/<item_name>/')
def showItem(category_name, item_name):
    item = session.query(Items).filter_by(name=item_name).one()
    categories = session.query(Category).all()
    creator = getUserInfo(item.user_id)
    user = getUserInfo(login_session['user_id'])
    if 'username' not in login_session or creator.id != login_session['user_id']:
        return render_template('publicitemdescription.html',
                                item=item,
                                category=category_name,
                                categories=categories,
                                user=user)
    else:
        return render_template('itemdescription.html',
                                item=item,
                                category=category_name,
                                categories=categories,
                                creator=creator,
                                user=user)
```
to this:
```python
# Show the specifics of an item
@app.route('/catalog/<category_name>/<item_name>/')
def showItem(category_name, item_name):
    item = session.query(Items).filter_by(name=item_name).one()
    categories = session.query(Category).all()
    if item:
        creator = getUserInfo(item.user_id)
    if 'username' not in login_session:
        return render_template('publicitemdescription.html',
                                item=item,
                                category=category_name,
                                categories=categories)
    if creator.id != login_session['user_id']:
        user = getUserInfo(login_session['user_id'])
        return render_template('publicitemdescription.html',
                                item=item,
                                category=category_name,
                                categories=categories,
                                user=user)
    else:
        return render_template('itemdescription.html',
                                item=item,
                                category=category_name,
                                categories=categories,
                                creator=creator,
                                user=user)
```
I think I stared at it for another few hours, kept on refreshing the page. It wasn't until I reset apache that it actually worked. Weird. I still don't know why that's the case.

---
###### Oauth with Google and Facebook
Lastly, I added the domain, http://ec2-52-37-15-134.us-west-2.compute.amazonaws.com, to both the Google and Facebook developer consoles, and I enabled my app in Facebook. This solved my Oauth issues.


#And that's it!
If you made it this far, thank you very much, and I'm super sorry that this is so long, but I thought it the best way to explain how I approached this project. Honestly, I think P5 should have been after P3, since its the next logical step after P3 (create the app, then deploy it). Otherwise, this has been excellent, and I look forward to doing some more classes/nanodegrees in the future!


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

####Apache
[Apache: Configuration Files, and Apache Docs in general](https://httpd.apache.org/docs/2.2/configuring.html)  
[Ubuntu: HTTPD - Apache2 Web Server](https://help.ubuntu.com/lts/serverguide/httpd.html)

####Flask
[Digital Ocean: How to deploy a flask application on an ubuntu vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

####Postgresql
[PostgreSQL: Documentation](http://www.postgresql.org/docs/9.3/interactive/tutorial.html)  
[Digital Ocean: How to install and use PostgreSQL on Ubuntu vps](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)  
[Digital Ocean: How to secure postgresql on Ubuntu vps](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)  
[Trackets Blog: PostgreSQL Basics by Example](http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html)  
[Kill The Yak: Use PostgreSQL with Flask or Django](http://killtheyak.com/use-postgresql-with-django-flask/)

####Fail2Ban
[Digital Ocean: How to protect SSH with Fail2Ban on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

####Unattended-Upgrades
[Ubuntu Docs: Automatic Security Upgrades](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

####Glances
[Glaces docs](http://glances.readthedocs.org/en/latest/glances-doc.html#introduction)

####Tail -f command
[Superuser (stackexchange): Linux command line utility for watching log files live?](http://superuser.com/questions/229627/linux-command-line-utility-for-watching-log-files-live)

####SU command
[The su Command](http://www.linfo.org/su.html)

####Unable to Resolve Host x.x.x.x
[Askubuntu: Error message when I run sudo: unable...](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)

####Markdown: for this amazing README!
[Github: Adam Pritchard's Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)  

