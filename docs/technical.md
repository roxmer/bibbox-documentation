# About this guide

This guide serves as a technical documentation for BIBBOX developers. It is used for documenting the under-the-hood functionality, explain how everything works and how to debug common problems.

# Vagrant/Puppet Installer

### Configuration lookup

After starting the installation process by running `vagrant up` from terminal, Vagrant will look up the **Vagrantfile** configuration.


### VM Configuration

The upper part of the **Vagrantfile** is the configuration area, where you can set all the variables to control you virtual machine.

**Required parameters:**

* bibboxbaseurl
* ip
* http_port
* ssh_vagrant_port
* ssh_port

| Parameter        | Description                                                                                      | Default           |
|------------------|--------------------------------------------------------------------------------------------------|-------------------|
| vmname           | Name of your virtual machine.                                  	                              | eB3Kit            |
| bibboxbaseurl    | Base url of your BIBBOX installation. Needs to match 'bibboxbaseurl' parameter in 'environments\production\manisfests\config.pp'.                                                                       | eb3kit.bibbox.org |
| cpus             | Number of CPU cores assigned to the virtual machine.                                             | admin@bibbox.org  |
| memory           | Total amount of memory in MB (RAM) available to the virtual machine.                             | 8192              |
| disksize         | Amount of additional disk space in MB (hard drive).                                              | 301 * 1024 	      |
| diskname         | The disk file will be named "disk-**diskname**.vdi".                                             | 301GB             |
| ip               | The static IP to your BIBBOX within the host's network.                                          | 192.168.10.10     |
| http_port        | The port to your BIBBOX within the host's network.                                               | 1080              |
| ssh_vagrant_port | Port for SSH access used internally by Vagrant.                                                  | 2230              |
| ssh_port         | Port used for SSH access from outside the host machine.                                          | 2231              |

This is how it looks like in the file:

```
# Name of the virtual machine
# default: "eb3kit"
vmname = "eb3kit"

# Base url the bibbox kit (should match puppet config file)
# default: "eb3kit.bibbox.org"
bibboxbaseurl = "eb3kit.bibbox.org"

# Number of assigned CPU cores
# default: 4
cpus = 4

# Total amount of memory in MB (RAM)
# default: "8192" (8GB)
memory = "8192"

# Amount of additional disk space in MB (hard drive)
# default: 301 * 1024 (301GB)
disksize = 301 * 1024

# Name of the disk
# default "301GB"
diskname = "301GB"

# Static IP and port within the host's network
# ip default: "192.168.10.10"
# http_port default: 1080
ip = "192.168.10.10"
http_port = 1080
# https_port = XXXX

# Ports used for SSH connection
# ssh_vagrant_port is only for Vagranbt internally
# default: 2230
# ssh_port is used to connect from outside
# default: 2231
ssh_vagrant_port = 2230
ssh_port = 2231
```

### VM setup procedure

1. When running the **Vagrantfile** and after reading the configuration parameters, Vagrant will start downloading and installing the virtual machine image from the Hashicorp Atlas platform. For BIBBOX we use an UBUNTU 14.04 64Bit image from Puppetlabs. This image already comes with Puppet and Python (2 and 3) preinstalled.
    
        # OS image of the virtual machine
        config.vm.box = "puppetlabs/ubuntu-14.04-64-puppet"
        
2. Vagrant will then register the IP and port forwarding for thevirtual machine. This means that the IP and port that was set before will be the static address of the virtual machine within its host system.

        # Static IP and port within the host's network
        config.vm.network :private_network, ip: ip
        config.vm.network :forwarded_port, host: http_port,  guest: 80
        
3. Just like the the static public address before, another forwarding configuration makes the virtual machine available from outside the host machine via SSH. For this, the **ssh_vagrant_port** and **ssh_port** will be used.

        # Port forwarding for SSH
        config.vm.network :forwarded_port, guest: 22, host: ssh_vagrant_port, id: "ssh", disabled: true
        config.vm.network :forwarded_port, guest: 22, host: ssh_port, auto_correct: true
        
