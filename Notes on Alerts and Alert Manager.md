
#  Alert Manager

See some documents herE: https://prometheus.io/docs/alerting/latest/configuration/

## Aler Manager Installation
There are different ways to run Alert Manager, e.g. its own container  
In this case however we will set it up using binary from the site  
As usual:  
*  Create a new service account user
   ```sudo useradd -M -r -s /bin/false alertmanager```
*  Download, untar, copy the file into /usr/local/bin and set ownership
   ```
   wget https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz
   tar xvfz alertmanager-0.20.0.linux-amd64.tar.gz 
   sudo cp alertmanager-0.20.0.linux-amd64/alertmanager /usr/local/bin/ 
   sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager 
   ```
*  Create a directory for the config file 
   *  set ownership and copy the config file from the untar files
   *  create a data directory and set ownership
   ```
   sudo mkdir -p /etc/alertmanager 
   sudo cp alertmanager-0.20.0.linux-amd64/alertmanager.yml /etc/alertmanager 
   sudo chown -R alertmanager:alertmanager /etc/alertmanager 
   sudo mkdir -p /var/lib/alertmanager 
   sudo chown alertmanager:alertmanager /var/lib/alertmanager
   ```   
*  Create a unit file to start a service in systemd
   ```
   sudo vi /etc/systemd/system/alertmanager.service

   [Unit]
   Description=Prometheus Alertmanager
   Wants=network-online.target
   After=network-online.target
   
   [Service]
   User=alertmanager
   Group=alertmanager
   Type=simple
   ExecStart=/usr/local/bin/alertmanager \
     --config.file /etc/alertmanager/alertmanager.yml \
     --storage.path /var/lib/alertmanager/
     # Note that if this is a cluster you have to add one extra 
     # entry for every other member of the cluster (AM cluster port 
     # is 9094). Also the last line before this of course needs a \
     # --cluster.peer=<SERVER2_IP>:9094    
   
   [Install]
   WantedBy=multi-user.target
   ```
*  start/enable the service and verify it runs
   ```
   sudo systemctl start alertmanager
   sudo systemctl enable alertmanager
   sudo  systemctl status alertmanager 

   #Note if you are changing a live unit file you might also need a systemctl daemon-reload

   # you can also browse to ttp://<SERVER_IP>:9093.
   ```

*  Also install **amtool** : a CLI tool bundled with Alertmanager and  
   Verify amtool is working by pulling the current Alertmanager configuration:
   ```
   sudo cp alertmanager-0.20.0.linux-amd64/amtool /usr/local/bin/
   sudo mkdir -p /etc/amtool
   sudo cat cat << EOF >> /etc/amtool/config.yml
   alertmanager.url: http://localhost:9093
   EOF

   amtool config show

   [..redacted output..]
   ```
*  Finally configure **Prometheus** to interact with **Alert Manager**
   Under alerting, add your Alertmanager as a target and then restart prometheus
   
   ```
   sudo vi /etc/prometheus/prometheus.yml

   alerting:
     alertmanagers:
     - static_configs:
       - targets: ["localhost:9093"]
   # Note: if you have a cluster of Alert Manager hosts you just put them
   # together in the targets list, e.g.
   #    - targets: ["localhost:9093", "10.1.0.102:9093"]  
   
   sudo systemctl restart prometheus
   ```
   *  Verify Prometheus is able to reach the Alertmanager.  
      Access the Prometheus Expression Browser in a web browser at  
      ```http://<PROMETHEUS_IP>:9090/graph```. Run the following query and ensure the current value is **1**: **prometheus_notifications_alertmanagers_discovered**
    *  Noite: you can also see the alert managers configured in ```http://<PROMETHEUS_IP>:9090/status``` 


## Some Alert configuration
Alert manager configuration is available in ```/etc/alertmanager/alertmanager.yml```   
Alert manager clusters replicate the config that you create on one or the other  
however remember that they still have their own config file

Some parameters are global, e.g.:  
  * **resolve_timeout**: if an alert does not come with its own resolve timeout, this is applied
  * **smtp_from**: for email alerts, this is the field **from**
  * **slack_parameters**: check web page for slack parameters and other stuff

Alert manager can load alert templates, so it can be a good idea to create a folder for them:
```/etc/alertmanager/templates/```
You also need to add this entry in the alertmanager.yml file:
```
templates:
  - "/etc/alertmanager/templates/*.tmpl"
```

