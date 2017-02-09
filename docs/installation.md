The following instructions will guide an adminisistrator through the setup process of an BIBBOX.

There are two possible installation types:

* Building and installing a new BIBBOX automatically
* Installing a pre-built BIBBOX virtual machine

Notice: It is recommended to build a new BIBBOX using the automated installation scripts.


# Requirements

Requirements

* Download and install Git -> <https://git-scm.com/>
* Download and install Vagrant -> <https://www.vagrantup.com>
* Download and install VirtualBox -> <https://www.virtualbox.org>


The automatic BIBBOX setup is dependend on multiple repositories and one ZIP file download link:

* Liferay ZIP file: <http://downloads.bibbox.org/liferay-ce-portal-tomcat-7.0-ga3.zip>
* BIBBOX Repository 'sys-bibbox-vmscripts': <https://github.com/bibbox/sys-bibbox-vmscripts>
* BIBBOX Repository 'application-store': <https://github.com/bibbox/application-store>
* BIBBOX Repository 'sys-bibbox-frontend': <https://github.com/bibbox/sys-bibbox-frontend>
* BIBBOX Repository 'sys-bibbox-backend': <https://github.com/bibbox/sys-bibbox-backend>
* BIBBOX Repository 'sys-activities': <https://github.com/bibbox/sys-activities>
* BIBBOX Repository 'sys-idmapping': <https://github.com/bibbox/sys-idmapping>


# Configuration

The following parameters can be adjusted in `environments\production\manisfests\config.pp`:

| Parameter     | Description                                                                                                          | Default           |
|---------------|----------------------------------------------------------------------------------------------------------------------|-------------------|
| bibboxkit     | Name of the BIBBOX kit, currently only eB3kit is available.                                  						   | eB3Kit            |
| bibboxbaseurl | Base url of the kit which identifies the virtual machine. Needs to match 'bibboxbaseurl' parameter in 'Vagrantfile'. | eb3kit.bibbox.org |
| serveradmin   | Mail address of the administrator.                                                                                   | admin@bibbox.org  |
| db_user       | User of the Liferay database.                                                                                        | liferay           |
| db_password   | Password of the Liferay database.                                                                                    | bibbox4ever	   |
| db_name       | Name of the Liferay database.                                                                                        | lportal           |


# Quick installation guide

1. Download or clone the Git repository, **git clone <https://github.com/bibbox/kit-eb3kit.git> your-vm-name**
2. Open up a terminal and navigate to the repository, **cd your-vm-name**
3. Edit the vagrant configuration as descibed above, **nano Vagrantfile**
4. Edit the puppet configuration as descibed above, **nano  environments/production/manifests/config.pp**
5. Execute command in your base directory,  **vagrant up**
6. Wait while your BIBBOX is being installed (can take quite long, depending on internet connection)
7. Configure your proxy or DNS to access the BIBBOX VM at **http://bibboxbaseurl** or use **<http://192.168.10.1:1080>** (replace 1080 by the port you configured in the Vagrantfile)
8. You can login with the users *admin*, *pi*, *curator* and *operator*.  For all users the default password is *graz2017*. 
9. The liferay portal can be configured as user *bibboxadmin* 



# Detailed Instructions

## 1.) Setting up the requirements

In order for the automatic BIBBOX setup to work, you need to have Git, Vagrant and VirtualBox installed on your machine.
You can download and those tools from their official websites:

<https://git-scm.com/downloads>

![alt text](images/installation/git-download.jpg "Git")


<https://www.vagrantup.com/downloads.html>

![alt text](images/installation/vagrant-downloads.jpg "Vagrant")


<https://www.virtualbox.org/>

![alt text](images/installation/virtualbox-download.jpg "VirtualBox")


If you only have SSH access to the machine you want to install BIBBOX on, you can also download and install Vagrant and VirtualBox from there.

Run these commands for downloading and installing Vagrant and Virtualbox on Debian or Ubuntu:

`$ sudo apt-get update`

`$ sudo apt-get install git`

`$ sudo apt-get install vagrant`

`$ sudo apt-get install virtualbox`



## 2.) Cloning the installer scripts

Next, you need to choose an directory to clone the installation scripts to.
If you are installing from SSH you can create an directory with:

`mkdir /path/to/directory-name`

You can then navigate to the new directory with:

