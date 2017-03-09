# Running BIBBOX portal

### How to start the portal
### How to install an App
- The App Store
- App Dashboard / GUI
- Working in the terminal
### Export / Import / Backup of Apps

### Troubleshooting
  - Login via SSH
  
        host:$ vagrant ssh
  - Check if you have enough diskspace left 
       
        bibbox:$ sudo df -h *

  - Check if Apache is running
  
          bibbox:$ sudo service apache2 status

  - Check the liferay log

          bibbox:$ tail -f /opt/liferay/tomcat-8.0.32/logs/catalina.out