4. Now the virtual hardware configuration will be done in Vagrant's provider function. This is where the CPU cores, memory limit and disk size, as well as the virtual machine's name will be set.

        # Start of provisioning
        config.vm.provider "virtualbox" do |vb|

            # Enable multiple CPUs and set RAM and CPUs of VM
            vb.customize ["modifyvm", vmname, "--ioapic", "on"  ]
            vb.memory = memory
            vb.cpus = cpus

            # Create new disk of size 301GB
            file_to_disk = File.realpath( "." ).to_s + "/disk-" + diskname + ".vdi"

            if ARGV[0] == "up" && ! File.exist?(file_to_disk) 
               puts "Creating " + diskname + " disk #{file_to_disk}."
               vb.customize [
                    'createhd', 
                    '--filename', file_to_disk, 
                    '--format', 'VDI', 
                    '--size', disksize
                ] 
               vb.customize [
                    'storageattach', vmname, 
                    '--storagectl', 'IDE Controller', 
                    '--port', 1, '--device', 0, 
                    '--type', 'hdd', '--medium', 
                    file_to_disk
                ]
            end

            vb.name = vmname
        end
        
5. At this point, the virtual machine is already up and running with forwarded addresses, a set number of CPU cores, memory and a disk for storing data. The next step is called **Provisioning** and will actually prepare our environment within the virtual machine.

        # Provision the VM with several puppet modules, add additional disk space enable bash for ssh and download Liferay if it doesn't exist yet
        config.vm.provision "shell", inline: <<-SHELL
            . . .
            
