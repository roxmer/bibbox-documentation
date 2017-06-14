# Requirements

### Hardware

**Minimum**

The following will show you the minimum requirements of the BIBBOX virtual machine. Please note, that installing any applications within BIBBOX will require additional resources!

* CPU cores: 1
* Memory: 4096 MB
* Disk space: 20 GB

**Recommended for Development**

For development purposes we recommend a virtual machine with the following specifications:

* CPU cores: 4
* Memory: 8192 MB
* Disk space: 100 GB

**Production**

For production, please calculate the additional resources you will need, depending on the applications you are going to install within the BIBBOX.


### Software

* Download and install VirtualBox -> <https://www.virtualbox.org>

# Installation instructions

### 1.) Download BIBBOX image

[SERVER] - If you want to run BIBBOX on a remote server and access it from outside with a browser, download the latest image from <http://downloads.bibbox.org/bibbox/without-GUI/bibbox-latest.ova>

[LOCAL] - If you want to run BIBBOX on your local PC and just do some tests, download the latest image from <http://downloads.bibbox.org/bibbox/with-GUI/bibbox-latest.ova>

### 2.) Import the image into VirtualBox

If you only have access via command line, you can import the image with the following command:
```vboxmanage import path/to/bibbox-latest.ova```

If you have a user interface on your system (e.g. it's your local PC), follow these steps:

* Open up VirtualBox on your PC or Mac
* CLick on **File > Import Appliance...**
* Select the downloaded image and click **Continue**

### 3.) Start the machine

#### 3a.) LOCAL

If you download the image for local testing, this machine comes with a GUI (Graphical User Interface) and is ready to be used as is.

You can just log in with username **vagrant** and passwords **bibbox4ever** and start using the BIBBOX by accessing it in a Browser like Firefox at the URL <http://bibbox.local.test>.

Please be aware, that after the virtual machine has started, it takes several minutes until the BIBBOX can be accessed.

#### 3b.) SERVER

If you decided to download the server version of BIBBOX without the GUI, you will need to do some manual configuration in order to use the BIBBOX.

1. First of all, you will need to decide for an URL on which to access the BIBBOX. For this, you or your organisation needs to provide a domain of the likes of **bibbox.org** or **your-domain.com**. For this example, we assume the domain is name **your-domain.com** and we want to access the BIBBOX at **bibbox.your-domain.com**.
2. If you only have access to the terminal of your server, you can start the BIBBOX virtual machine with `VBoxManage startvm "BIBBOX_VM_NAME" --type headless`. Otherwise just start the machine from the VirtualBox GUI and log in with user **vmadmin** and password **bibbox4ever**.
3. In case you are following this guide from command line, you will need to connect to the virtual machine by running `ssh vmadmin@192.168.10.10`, then accept the ECDSA key fingerprint by entering **yes** and providing the passwort for the user **vmadmin*, which is set to **bibbox4ever** by default.
4. Now you need to make some small changes in multiple files. Please open the following files one by one with `sudo nano path/to/file`, change the URL **eb3kit.bibbox.org** to your URL (e.g. **bibbox.your-domain.com**) and save the files with **CTRL + O** and **ENTER**. You can exit the editor with **CTRL + X**.:

    * In `/etc/bibbox/bibbox.cfg`
    
        change
    
            bibboxbaseurl="eb3kit.bibbox.org"
            
        to
        
            bibboxbaseurl="bibbox.your-domain.com"
            
    * In `/etc/apache2/sites-available/000-default.conf`
    
        change
    
            ServerName error.eb3kit.bibbox.org
            
        to
        
            ServerName error.bibbox.your-domain.com
            
    * In `/etc/apache2/sites-available/001-default-application-store.conf`
    
        change
    
            ServerName eb3kit.bibbox.org
            
        to
        
            ServerName bibbox.your-domain.com
            
    * In `/etc/apache2/sites-available/050-liferay.conf`
    
        change
    
            ServerName eb3kit.bibbox.org
            
        to
        
            ServerName bibbox.your-domain.com
            
    * And in `/etc/hosts`
    
        change
    
            127.0.0.1       eb3kit
            
        to the first part of your URL, e.g.
        
            127.0.0.1       bibbox
            
5. Finally run `sudo service apache2 reload` to make the changes take effect.

#### 4.) Network configuration

If your hosting provider offers you an administration panel for managing domains and subdomains, you should use that to point to your BIBBOX. Otherwise, if you use Apache, you can do it yourself using this guide:

1. On your server navigate to your apache configuration directory. On Linux base machines this defaults to **/etc/apache2/sites-available**.
2. Create a file named **005-bibbox.conf**. On Linux based systems you can do this with `nano 005-bibbox.conf`.
4. Copy this proxy configuration into the file, change the ServerName, ServerAlias and the port you configured for your virtual machine. You should also change the name of the log files according to your vm name. Then save with **CTRL + O** and **Enter**.

        <VirtualHost *:80>
            ServerName bibbox.your-domain.com
            ServerAlias *.bibbox.your-domain.com

            <Proxy *>
                Order deny,allow
                Allow from all
            </Proxy>

            ErrorLog ${APACHE_LOG_DIR}/bibbox.error.log

            # Possible values include: debug, info, notice, warn, error, crit,
            # alert, emerg.
            LogLevel debug

            CustomLog ${APACHE_LOG_DIR}/bibbox.access.log combined

            ProxyRequests           Off
            ProxyPreserveHost On
            ProxyPass               /api/kernels/       ws://127.0.0.1:80/api/kernels/
            ProxyPassReverse        /api/kernels/       ws://127.0.0.1:80/api/kernels/
            ProxyPass               /       	        http://127.0.0.1:80/
            ProxyPassReverse        /       	        http://127.0.0.1:80/
        </VirtualHost>

5. Now navigate to the **/etc/apache2/sites-enabled** directory and create a symbolic link to your new proxy file with `ln -s ../sites-available/005-bibbox.conf`. 
6. Next reload Apache to make it recognize your changes by running `service apache2 reload`.
7. You can now access the BIBBOX from anywhere in the web by calling your URL (e.g. **bibbox.your-domain.com**) in the browser's address bar!

![alt text](images/installation/bibbox.jpg "Welcome to BIBBOX")