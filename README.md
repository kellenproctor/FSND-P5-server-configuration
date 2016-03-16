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

Also, before we dive into this, hats off to Digital Ocean for some fantastic Community tutorials. I think most of the links in the Resource Lists, and some other ones I used, came from them. Also, thanks a bunch to Michael for the Configuring Linux Web Servers class. That was tremendously helpful to get up to speed. Alright, here we go:

####Basic Configuration
1. Set up the Virtual Machine, followed steps 1-5 in the Udacity Development environment guide, no brainer.
2. While working on the project, I realized that I had to use the `ssh` command a lot, and I remember setting up some sort of file to make it easier to do, so all you had to type was `ssh udacity` instead of the full `ssh -i ~/.ssh/udacity_key.rsa root@52.37.15.134` command (please note that I did set up a grader user, and most of the work done past the initial stages were with the grader user and `sudo`, this is just for example). So I found this great article at Nerderati called [Simplify your life with an ssh config file](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/), set up a config file at `~/.ssh/config`, and added the following to it 
```bash
Host udacity  
   Hostname 52.37.15.134  
   Port 2200  
   User root  
   IdentityFile ~/.ssh/udacity_key.rsa
```  
This was super helpful  
3. 




### Third Party Resources
####Resource Lists
[Udacity: Allan's Markedly underwhelming and potentially wrong resource list for P5](https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587)  
[Udacity: Kirk Brunson's Project 5 Resources](https://discussions.udacity.com/t/project-5-resources/28343)  
[Udacity: Aaron_M's P5 How I got through it](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342)  
[Github: Norbert Stuken's FSND-P5_Linux-Server-Configuration](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)  

####SSH
[Nerderati: Simplify your life with an ssh config file](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)  

####Markdown
[Github: Adam Pritchard's Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)  