6. During provisioning, we will execute several shelll commands for setting up the virtual machine to our needs, before we start the BIBBOX setup itself. For this purpose, we first enable **Bash** and connect our storage disk to the virtual machine in Raid-0 mode. Then we copy our logrotate script **liferay** to autostart, create a directory for our puppet modules and copy the **vmbuilder** module into it.

        sudo sed -ie 's#SHELL=/bin/sh#SHELL=/bin/bash#g' /etc/default/useradd
        sudo bash /vagrant/resources/add_disk.sh
        sudo cp /vagrant/resources/liferay /etc/logrotate.d/
        mkdir -p /etc/puppetlabs/code/modules
        cp -r /vagrant/modules/* /etc/puppetlabs/code/modules

    **liferay**
    
        /opt/liferay/tomcat-8.0.32/logs/catalina.out {
            copytruncate
            daily
            missingok
            rotate 7
            compress
            size 10M
        }
        
7. Next in the provisioning, we will download the source files of the Liferay portal as a handy ZIP file. The file should only be downloaded, if it doesn't yet exist on the virtual machine. It will be saved to **/opt/**. If the file is already there, because we ran the Vagrant provisioning more than once, only a message should be displayed.

        if [ ! -f "/opt/liferay-ce-portal-tomcat-7.0-ga3.zip" ]; then
            echo "Downloading Liferay, this might take a while...";
            wget -nc -nv -P /opt/ "http://downloads.bibbox.org/liferay-ce-portal-tomcat-7.0-ga3.zip";
        else
            echo "Liferay sources already exist. Skipping download.";
        fi

8. This next step is very important in specific for our BIBBOX, because this will download a lot of Puppet modules we need in order to build our BIBBOX. ALl these modules will be downloaded into our **modules** directory we created before. Please note, that all modules in this directory will be environment independent. This means, any modules in this folder can be used be development and production Puppet environments. For now we only use the production environment anyway.

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
        
9. The last step in our shell provisioner is the installation of some additional tools, namely **inotify-tools** and **python3-pip**. For this we first update the **apt-key** and **apt-get** and then install the tools. This is also the last step ouf our provisioning in which we run shell commands.

            sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 7F438280EF8D349F
            sudo apt-get update
            sudo apt-get install -y inotify-tools
            sudo apt-get install -y python3-pip
        SHELL

10. Now the time for another provisioning has come. This time we don't execute shell commands, but instead do some minor configuration for out Puppet tools. One handy tool that comes with Puppet 4 is **Facter** which is used to list facts about the virtual machine. In our provisioning we tell **Facter**, the domain name we want to run our BIBBOX on later. Then we define our default environment path and default environment. That's all for the provisioning of our machine.

        config.vm.provision "puppet" do |puppet|
            puppet.facter = {
                "fqdn" => bibboxbaseurl
            }

            puppet.environment = "production"
            puppet.environment_path = "environments"
        end

11. The last step of the VM configuration chapter and the **Vagrantfile** ist as follows. To prevent a warning of **Facter** we set the hostname of the virtual machine to the domain name set in **Facter** before. We also check if Vagrant has the **vagrant-cachier** plugin installed and if so, configure it to be enabled for this box.

        # To avoid the Warning: Could not retrieve fact fqdn
        config.vm.hostname = bibboxbaseurl

        if Vagrant.has_plugin?("vagrant-cachier")
            config.cache.scope = :box
        end
        
12. That's it for the virtual machine configuration. The machine is now online and Puppet will automatically execute the BIBBOX setup scripts. More about that in the next chapter.



### BIBBOX configuration

Vagrant has now set up the virtual machine, defined its resources and provisioned our Puppet tools. Next, it will automatically apply our default Puppet manifest `environments/production/manifests/config.pp`, which is a config layer that let's us define our BIBBOX settings a little more specific. 

**Required parameters:**

* bibboxbaseurl
* serveradmin

| Parameter     | Description                                                                                      | Default           |
|---------------|--------------------------------------------------------------------------------------------------|-------------------|
| bibboxkit     | Name of the BIBBOX kit, **currently only eB3kit is available!**                             	   | eB3Kit            |
| bibboxbaseurl | Base url of your BIBBOX installation. Needs to match 'bibboxbaseurl' parameter in 'Vagrantfile'. | eb3kit.bibbox.org |
| serveradmin   | Mail address of the administrator.                                                               | admin@bibbox.org  |
| db_user       | User of the Liferay database.                                                                    | liferay           |
| db_password   | Password of the Liferay database.                                                                | bibbox4ever	   |
| db_name       | Name of the Liferay database.                                                                    | lportal           |

This is how it looks like in the file:

```
#################################
##       Configurations        ##
#################################

class { 'vmbuilder':

	# General Kit Information
	# Currently only "eB3Kit" is available for bibboxkit
	bibboxkit		=> "eB3Kit",
	bibboxbaseurl	=> "eb3kit.bibbox.org",
	serveradmin		=> "admin@bibbox.org",

	# Database Information
	db_user			=> "liferay",
	db_password		=> "bibbox4ever",
	db_name			=> "lportal"

}
```


### BIBBOX setup procedure

1. The Puppet configuration file automatically calls the **vmbuilder** class found in `modules/vmbuilder/manifests/init.pp` and hands over our configuration parameters. If any üarameters are unset or missing, the **vmbuilder** class will use its default parameters.

        class vmbuilder(

            $bibboxkit      = "eB3Kit",
            $bibboxbaseurl  = "eb3kit.bibbox.org",
            $serveradmin    = "admin@bibbox.org",

            $db_user        = "liferay",
            $db_password    = "bibbox4ever",
            $db_name        = "lportal"

        ) {
            . . .
            
2. The first thing Puppet will do in this class, is to override the hosts file and the creation of our our user groups **bibbox**, **docker** and **liferay**.

        # Override hosts file
        file { '/etc/hosts':
            ensure 		=> 'file',
            source 		=> '/vagrant/resources/hosts',
            notify		=> Class['postgresql::server']
        }
        

        # Ensure groups 'bibbox' and 'docker'
        group { 'bibbox':
            ensure		=> 'present',
            gid		    => 501
        }
        group { 'docker':
            ensure		=> 'present',
            gid		    => 502
        }
        group { 'liferay':
            ensure		=> 'present',
            gid		    => 503
        }
        
3. Now that the groups exist, we can create the system users we need for BIBBOX, Docker and Liferay. We also add our new groups to the **root** user.

        # Ensure users 'bibbox' ans 'liferay'
        user { 'bibbox':
            ensure		=> 'present',
            gid		    => '501',
            groups		=> ['bibbox', 'docker'],
            uid		    => '501'
        }
        user { 'liferay':
            ensure		=> 'present',
            groups		=> ['bibbox', 'liferay', 'docker'],
            uid		    => '502'
        }
        user { 'root':
            ensure 		=> 'present',
            groups 		=> ['bibbox', 'liferay', 'docker']
        }
        user { 'vmadmin':
            ensure 		=> 'present',
            groups 		=> ['sudo', 'docker'],
            password 	=> '$6$mEf4vngN$erw.SnwvN1g77a/i2FdcGKveS7ktRvvauJHfH/XzHR9eMERn8p5sJzF7GTUoypvVU37u6IPaUeTvC9UZj8zOL1',
            managehome	=> true,
            home 		=> '/home/vmadmin',
            uid 		=> '507'
        }
        
4. Next we will use the Apache Puppet module to install an apache server and activate some proxy modules for it.

        # Install apache with default config
        class { 'apache':
            default_vhost     => false,
            purge_vhost_dir   => false
        }
        class { 'apache::mod::proxy': }
        class { 'apache::mod::proxy_http': }
        class { 'apache::mod::proxy_ajp': }
        class { 'apache::mod::proxy_wstunnel': }


        # Define default vhost, or apache can't start
        apache::vhost { $bibboxbaseurl:
            port    	=> '80',
            docroot 	=> '/var/www/vhost'
        }
        
5. Another Puppet module named ** JDK Oracle** will help us install Oracle Java in our virtual machine.

        # Install oracle java
        class { 'jdk_oracle':
            jce => true
        }
        
6. The Puppet **PostgreSQL** module will install and set up our database we use for Liferay portal.

        # Install and configure postgresql
        class { 'postgresql::server': }
        postgresql::server::role { $db_user:
            password_hash 	=> postgresql_password($db_user, $db_password),
        }
        postgresql::server::db { $db_name:
            user     	=> $db_user,
            password 	=> postgresql_password($db_user, $db_password)
        }
        postgresql::server::database_grant { $db_name:
            privilege 	=> 'ALL',
            db        	=> $db_name,
            role      	=> $db_user
        }
        
7. Now, we will use the Puppet Archive module to unzip our Liferay sources into the **/opt/** directory.

        # Unzip liferay sources to 'liferay' directory
        archive { '/opt/liferay-ce-portal-tomcat-7.0-ga3.zip':
            ensure 		    => 'present',
            extract       	=> true,
            extract_path  	=> '/opt',
            creates       	=> '/opt/liferay-ce-portal-7.0-ga3',
            cleanup       	=> true,
            notify		    => File['MoveLiferayContents']
        }
        
8. Thanks to the **notify** and **subscribe** listeners we can now trigger a chain of events in a specific order. When we unzipped the Liferay sources just now, **notify** told our next functions to execute as soon as the unzipping has finished. In these next events we copy the unzipped directory to **liferay**, copy our TomCat binary folder, set some permissions for the **deploy** and **bin** folders and delete the old liferay directory.

        # Copy unzipped liferay contents and delete source
        file { "MoveLiferayContents":
            ensure 		=> 'directory',
            path 		=> '/opt/liferay',
            source 		=> '/opt/liferay-ce-portal-7.0-ga3',
            recurse 	=> true,
                owner		=> 'liferay',
                group   	=> 'liferay',
                subscribe 	=> Archive['/opt/liferay-ce-portal-tomcat-7.0-ga3.zip']
        }
        file { '/opt/liferay/deploy':
            ensure 		=> 'directory',
            mode 		=> '0777',
            owner 		=> 'liferay',
            group 		=> 'liferay',
            subscribe	=> File['MoveLiferayContents']
        }
        file { '/opt/liferay/tomcat-8.0.32/bin':
            ensure 		=> 'present',
            source 		=> '/opt/liferay-ce-portal-7.0-ga3/tomcat-8.0.32/bin',
            recurse		=> true,
            mode 		=> '0777',
            owner 		=> 'liferay',
            group 		=> 'liferay',
            subscribe	=> File['MoveLiferayContents']
        }
        file { '/opt/liferay-ce-portal-7.0-ga3':
            ensure 		=> 'absent',
            purge 		=> true,
            recurse 	=> true,
            force 		=> true,
            subscribe	=> File['/opt/liferay/tomcat-8.0.32/bin']
        }
        
9. In the next step, we will copy two **.war** files to Liferay's deploy folder. One of them holds the theme of BIBBOX, the other contains a special portlet, which loads our ReactJS-GUI and passes it the url parameters as **params** object for navigation.

        # Copy 'war' files to liferay deploy folder
        file { "/opt/liferay/deploy/BIBBOXDocker-portlet-7.0.0.1.war":
            ensure 		=> 'file',
            owner		=> 'liferay',
            group   	=> 'liferay',
            mode 		=> '0777',
            source 		=> '/vagrant/resources/BIBBOXDocker-portlet-7.0.0.1.war',
            subscribe	=> File['MoveLiferayContents']
        }
        file { "/opt/liferay/deploy/bibbox-theme.war":
            ensure 		=> 'file',
            owner		=> 'liferay',
            group   	=> 'liferay',
            mode 		=> '0777',
            source 		=> '/vagrant/resources/bibbox-theme.war',
            subscribe	=> File['MoveLiferayContents']
        }
        
10. Now we will let Puppet create some directories where BIBBOX will store its application instances and a copy of the application store from GitHub, as well as a directory for the import/export tool and error page.

        # Create directories used by bibbox
        file { '/opt/bibbox':
            ensure 	=> 'directory',
            mode 	=> '0777'
        }
        file { '/opt/bibbox/application-instance':
            ensure	=> 'directory',
            owner	=> 'liferay',
            group   => 'bibbox'
        }
        file { '/opt/bibbox/application-store':
            ensure	=> 'directory',
            owner	=> 'liferay',
            group   => 'bibbox'
        }
        file { '/opt/bibbox/application-import-export':
            ensure	=> 'directory',
            owner	=> 'root',
            group   => 'bibbox'
        }
        file { '/var/www/html/error':
            ensure	=> 'directory',
            owner	=> 'root',
            group   => 'root',
            mode	=> '0644'
        }
        
11. In this step we change the permissions of our apache config directories, so BIBBOX doesn't have any problems in writing new configs for its applications.

        # Set access rights for apache directories
        File <| title == '/etc/apache2/sites-available' |> {
            owner	=> 'root',
            group   => 'bibbox',
            mode 	=> '0777'
        }
        File <| title == '/etc/apache2/sites-enabled' |> {
            owner	=> 'root',
            group   => 'bibbox',
            mode 	=> '0777'
        }
        
12. Next, we clone our setup scripts, application store, ReactJS GUI, activity service and ID mapping service from GitHub into the corresponding directories in **/opt/bibbox/**.

        # Clone bibbox repositories
        vcsrepo { '/opt/bibbox/sys-bibbox-vmscripts':
            ensure   	=> 'present',
            provider 	=> 'git',
            source   	=> 'https://github.com/bibbox/sys-bibbox-vmscripts.git',
            subscribe 	=> File['/opt/liferay/tomcat-8.0.32/bin'],
            notify		=> Exec['pythonRequirements']
        }
        vcsrepo { '/opt/bibbox/application-store/application-store':
            ensure   	=> 'present',
            provider	=> 'git',
            owner		=> 'liferay',
            group   	=> 'bibbox',
            source  	=> 'https://github.com/bibbox/application-store.git',
            notify		=> File['/var/www/html/bibbox-datastore/index.html']
        }
        vcsrepo { '/opt/bibbox/sys-bibbox-frontend':
            ensure  	=> 'present',
            provider 	=> 'git',
            source   	=> 'https://github.com/bibbox/sys-bibbox-frontend.git'
        }
        vcsrepo { '/opt/bibbox/sys-activities':
            ensure   	=> 'present',
            provider 	=> 'git',
            source   	=> 'https://github.com/bibbox/sys-activities.git'
        }
        vcsrepo { '/opt/bibbox/sys-idmapping':
            ensure   	=> 'present',
            provider 	=> 'git',
            source   	=> 'https://github.com/bibbox/sys-idmapping.git'
        }
        
13. Some of the cloned resources must reside in specific places like the autostart directory **/etc/init.d/** so they get executed when starting the virtual machine. There are also some configuration files and other script we have to move. This is what we do in this step.

        # Copy bibbox configuration and init scripts
        file { "/etc/init.d/bibbox":
            source 		=> '/opt/bibbox/sys-bibbox-vmscripts/initscripts/etc/init.d/bibbox',
            owner  		=> 'root',
            group  		=> 'root',
            mode   		=> '0777',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        file { "/etc/init.d/functions":
            source 		=> '/opt/bibbox/sys-bibbox-vmscripts/initscripts/etc/init.d/functions',
            owner  		=> 'root',
            group  		=> 'root',
            mode   		=> '0777',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        file { "/etc/init.d/liferay":
            source 		=> '/opt/bibbox/sys-bibbox-vmscripts/initscripts/etc/init.d/liferay',
            owner  		=> 'root',
            group  		=> 'root',
            mode   		=> '0777',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        file { "/opt/liferay/portal-ext.properties":
            owner  		=> 'liferay',
            group  		=> 'liferay',
            source 		=> '/opt/bibbox/sys-bibbox-vmscripts/initscripts/liferay/portal-ext.properties',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        file { "/opt/liferay/portal-setup-wizard.properties":
            ensure  	=> 'file',
            owner  		=> 'liferay',
            group  		=> 'liferay',
            content 	=> epp('/vagrant/resources/templates/portal-setup-wizard.properties.epp', {
                'db_user'   	  => $db_user,
                'db_password'	  => $db_password,
                'db_name'	      => $db_name
            }),
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        file { "/etc/bibbox":
            recurse 	=> true,
            owner  		=> 'root',
            group  		=> 'bibbox',
            source 		=> '/opt/bibbox/sys-bibbox-vmscripts/initscripts/etc/bibbox',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-vmscripts']
        }
        
14. In this simple step, all we do is creating a new config directory **/etc/bibbox/conf.d**.

        # Create 'conf.d' directory
        file { '/etc/bibbox/conf.d':
            ensure		=> 'directory',
            owner  		=> 'root',
            group  		=> 'bibbox',
        }

