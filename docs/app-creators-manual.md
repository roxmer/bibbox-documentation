#  Apps Creators Manual

## How to join the team

Contact [Heimo Müller](mailto:heimo.mueller@medunigraz.at), [Robert Reihs](mailto:robert.reihs@medunigraz.at) or [Lukas Pessel](mailto:lukas.pessel@medunigraz.at). Any of them can add you to the Github team of an App repository. 

## Anatomy of an App

An **App** is described within a BIBBOX Github repository. By convention the name of the repository starts with the prefix **"app-"**
A template repository can be found at:  <https://github.com/bibbox/app-template>


An **App** consists at least of the following files and directories, please never change the name of these files!

* README.md

    The default Github readme file shoud contain information about the used official docker images and docker images from the [BIBBOX dockerhub](https://hub.docker.com/r/bibbox/). In addition you should describe all mounted volumes and their content.            

  
* INSTALL-APP.md

    Install instruction for the APP to follow after the first installation step (docker-compose up) is finished. This typicaly describes application specific configuration tasks. A link to this file is given in the Dashboard of the installed App. 
    
  
* appinfo.json
    
    Specifies Information about the App as displayed in the BIBBOX App Store. 
    
``` code
    {
            "name": "A long name of the Application, shown in the App Store",
            "short_name": "APP short name",
            "version": "V.1.0",
            "description": "Long description of the Application shown in Store, can be multiline",
            "short_description": "Short description of the Application shown in Store",
            "catalogue_url": "URL to biobankapps, if applicaple",
            "application_url": "URL to App official gace",
            "tags": ["Tag1", "Tag2"]
    }
```

* docker-compose-template.yml

    Based on this file the docker-compose.yml will be generated during the installation, for the §§INSTANCE, §§PORT and all other environment and configuration variables staring withn "§§" will be replaced during the installation. Each template for a docker compose file should link to the bibbox-default-network network and use the busybox container as central mount point. 
    
```
 
    version: '2'
    
    networks:
        bibbox-default-network:
          external: true
    
    services:
    
      §§INSTANCE-container1:
        image: bibbox/app-image
        container_name:  §§INSTANCE-container1
        restart: unless-stopped
        networks:
          - bibbox-default-network
        links:
          - §§INSTANCE-lapp-db:app-db
        ports:
          - "§§PORT1:80"
        depends_on:
          - §§INSTANCE-app-db
          - §§INSTANCE-app-data
        volumes_from: 
          - §§INSTANCE-app-data
    
    §§INSTANCE-container2:
      image: bibbox/app-image2
      container_name:  §§INSTANCE-container2
      restart: unless-stopped
      networks:
        - bibbox-default-network
      ports:
        - "§§PORT2:8000"
      depends_on:
        - §§INSTANCE-app-data
      volumes_from: 
        - §§INSTANCE-app-data
    
      §§INSTANCE-app-db:
        image: mysql:8
        container_name: §§INSTANCE-limesurvey-db
        restart: unless-stopped
        networks:
          - bibbox-default-network
        user: root
        environment:
          - MYSQL_ROOT_PASSWORD=thispasswordisneverusededoutsidethecontainer
          - MYSQL_DATABASE=§§MYSQL_DATABASE_NAME
          - MYSQL_USER=§§MYSQL_DATABASE_USER
          - MYSQL_PASSWORD=§§MYSQL_DATABASE_PASSWORD
        volumes_from: 
          - §§INSTANCE-app-data
        depends_on:
          - §§INSTANCE-app-data
    
      §§INSTANCE-app-data:
        image: busybox
        container_name: §§INSTANCE-app-data
        volumes:
          - ./var/app-data:/var/app-data
          - ./var/app-config:/var/app-config
    
```


* environment-parameters.json

    Environment parameters for the docker-compose-template file. The user can set the values for these parameters in a GUI during the installation of an App. 
   
    
```json
    [
       {
          "id":"NAME_AS_IN_DOCKER_TEMPLATE_1",
          "display_name":"Name Displayed in GUI at the Installation",
          "type":"text, number, password",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       },
       {
          "id":"MYSQL_DATABASE_NAME",
          "display_name":"Name of the mysql database",
          "type":"text,",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       },
       {
          "id":"MYSQL_DATABASE_USER",
          "display_name":"Name of the mysql user",
          "type":"text",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       },
       {
          "id":"MYSQL_DATABASE_PASSWORD",
          "display_name":"Password of mySQL user",
          "type":"password",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       }

    ]
```

* configuration-parameters.json

    Configuration parameters for the docker-compose-template file. The user can set the values for these parameters in a GUI during the installation of an App and also in an later interactive configuration panel (will be provided in the next release). 
    
``` json

    [
       {
          "id":"NAME_AS_IN_DOCKER_TEMPLATE_1",
          "display_name":"Name displayed in GUI at the Installation and Configuration",
          "type":"text, number, password",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       },
       {
          "id":"NAME_AS_IN_DOCKER_TEMPLATE_2",
          "display_name":"Name displayed in GUI at the Installation and Configuration",
          "type":"text, number, password",
          "default_value":"Default Value",
          "description":"Description shown in the GUI",
          "min_length": "1",
          "max_length": "128"
       }
    ]
    
```

* filestructure.json

    Specifies a list of directories to be created in the App instance folder, and a list of files to be copied from the App repository to the App instance folder. All files listed here are copied by default to the App instance folder.
    
    
``` json
    {
        "makefolders":[
            "var/app-config",
            "var/app-data"
        ],
        "copyfiles":[
            {
                "source":"config/database/db.conf",
                "destination":"var/app-config/db.conf"
            },
            {
                "source":"config/app-data/db.data",
                "destination":"var/config/app-data/db.data"
            }
        ]
    }
```


* icon.png

    Icon of the App in the application store und in the application dashboard. Please use a square format, e.g. 512x512 pixeland PNG with a transparency channel.
    
  
* sys-info.json

    A list of containers, that must run in order display the container as "running" in the application dashboard. In addtion a list of "non running" containers can be specified. The names must be the same as the service names in the docker-compose-template.yml. 

    
``` json
    {
      runningcontainers:["§§INSTANCE-appcontainer1", "§§INSTANCE-appcontainer2"],
      supportcontainers:["§§INSTANCE-app-data"]    
    }
```
    

* portinfo.json

    Defines the mapping of ports as given in the docker-compose-template.yml to an URL. To achieve this proxy files are generated an the apache server of a BIBBOX instance is reloaded, when an App is installed. If your BIBBOX base URL is `apps.mybiobank.com` the installation of an APP with the instance ID `lims` generates two proxy files, which maps
    
    * `lims.apps.mybiobank.com` to port `80` of `container1`, and
    * `lims-monitoring.apps.mybiobank.com` to port `8000` of `container2`.

``` json    
    {
      "mappings": [
        {
          "id": "§§PORT1",
          "url": "§§INSTANCE",
          "protocol": "HTTP",
          "proxy" : "PRIMARY"
        },
        {
          "id": "§§PORT2",
          "url": "§§INSTANCE-monitoring",
          "protocol": "HTTP",
          "proxy" : "PRIMARY"
        }
      ]
    }
```

* images

Directory containing all docker images used in the docker-compose-template.yml (apart from offcial docker images). For each docker image a sub directory containing the Dockerfile and configs has the created. In the BIBBOX dockerhub these files are used to build the docker images. Please not, that for versioned Apps both in the dockerhub and in the docker-compose-template.yml version tags has to be used. 


## Versioning

Each App should be versioned. We distinguish between

* **Development Version** generated from the master branch of the repository. The developmemt version is identified by the tag `development` in the kit description,  

* **Production Versions** generated from a specific release in the repository. For each major release. i.e. given by a new version of dockerized software tool, a new branch should be generated in the repository. Please note, that the tag name for the Github release should not contain dots as seperators, please use "_" instead. 



## Register an App

For the registration of an App two configuration files have to be extended:

* [applications.json](https://github.com/bibbox/application-store/blob/master/applications.json) maps a Github repository to a human readable name of the App 


* [eB3Kit.json](https://github.com/bibbox/application-store/blob/master/eB3Kit.json) puts the App into a specific **kit**. A **kit** defines ta group of Apps together with metadata, which are then displayed in the App store and can be installed in a BIBBOX instance. Today eB3kit is the default kit of the B3Afria project, in the future further kits will be defined. 




