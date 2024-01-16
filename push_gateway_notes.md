# Push Gateway installation  

*  Create a local account user to run push gateway  
   Note: no home /  no bash login 
```sudo useradd -M -r -s /bin/false pushgateway```
*  Download the pushgateway files and extract them, copy them /usr/local/bin an change the owner
   ```
   wget https://github.com/prometheus/pushgateway/releases/download/v1.2.0/pushgateway-1.2.0.linux-amd64.tar.gz
   tar xvfz pushgateway-1.2.0.linux-amd64.tar.gz
   sudo cp pushgateway-1.2.0.linux-amd64/pushgateway /usr/local/bin/
   sudo chown pushgateway:pushgateway /usr/local/bin/pushgateway
   ```
*  Create a unit file to start\enable the service
   ```
   sudo vi /etc/systemd/system/pushgateway.service   

   [Unit]
   Description=Prometheus Pushgateway
   Wants=network-online.target
   After=network-online.target   

   [Service]
   User=pushgateway
   Group=pushgateway
   Type=simple
   ExecStart=/usr/local/bin/pushgateway   

   [Install]
   WantedBy=multi-user.target   

   systemctl start pushgateway
   systemctl enable pushgateway
   ```
*  Verify PushGateway status andd that it exposes metrics port: 9091
   ```
   sudo systemctl status pushgateway
   curl localhost:9091/metrics
   ```

*  In the Prometheus server, add a scraping config to get the metrics from Pushgateway
   Be sure to set **honor_labels: true** so that labels like job name and instance sent  
   to the pushgateway will be honored (otherwise Prometheus will replace them with the  
   values used for thes sraping job)
   *  Edit the /etc/prometheus/prometheus.yml ( under the **scrape_configs:** section)
   ```
   sudo vi /etc/prometheus/prometheus.yml

   - job_name: 'Pushgateway'
     honor_labels: true
     static_configs:
     - targets: ['localhost:9091']
   ```
   *  Restart Prometheus to load the new configuration:  
      ```sudo systemctl restart prometheus```

   *  Verify that The Prometheus server can scrape the gateway: http:<IP>:9090/  
      Run a query to view some Pushgateway metric data: pushgateway_build_info  

<P>
<P>

# Push Metric Data to Pushgateway  

Note: you want to specify the application *job_name* and *instance* and have the Push Gateway honor those
Or Prometheus will replace them with the values of the setting for the job used to scrape the Push Gateway

In Python here you have a small snippet of code to send a metric to push gateway
```
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

registry = CollectorRegistry()
g = Gauge('job_last_success_unixtime', 'Last time a batch job successfully finished', registry=registry)
g.set_to_current_time()
push_to_gateway('localhost:9091', job='batchA', registry=registry)
```

In Bash, you can use curl; Note that:  
*  job_name and instance IP are passed as part of the url
*  multiple metrics can be sent with the same message
*  data is sent as binary 
*  keep the 2 spaces before the metrics in the message
```
cat << EOF | curl --data-binary @- http://<pushgatewayIP>:9091/metrics/job/<job_name>/instance/10.0.1.102
  # TYPE job_executed_successful gauge
  job_executed_successful 1
  # TYPE job_num_files_deleted gauge
  job_num_files_deleted $num_files
EOF
```