15. Now we take our some of our configuration parameters we passed to the **vmbuilder** class and use them to render one of our BIBBOX configuration files **/etc/bibbox/bibbox.cfg**.

        # Render 'bibbox.cfg' template file
        file { "/etc/bibbox/bibbox.cfg":
            ensure  	=> 'file',
            owner  		=> 'root',
            group  		=> 'bibbox',
            content 	=> epp('/vagrant/resources/templates/bibbox.cfg.epp', {
                'bibboxkit'	    => $bibboxkit,
                'bibboxbaseurl'	=> $bibboxbaseurl
            })
        }
        
16. The time has now come to boot up our Apache, Liferay and BIBBOX services and make sure, they are always running in case we restart the virtual machine.

        # Start bibbox and liferay services
        Service <| title == 'apache2' |> {
            ensure 		=> running,
            enable 		=> true,
            subscribe	=> File['/etc/init.d/bibbox']
        }
        service { 'bibbox':
            ensure 		=> running,
            enable 		=> true,
            subscribe	=> File['/etc/init.d/bibbox']
        }
        service { 'liferay':
            ensure 		=> running,
            enable 		=> true,
            subscribe	=> File['/etc/init.d/liferay']
        }
        
17. Now we create some directories and symbolic links for our BIBBOX datastore, the error page and the TomCat log.

        # Create and symlink datastore directories
        file { '/var/www/html/bibbox-datastore':
            ensure	=> 'directory'
        }
        file { '/var/www/html/bibbox-datastore/bibbox':
            ensure	=> 'link',
            target	=> '/opt/bibbox/application-store/'
        }
        file { '/var/www/html/bibbox-datastore/js':
            ensure	=> 'directory',
            owner		=> 'root',
            group 	=> 'bibbox',
            mode 		=> '0777'
        }
        file { '/var/www/html/bibbox-datastore/index.html':
            ensure		=> 'file',
            source		=> '/vagrant/resources/index.html'
        }
        file { '/var/www/html/error/index.html':
            ensure		=> 'file',
            source		=> '/vagrant/resources/error/index.html'
        }
        file { '/var/www/html/bibbox-datastore/log.out':
            ensure		=> 'link',
            target		=> '/opt/liferay/tomcat-8.0.32/logs/catalina.out'
        }
        
