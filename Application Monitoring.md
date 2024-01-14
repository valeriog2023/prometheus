You can use exporter to monitor nodes but also specific applications  
Some examples  
  *  Apache exporter
     * Create a user/group for the exporter to run with
     * download and untar the exporter
     * move the binary to /usr/local/bin and set the owner to the user/group you created
     * create a unit file to start the exporter as new service. The service will run the exporter
     * metric will be made available to localhot:9117
<P>
In the exporter server create a scrape config:
  * add a job in the static config section pointing to the IP:port where the exporter runs
  * restart Prometheus

<P>

# Job and Instances
Prometheus automatically add instace labels  
Instances are endpoints Prometheus scrapes  
Job is a collection of instances scraped for the same purpose

E.g.  
job_name: api_server  
  instance: <ip1>
  instance: <ip2>
  instance: <ip3>

Prometheus automatically creates meta metrics, e.g.  
  *  **up{job="<job_name>, instance="<instance_name>}"**:  
     was the scrape successful  
  *  **scrape_duration_seconds"<job_name>, instance="<instance_name>}**:   
     number of seconds it takes to complete the scrape, etc..




# Push Gateway
If a job runs for a period of time, on a schedule or on events, metrics can't be pulled  
but need to be pushed. (only use when necessary) Push Gateway works as a middle man:
  *  Jobs push their metrics to the gateway
  *  Prometheus scrapes the push Gateway