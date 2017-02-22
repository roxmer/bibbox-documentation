# BIBBOX and eB3Kit Dokumentation

The [User Documentation](http://bibbox.readthedocs.io/en/latest/) for BIBBOS and the B3Africa eB3Kit is made by mkdocs, and can be 
found @ [http://bibbox.readthedocs.io/en/latest/](http://bibbox.readthedocs.io/en/latest/).

The developer documentation is at the specific bibbox repository. The repositories are strcutured according to the following naming conventions:

* **application-store** repositores describes a lost of all applications (applications.json) and a annotated list of application for a 
    specific applicatoin doamin, called kits. Currently we provide one defaults kit, described in eB3kit.json 

* **KIT-.... repositories** provides the blueprint "kits". A kit is a VM providing the base BIBBOX framework togethers with a set of predefined 
   tools (kit.json ind the applications directory) and pre-loaded images and (reference) databases.  Currently we provide one defaults kit, 
   described in  **kit-eb3kit**

* **APP-.... repositories** provide the blueprint for APPs. Each bibbox application is made up my a set of either offical docker images or 
   docker images wiethin the BIBBOX docker hub [BIBBOX docker hub](https://hub.docker.com/r/bibbox/). 

* **SYS-.... repositories** provides the source code running inside the virtual machine, which provides all services of the BIBBOX SaaS framework. 
  * **sys-bibbox-backend-liferay** backend of the BIBBOX portal, based on the liferay framwork. This provides functionaty for the management of 
     applications instances, the application store and the central user management. 
  * **sys-bibbox-frontend** all frontend code (React libraries). They are loaded by the backend at runtime. 
  * **sys-activities** Actitivities sercice, for logging and synchronisation of all high level activities (installation, dleeting, start/stop, etc.) 
          running as a docker container.  
  * **sys-idmapping** ID mapping sercice, for mapping of internal IDs and connection to external ID systems (B2Handle, EPIC) running a a docker container. 
  * **sys-bibbox-vmscripts** Collection of phyton scripts for
     * Management of applications, e.g. installation, port management, file management.
     * Setup script for the configuration of the portal, called after vagrant / puppet scripts are finished.
     
* **RES-.... repositories**  ressources. e.g. a common ICON set for biobank applications.  