18. In this step, we copy our ReactJS GUI sources to the Apache datastore, so we can use them by including an URL in our Liferay portlets.

        # Copy gui resources to datastore
        file { '/var/www/html/bibbox-datastore/js/js':
            recurse		=> true,
            source		=> '/opt/bibbox/sys-bibbox-frontend/js',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-frontend']
        }
        file { '/var/www/html/bibbox-datastore/js/css':
            recurse		=> true,
            source		=> '/opt/bibbox/sys-bibbox-frontend/css',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-frontend']
        }
        file { '/var/www/html/bibbox-datastore/js/images':
            recurse		=> true,
            source		=> '/opt/bibbox/sys-bibbox-frontend/images',
            subscribe	=> Vcsrepo['/opt/bibbox/sys-bibbox-frontend']
        }
        
19. We now delete a small config file to symbolize our initialization scripts, that the configurations have been fully applied and the machine can reapply them in case provisioning is executed again.

        # Remove setup config file to reapply configuration on provisioning
        file { '/etc/bibbox/conf.d/setup.cfg':
            ensure	=> 'absent'
        }
        
20. The following lines will make sure we have all requirements for our upcoming Python scripts installed.

        # Install requirements for liferay setup script
        exec { 'pythonRequirements':
            path		=> '/usr/bin',
            command		=> '/usr/bin/pip3 install -r /opt/bibbox/sys-bibbox-vmscripts/setup-liferay/scripts/requirements.txt'
        }
        
