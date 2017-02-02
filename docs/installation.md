# Installation 

The following instructions will guide an adminisistrator through the setup process of an BIBBOX.

There are two possible installation types:

* Building and installing a new BIBBOX automatically
* Installing a pre-built BIBBOX virtual machine

Notice: It is recommended to build a new BIBBOX using the automated installation scripts.


## Building a new BIBBOX

### Requirements

* Download and install Git -> <https://git-scm.com/>
* Download and install Vagrant -> <https://www.vagrantup.com>
* Download and install VirtualBox -> <https://www.virtualbox.org>


### Dependencies

The automatic BIBBOX setup is dependend on multiple repositories and one ZIP file download link:

* Liferay ZIP file: <http://downloads.bibbox.org/liferay-ce-portal-tomcat-7.0-ga3.zip>
* BIBBOX Repository 'sys-bibbox-vmscripts': <https://github.com/bibbox/sys-bibbox-vmscripts>
* BIBBOX Repository 'application-store': <https://github.com/bibbox/application-store>
* BIBBOX Repository 'sys-bibbox-frontend': <https://github.com/bibbox/sys-bibbox-frontend>
* BIBBOX Repository 'sys-bibbox-backend': <https://github.com/bibbox/sys-bibbox-backend>
* BIBBOX Repository 'sys-activities': <https://github.com/bibbox/sys-activities>
* BIBBOX Repository 'sys-idmapping': <https://github.com/bibbox/sys-idmapping>


### Configuration

The following parameters can be adjusted in `environments\production\manisfests\config.pp`:

| Parameter     | Description                                                                                                          | Default           |
|---------------|----------------------------------------------------------------------------------------------------------------------|-------------------|
| bibboxkit     | Name of the BIBBOX kit, currently only eB3kit is available.                                  						   | eB3Kit            |
| bibboxbaseurl | Base url of the kit which identifies the virtual machine. Needs to match 'bibboxbaseurl' parameter in 'Vagrantfile'. | eb3kit.bibbox.org |
| serveradmin   | Mail address of the administrator.                                                                                   | admin@bibbox.org  |
| db_user       | User of the Liferay database.                                                                                        | liferay           |
| db_password   | Password of the Liferay database.                                                                                    | bibbox4ever	   |
| db_name       | Name of the Liferay database.                                                                                        | lportal           |


### Quick installation guide for expert users

1. Download or clone the Git repository, **git clone <https://github.com/bibbox/kit-eb3kit.git> your-vm-name**
2. Open up a terminal and navigate to the repository, **cd your-vm-name**
3. Edit the vagrant configuration as descibed above, **nano Vagrantfile**
4. Edit the puppet configuration as descibed above, **nano  environments/production/manifests/config.pp**
5. Execute command in your base directory,  **vagrant up**
6. Wait while your BIBBOX is being installed (can take quite long, depending on internet connection)
7. Configure your proxy or DNS to access the BIBBOX VM at **http://bibboxbaseurl** or use **<http://192.168.10.1:1080>** (replace 1080 by the port you configured in the Vagrantfile)
8. You can login with the users *admin*, *pi*, *curator* and *operator*.  For all users the default password is *graz2017*. 
9. The liferay portal can be configured as user *bibboxadmin* 



### Detailed beginners guide

#### 1.) Setting up the requirements

In order for the automatic BIBBOX setup to work, you need to have Git, Vagrant and VirtualBox installed on your machine.
You can download and those tools from their official websites:

<https://git-scm.com/downloads>

![alt text](images/git-download.jpg "Git")


<https://www.vagrantup.com/downloads.html>

![alt text](images/vagrant-downloads.jpg "Vagrant")


<https://www.virtualbox.org/>

![alt text](images/virtualbox-download.jpg "VirtualBox")


If you only have SSH access to the machine you want to install BIBBOX on, you can also download and install Vagrant and VirtualBox from there.

Run these commands for downloading and installing Vagrant and Virtualbox on Debian or Ubuntu:

`$ sudo apt-get update`

`$ sudo apt-get install git`

`$ sudo apt-get install vagrant`

`$ sudo apt-get install virtualbox`



#### 2.) Cloning the installer scripts

Next, you need to choose an directory to clone the installation scripts to.
If you are installing from SSH you can create an directory with:

`mkdir /path/to/directory-name`

You can then navigate to the new directory with:

`cd /path/to/directory-name`

From there you can clone the repository <https://github.com/bibbox/kit-eb3kit.git> either with the GitHub client <https://desktop.github.com/> or via SSH terminal command:

`git clone https://github.com/bibbox/kit-eb3kit.git "your-kit-name"`

Once the repository has finished cloning, navigate into it:

`cd "your-kit-name"`


#### 3.) BIBBOX installer configuration

You can change the default configuration of your BIBBOX installation in the following files:

**environments/production/manifests/config.pp**

**Vagrantfile**

To open up the files and changing their contents via SSH, try running the following commands on Debian or Ubuntu:

`sudo nano environments/production/manifests/config.pp`

`sudo nano Vagrantfile`

You can save your changes with "Control + O" and then "Enter" keyboard combinations.

Please find more information for configuration options in the **Configuration** chapter.



#### 4.) Running the installer

You can now run the automated installation process with the following command:

`vagrant up`

BIBBOX will now automatically download all dependencies, set up a virtual machine, copy and symlink its contents and boot up the system.
Please note, that this can take quite a while depending on internet connection speed.

Just leave the terminal open and do some other work while the installation is running.
You will recognize it has finished, when you see a message saying "Applied catalog in xyz seconds" and you can write commands again in the terminal.

![alt text](images/installation-finished.png "Finished installation")


#### 5.) Accessing the BIBBOX in the browser

After the automated installation has finished, BIBBOX will automatically start its internal configuration process and boot up the BIBBOX portal.

Please note, that this process will take several minutes!

After everything is finished, you can access your freshly installed BIBBOX in your browser at http://**192.168.10.1**:18080 (replace the fat numbers with the IP address of your host machine).
If you have access to your hosting providers administration panel you can also rent a domain name like "bibbox.org" and point it to that address.

![alt text](images/bibbox.jpg "Welcome to BIBBOX")



#### 6.) Login and administration

You can now log into your BIBBOX with one of the five default users and the password **graz2017**:

* bibboxadmin
* admin
* pi
* curator
* operator

If you want to make changes to the default configuration of the portal (e.g. change the title or logo), you need to log in as **bibboxadmin**.


#### 7.) Enjoy

That's all, enjoy your BIBBOX!



### In-depth guide and configuration



## Use a pre-built BIBBOX VM