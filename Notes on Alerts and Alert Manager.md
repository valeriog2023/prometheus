
#  Install alert Manager
There are different ways to run Alert Manager, e.g. its own container  
In this case however we will set it up using binary from the site  
As usual:  
*  Create a new service account user
*  Download, untar, copy the file into /usr/local/bin and set ownership
*  Create a directory for the config file 
   *  set ownership and copy the config file from the untar files
   *  create a data directory and set ownership
*  Create a unit file to start a service in systemd