21. In this next step, we install **Docker** and **Docker Compose** from our Puppet **docker_platform** module, create a new default docker network named **bibbox-default-network**, boot up our **sys-activities** Activity monitor and **sys-idmapping** ID mapping service, and apply a small Shell script which makes sure, Liferay can delete our Docker container folders without permission problems when we delete applications.

        # Install docker and docker compose
        include 'docker'
        class { 'docker::compose': 
            version => '1.8.1'
        }


        # Compose and run the sys-activities container, create docker network and copy script for deleting docker containers
        docker_network { 'bibbox-default-network':
            ensure   	=> present,
            subscribe	=> Vcsrepo['/opt/bibbox/sys-activities']
        }
        exec { 'dockerUpActivities':
            path		=> '/usr/bin',
            command 	=> '/usr/bin/sudo /usr/local/bin/docker-compose -f /opt/bibbox/sys-activities/docker-compose.yml up -d',
            timeout     => 1800,
            subscribe	=> Docker_network['bibbox-default-network'],
            require 	=> Class['docker::compose']
        }
        exec { 'dockerUpIdMapping':
            path		=> '/usr/bin',
            command 	=> '/usr/bin/sudo /usr/local/bin/docker-compose -f /opt/bibbox/sys-idmapping/docker-compose.yml up -d',
            timeout     => 1800,
            subscribe	=> Docker_network['bibbox-default-network'],
            require 	=> Class['docker::compose']
        }
        file { "/etc/bibbox/delete_root_folder_applications.sh":
            ensure		=> 'file',
            owner  		=> 'root',
            group  		=> 'root',
            mode		=> '0744',
            source 		=> '/vagrant/resources/delete_root_folder_applications.sh'
        }
        file { "/etc/sudoers.d/liferaydeletescript":
            ensure  	=> 'file',
            owner  		=> 'root',
            group  		=> 'root',
            mode		=> '0440',
            source 		=> '/vagrant/resources/liferaydeletescript'
        }
        
