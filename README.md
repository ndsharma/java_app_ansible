Author - Nitin Dutt Sharma
Document Version - 1.0
Date - 19/07/2017

Purpose
=========

This document provides relevant information about the software versions installed, the dependencies of files present in the tarball and how these file interact with each other.

Objective
----------
The objective of this development is to build a software package that will deploy a java app on 3 nodes. This app with then be served and load balanced by nginx server. This was needed to be achieved by a single vagrant up command.

Requirements
-------------
First requirement is that Vagrant Version: 1.9.1 and Virtual box Version 5.1.10 r112026 (Qt5.6.2) are installed on the host machine.

The version of Ansible used is 2.3.1.0 but we only need version 1.9 or above which is being managed by the deployment itself

Default version of Java is being used in this case it is java version "1.7.0_131"

If this is being run from a Windows Pc then another requirement is that bash shell must be installed on the PC. I used git installation to install the bash shell or the following link should help.
https://www.windowscentral.com/how-install-bash-shell-command-line-windows-10

Assumption
----------
This software application package was built and tested on a Windows 10 host machine with Vagrant and Virtual box Installed.

Vagrantfile
-----------
Vagrant file is the main file that defines how Vagrant VM's are configured and the number of the VMs to be initiated. It also defines which Vagrant base box is to be used which is "puppetlabs/ubuntu-14.04-64-nocm" in this case and the ip addresses to be allocated. It also tells the "vagrant up" command to run a bootstrap file.

bootstrap-appserver.sh
----------------------
The bootstrap file is a script that runs the additional commands when the Vagrant VMs are being setup. It first installs the dependencies for Ansible software and then add Ansible's repo, update the apt cache and lastly install the Ansible software. This is done in a specific sequence of commands so that we get the latest version of software. When the sequence of this commands was changed it affected the Ansible version that got installed.

Lastly the bootstrap file tells the Ansible playbook to be run.

java_site.yml
-------------
This is the main playbook file which is run when bootstrap-appserver script is called by the Vagrantfile while Vagrant VMs are being setup.

This playbook confirms which role plays are to be run in what sequence and on which servers. There are 3 roles which java_site site playbook refers to common, java and nginx. The common and java roles are being run or all appservers which nginx is only being run on appserver-1 which is also acting as a nginx web proxy and the load balancer.

To understand it easily the java_site playbook is a top level playbook which is acting as a wrapper that further calls and runs the role playbooks.

common role
-----------
This role contains the playbook(task/main.yml) that is targeted at all servers in the inventory. This is opening the port 80 in the firewall and then it deploys a configuration file in the /etc/sudoers.d folder to enable passwordless sudo access for the vagrant user and also password enabled sudo access for the admin group.

This 10_vagrant config file is saved in the file folder of the common role. In the end it notifies the (handler/main.yml) file saved in the handler folder to restart the firewall service.

java role
---------
This role contains the playbook(task/main.yml) that is targeted at all servers in the inventory. Even though this role is targeted at all server but it is kept seperate as this only deals with java software and java app related configuration.

This role will install Java JDK default version in this case it is java version "1.7.0_131". It will then create a the /tmp/java_app app directory. Copy the java app jar (spring-boot-sample.jar) file in the files directory for this role.

Next step in the role is the installation of a package called supervisor. This is required to monitor the java app is running even if the service is rebooted. This is done by copying the (templates/java_app.conf.j2) file to the /etc/supervisor/conf.d/java_app.conf folder.

Lastly it will notify the (handler/main.yml) to restart the supervisor service.

nginx role
----------
Nginx role will install the latest nginx package on the main server (appserver-1). After which it will deploy 2 templates conf file for Nginx.

(templates/default.j2) conf file gets copied to the /etc/nginx/sites-available/default folder and is used to configure the nginx reverse proxy and load balancing. Reverse proxy is needed because the sample application listens on port 8080 and we want server to listen on port 80.

(templates/nginx.conf.j2) is copied to /etc/nginx/ and configures the logging upstreamlog in the /var/log/nginx/access.log file which we can use to test the load balancing.


Deploying the app
------------------
Once vagrant and bash shell(only if using onn Windows PC) has been setup untar the files and it should create a directory called technical_test. Change directory to that directory.

Once you are in the technical_test directory run the command ./deploy_java.sh (linux based Vagrant host) or sh .\deploy_java_site.sh (Windows based Vagrant Host)

This will run the vagrant up command to deploy and configure the Vagrant VM's and then run the curl command to show that the app is being served by all 3 nodes.