`cd /path/to/directory-name`

From there you can clone the repository <https://github.com/bibbox/kit-eb3kit.git> either with the GitHub client <https://desktop.github.com/> or via SSH terminal command:

`git clone https://github.com/bibbox/kit-eb3kit.git "your-kit-name"`

Once the repository has finished cloning, navigate into it:

`cd "your-kit-name"`


## 3.) BIBBOX installer configuration

You can change the default configuration of your BIBBOX installation in the following files:

**environments/production/manifests/config.pp**

**Vagrantfile**

To open up the files and changing their contents via SSH, try running the following commands on Debian or Ubuntu:

`sudo nano environments/production/manifests/config.pp`

`sudo nano Vagrantfile`

You can save your changes with "Control + O" and then "Enter" keyboard combinations.

Please find more information for configuration options in the **Configuration** chapter.



## 4.) Running the installer

You can now run the automated installation process with the following command:

`vagrant up`

BIBBOX will now automatically download all dependencies, set up a virtual machine, copy and symlink its contents and boot up the system.
Please note, that this can take quite a while depending on internet connection speed.

Just leave the terminal open and do some other work while the installation is running.
You will recognize it has finished, when you see a message saying "Applied catalog in xyz seconds" and you can write commands again in the terminal.

![alt text](images/installation/installation-finished.png "Finished installation")


## 5.) Proxy , DNS configuration

After the automated installation has finished, BIBBOX will automatically start its internal configuration process and boot up the BIBBOX portal.

Please note, that this process will take several minutes!

After everything is finished, you can access your freshly installed BIBBOX in your browser at http://**192.168.10.1**:18080 (replace the fat numbers with the IP address of your host machine).
If you have access to your hosting providers administration panel you can also rent a domain name like "bibbox.org" and point it to that address.

![alt text](images/installation/bibbox.jpg "Welcome to BIBBOX")



## 6.) Login and administration

You can now log into your BIBBOX with one of the five default users and the password **graz2017**:

* bibboxadmin
* admin
* pi
* curator
* operator

If you want to make changes to the default configuration of the portal (e.g. change the title or logo), you need to log in as **bibboxadmin**.


## 7.) Enjoy

That's all, enjoy your BIBBOX!



# Reference Information

This chapter will explain in more detail the technical processes during the installation of a BIBBOX.

## 1.) Vagrant, Puppet manifest


### Configuration lookup

After running `vagrant up` from terminal, Vagrant will look up the Vagrantfile configuration


### Name configuration

This defines a variable `bibboxbaseurl` to store the name of our virtual machine. Default is `eb3kit.bibbox.org`.

```
# Base url the bibbox kit (should be same as in puppet config file)
bibboxbaseurl = "eb3kit.bibbox.org"

vb.name = bibboxbaseurl
```


### Downloading and installation of the Virtual Box image

In the Vagrantfile, the Virtual Box image to set up the virtual machine is defined.
In this case it's **puppetlabs/ubuntu-14.04-64-puppet**, an Ubuntu Trusty image found in Hashicorp's Atlas platform, which already includes Puppet 4.3.

`config.vm.box = "puppetlabs/ubuntu-14.04-64-puppet"`


### IP / Port configuration

After downloading and installing the image, Vagrant will now configure which IP and ports should be mapped to which part of the virtual machine.

```
#Webserver
config.vm.network :forwarded_port, host: 1080,  guest: 80

#Tomcat - Liferay
config.vm.network :forwarded_port, host: 18080, guest: 8080
config.vm.network :forwarded_port, host: 18000, guest: 8000
config.vm.network :forwarded_port, host: 18081, guest: 8081

config.vm.network :private_network, ip: "192.168.10.10"

config.vm.network :forwarded_port, guest: 22, host: 2230, id: "ssh", disabled: true
config.vm.network :forwarded_port, guest: 22, host: 2231, auto_correct: true
```


### CPU / Memory definition

Next, Vagrant will define the amount of memory (RAM) and CPU cores of the host machine, that will be accessible by the virtual machine.

```
# (Option 1) Customize the amount of memory and the cpu cores of the VM:

vb.memory = "10240"	# 10GB of RAM
vb.cpus = 4
```