22. Our web services like the ID mapper, Activity monitor, application store and Liferay portal must be reachable by URL. For this purpose we now create some virtual host configuration in our Apache, which points to these services.

        # Configure vhosts for apache
        file { "/etc/apache2/sites-available/000-default.conf":
            ensure  => 'file',
            content => epp('/vagrant/resources/templates/000-default.conf.epp', {
                'bibboxbaseurl'	=> $bibboxbaseurl
            })
        }
        file { "/etc/apache2/sites-available/001-default-application-store.conf":
            ensure  => 'file',
            content => epp('/vagrant/resources/templates/001-default-application-store.conf.epp', {
                'serveradmin'	=> $serveradmin,
                'bibboxbaseurl'	=> $bibboxbaseurl
            })
        }
        file { "/etc/apache2/sites-available/050-liferay.conf":
            ensure  => 'file',
            content => epp('/vagrant/resources/templates/050-liferay.conf.epp', {
                'bibboxbaseurl'	=> $bibboxbaseurl
            })
        }
        
23. In this piece of Puppet code we create symbolic links for our new Apache vhosts in order to make them active. This will also reload Apache and tell it to look out for those new hosts.

        # Symlink the new vhosts config files
            file { '/etc/apache2/sites-enabled/001-default-application-store.conf':
                ensure		=> 'link',
                target		=> '/etc/apache2/sites-available/001-default-application-store.conf',
                subscribe => File['/etc/apache2/sites-available/001-default-application-store.conf']
            }
            file { '/etc/apache2/sites-enabled/050-liferay.conf':
                ensure		=> 'link',
                target		=> '/etc/apache2/sites-available/050-liferay.conf',
                subscribe => File['/etc/apache2/sites-available/050-liferay.conf']
            }
        
24. At the very end of our **vmbuilder** class we copy our own URL configuration into place to enable our very own URL navigation system for the ReactJS GUI. This is basically our own routing configuration. With this, the class is closed and the BIBBOX fully set up. All we have to do now is wait, until Liferay has fully bootedn and we can log in to our new BIBBOX.


            # Copy 'urlrewrite.xml' to tomcat
            file { "/opt/liferay/tomcat-8.0.32/webapps/ROOT/WEB-INF/urlrewrite.xml":
                ensure 	=> 'file',
                source 	=> '/vagrant/resources/urlrewrite.xml'
            }

        }
        
25. Enjoy your new BIBBOX and all it's nice apps and features!