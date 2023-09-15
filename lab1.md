# Lab 1: Explore IDE

In this lab you will explore the coder environment as well as set up the directory structure for the rest of the labs. 

## The Coder Project

The Arista lab environment runs what's known as an IDE (Integrated Development Environment) called code-server. It's based on Microsoft's Visual Studio Code (VSCode) which is currently one of the most often used IDEs, especially with network automation. The code-server project takes VSCode and runs it as a web application server. Both VSCode and code-server are open source and free. 

## Exploring code-server

In your lab environment, click on Programmability IDE link.

<img src=lab1-images/1.png width="50%" height="50%" border=1>

Use the password that starts with "arista" followed by four alphanumeric characters for the password prompt:

<img src=lab1-images/2.png width="50%" height="50%" border=1>



It will ask for a password. It's the same password used for CVP, and can be found in the lower right hand corner of your inital lab page. 

<img src=lab1-images/3.png border=1>

This will bring up code-server. In this environment, you can create files and directories, sync up with GitHub/GitLab repositories, make your own repos and so forth. 

You may see some pop-ups asking to trust authors, activate themes, etc. Select yes/activate on them. 


<img src=lab1-images/4.png width="50%" height="50%" border=1>

You can X the windows for "Get Started" and "Welcome to GitLens" (and any others) and dismiss any notifications. 

When you're done you'll have a screen that looks like this: 

<img src=lab1-images/5.png width="75%" height="75%" border=1>

In the upper left you'll see the "hamburger" icon. It's called the hamburger icon as it resembles a hamburger with its three layers. Under that icon, select "Terminal" and then "New Terminal". 

<img src=lab1-images/6.png width="75%" height="75%" border=1>

This will open up a terminal section at the bottom. This is where most of the work will be done. You can open up multiple terminals with the "+" icon in the upper right, as well as resize the terminal to take up the entire screen or a small portion of the screen. Feel free to play around with the interface. 

<img src=lab1-images/7.png width="75%" height="75%" border=1>

In this window, confirm that Ansible is installed with the command <b><tt>ansible --version</tt></b>

<pre>
➜  project <b>ansible --version</b>
ansible [core 2.12.8]
  config file = /home/coder/.ansible.cfg
  configured module search path = ['/home/coder/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/dist-packages/ansible
  ansible collection location = /home/coder/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True

</pre>

Use the Linux <b><tt>cd</tt></b> command to change directories into the labfiles directory which is already there. The labfiles directory will persist (as the name implies) between reboots of the lab environment. Most files and subdirectories outside of either labfiles (or persist) will be erased upon reboot. 

<pre>
➜  project <b>cd labfiles</b> 
➜  persist 
</pre>

To simplify creation of the various files, there is a Github repository that you will clone. Use the command <tt><b>git clone https://github.com/sdn-pros/AVD_Labs.git</b></tt>

<pre>
labfiles <b>git clone https://github.com/sdn-pros/AVD_Labs.git</b>
Cloning into 'AVD_Labs'...
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 8 (delta 0), reused 8 (delta 0), pack-reused 0
Receiving objects: 100% (8/8), done.
➜  labfiles 
</pre>

This will clone the git repository onto your lab environment. In the upper left corner click on the "labfiles" directory which will open up the directory to show you the new AVD_Labs directory. 

<img src=lab1-images/8.png width="75%" height="75%" border=1>


You should see three directories: Ansible_EOS, Ansible_CVP, AVD_L3LS, AVD_L3LS_Final, AVD_L3LS_MultiDC and perhaps a few others. These directories contain files for this (and other) labs. 

## Change Password

Open the file <tt>inventory.yml</tt> in the AVD_L3LS directory. 

<img src=lab1-images/9.png width="75%" height="75%" border=1>

In this file change the two values of ansible_passwword to the password from your passwords link. 

<img src=lab1-images/10.png width="75%" height="75%" border=1>