```
# (Option 2) Give VM 1/4 system memory & access to all cpu cores on the host

host = RbConfig::CONFIG['host_os']

if host =~ /darwin/
	vb.cpus = `sysctl -n hw.ncpu`.to_i
	# sysctl returns Bytes and we need to convert to MB
	vb.memory = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
elsif host =~ /linux/
	vb.cpus = `nproc`.to_i
	# meminfo shows KB and we need to convert to MB
	vb.memory = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
else	# Fallback for Windows
	vb.cpus = 4
	vb.memory = 10240
end
```


### Storage definition

In this step, an additional storage disk will be added to the virtual machine. The default size for it is 301GB.

```
# Create new disk of size 301GB
file_to_disk = File.realpath( "." ).to_s + "/disk-300GB.vdi"

if ARGV[0] == "up" && ! File.exist?(file_to_disk) 
   puts "Creating 300GB disk #{file_to_disk}."
   vb.customize [
        'createhd', 
        '--filename', file_to_disk, 
        '--format', 'VDI', 
        '--size', 301 * 1024 # 301 GB
        ] 
   vb.customize [
        'storageattach', bibboxbaseurl, 
        '--storagectl', 'IDE Controller', 
        '--port', 1, '--device', 0, 
        '--type', 'hdd', '--medium', 
        file_to_disk
        ]
end
```


### Provisioning

Now that the virtual machine is set up and booted, our provisioning will execute multiple commands within the virtual machine.
We use this opportunity to set the storage disks in Raid-0 mode, download Liferay and install some Puppet modules aas well as Python3 pip and iNotify tools.

```
# Provision the VM with several puppet modules
config.vm.provision "shell", inline: <<-SHELL
sudo bash /vagrant/resources/add_disk.sh
mkdir -p /etc/puppetlabs/code/modules
cp -r /vagrant/modules/* /etc/puppetlabs/code/modules
if [ ! -f "/opt/liferay-ce-portal-tomcat-7.0-ga3.zip" ]; then
  echo "Downloading Liferay, this might take a while...";
  wget -nc -nv -P /opt/ "http://downloads.bibbox.org/liferay-ce-portal-tomcat-7.0-ga3.zip";
else
  echo "Liferay sources already exist. Skipping download.";
fi
puppet module install puppetlabs-stdlib --version 4.14.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-apt --version 2.3.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-ntp --version 6.0.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-firewall --version 1.8.1 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-apache --version 1.11.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppet-archive --version 1.2.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-vcsrepo --version 1.5.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppet-alternatives --version 1.0.2 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-docker_platform --version 2.1.0 --modulepath /etc/puppetlabs/code/modules
puppet module install puppetlabs-postgresql --version 4.8.0 --modulepath /etc/puppetlabs/code/modules
puppet module install tylerwalts-jdk_oracle --version 1.5.0 --modulepath /etc/puppetlabs/code/modules
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 7F438280EF8D349F
sudo apt-get update
sudo apt-get install -y inotify-tools
sudo apt-get install -y python3-pip
SHELL
```

Additionally we tell Puppet and the virtual machine the new vm name as defined in `bibboxbaseurl`.
Also, the default environment for Puppet will be set, so Puppet knows where to find its initialization manifests.

```
config.vm.provision "puppet" do |puppet|
	puppet.facter = {
	  "fqdn" => bibboxbaseurl
	}

	puppet.environment = "production"
	puppet.environment_path = "environments"
end
```


### Puppet configuration

Vagrant has now set up the virtual machine, defined its resources and provisioned our Puppet tools. Next, it will automatically apply our default Puppet manifest `environments/production/manifests/config.pp`.
This is a config layer that let's us configure our BIBBOX settings in a clean file.

```
#################################
##       Configurations        ##
#################################

class { 'vmbuilder':

	# General Kit Information
	bibboxkit		=> "eB3Kit",
	bibboxbaseurl	=> "eb3kit.bibbox.org",
	serveradmin		=> "admin@bibbox.org",

	# Database Information
	db_user			=> "liferay",
	db_password		=> "CHANGEulHbbFpulHbM74JuBk9@CwMS",
	db_name			=> "lportal"

}
```


### Puppet initialization

The Puppet configuration file automatically calls the vmbuilder class found in `modules/vmbuilder/manifests/init.pp` and hands over out configuration parameters.



## 2.) Installation scripts




## This chapter is still in development...


