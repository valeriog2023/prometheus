Prometheus is based on
- Metrics - very generic name (same for every source)
- Labels - provide multi dimensions to a metric; ech metric can be associated with multiple labels
  Labels are comma separated k=v pairs inside curly braces
The combination of a metric and a unique set of labels defines a time series

Note: when browsing a metric in the browser
you can do
metric_name                  e.g process_cpu_seconds_total
metric_name[period of time], e.g. process_cpu_seconds_total[5m]
metric_name{k=v}[time range] e.g process_cpu_seconds_total{job="prometheus"}[5m]

Metric Types refer to assumption of the kind of data exporter provides
- **Counter**: single number that only increases or resets to zero<P>
- **Gauge**: single number that can increase and decrease ovr time<P>
- **Histogram**: number of events that fall into a configurable set of buckets
             Buckets are differentiated by labels and they are each an independent time series  
             Histogram also include native global metrics  
               *<metric_name>_sum*  
               *<metric_name>_count*  
            E.G check  
            **prometheus_http_request_duration_seconds_bucket[5m]
            prometheus_http_request_duration_seconds_bucket{le="+Inf"}
            prometheus_http_request_duration_seconds_bucket{le="0.1"}**
            ....
- **Summary**: similar to Histogram but exposes metrics as quantiles insterad of buckets
           **prometheus_http_request_duration_seconds{quantile="0.95"}**  
           same as histogram they also give generic _sum and _count    


## Promtheuse Query Language PromQL
- Browser
- API
- Grafana

### Selector 
   The basic component is the selector which is a metric name possibly with a label
   if no label value is specified you will get all the possible values of the label (each is a specific time series)
  == Label matching operators can be used  
     =  : match value  
     != : don't match value  
     =~ : regex match, e.g. node =~"s.*"  
     !~ : regex does not match e.g. node !~"user|system"   

### Range Vector 
   Range Vector (also called instant vector) - to be added to th metric
   returns a range (vector) of values for a time series, just specify a time value in [ ]  
   *process_cpu_seconds_total[5m]* : values of node_cpu in the last 2 minutes     

### Offset modifier   
   Used to query about metrics from a specific point in time, e.g.  
   *process_cpu_seconds_total[5m] offset 1h*  
   gives a five minute period from an hour ago

### Operators

* Basic aritmetic operators: just run the math operation, e.g.     
  *process_cpu_seconds_total * 2*  

* Operations involving more than one matric at the same time, use matching rules.  
  By **default** you can only apply operations like add the values of one series to another   
  **only if all their labels match** e.g.  <P>
  *node_cpu_seconds_total{mode="system"} + node_cpu_seconds_total{mode="user"}* <P>
  will return nothing because the labesl don't match.
  You can however control this behaviour using the options: **ignoring** and **on**   
  E.g.  
  *node_cpu_seconds_total{mode="system"} + **ignoring(mode)** node_cpu_seconds_total{mode="user"}* <P>
  This  will ignore the different label values: system/user  <P>
  *node_cpu_seconds_total{mode="system"} + **on(cpu)** node_cpu_seconds_total{mode="user"}* <P>
  This will use the cpu label (common to both metrics) and add the series based on that label
* **Comparison Binary Operators**  
  The **default** behaviour is that Prometehus will remove allr esults where the result is **False**  
  Operators are: ==, !=, >, <, >=, <=  
  **Note:** you can use a **bool** modifier to change the default behaviour and
  instead of filtering out the values where the result is False, getting all the
  results values (zero or one). Just need to run the query as follows<P>
  *node_cpu_secods_total == **bool** 0*  
  you wll get the all the metrics (for all labels) that have or not have 0 as a value and the result of the test as the value
* **Logical Set/Binary Operators: and, or, unless**   
    * **and**:  metric1 and metric2  
    will show metric1 but only the time series that match existing combinations in metric2; the label set needs to be present in both metrics
    * **or**: metric1 or metric2, return the values for both
    * **unless**: metric1 unless metric2  
    will show metric1 time series unless labels are present in metric2 
* **Aggregation Operators**  
  sum, min, avg, max, stddev, count, count_values, bottomk, topk, quantile  
  they all do their specific function, they take a value inside ()  
  e.g. avg(metric)


### Query functions
Just have a look at the documentation.. there's a lot..  
clamp_max(time_series_selector, max): returns the value of the series but replaces all values over a threshold with a fixed min  
clamp_min(time_series_selector, min): returns the value of the series but replaces all values under a threshold with a fixed min   
rate(range_vector) : returns average per second rate of increase in a time series 
         how fast or how slow the time series changes.. useful for alerts




### Query API
Use the enpoint: /api/v1/query  
E.G.
```curl localhost:9090/api/v1/query?query=node_cpu_seconds_total```  
you will get back a json result. for more complex queries, you can use data encode  
```
curl localhost:9090/api/v1/query \
     --data-urlencode "query=node_cpu_seconds_total{cpu=\"0\"}"  
```  
For vector queries it is slightly different:  
Use the /api/v1/query_range endpoint to query a range vector.  
Supply **start** and **end** parameters to specify the start and end of the time-range.  
The **step** parameter determines how detailed the results will be. A step duration of 1 m will return values for every one minute during the specified time range:
```
start=$(date --date '-5 min' +'%Y-%m-%dT%H:%M:%SZ')
end=$(date +'%Y-%m-%dT%H:%M:%SZ')

curl "localhost:9090/api/v1/query_range? query=node_cpu_seconds_total&start=$start&end=$end&step=1m"
```   
Note tha promtheus expect this format for dates: ```%Y-%m-%dT%H:%M:%SZ``` 


### Expression browser
Expression browwser is the native Prometheus UI, available at http://<ip>:9090
**Console templates** are a way to customize templates using the native prometheus server; they are written in golang template language (similar to jonja2) and they can include html
when you init the srver you can specify the location of the templates with the option
```
--web.console.templates <path, e.g. "/etc/prometheus/templates">
```
When using the browser you can see the available templates at
```
http://<ip>:9090/consoles
```
A simple template file would be something like this below; note that 
 * To create the template you place the content in a *<file.html>* file inside the templates folder
 * To access the template you point to ```http://<ip>:9090/consoles/<file.html>```
 * it uses an existing tempalte as a basis: **prom query drilldow**
 * the query is pased inside  the value of **args**
 * you can use a **graph** library
      
```
{{template "head" .}}
{{template "prom_content_head" .}}
<h1>Disk IO Rate</h1>
<br />
Current Disk IO: {{ template "prom_query_drilldown" (args "rate(node_disk_io_time_seconds_total{job='Linux Server'}[5m])") }}
{{template "prom_content_tail" .}} {{template "tail"}}
```
using Graph library
```
{{template "head" .}}
{{template "prom_content_head" .}}
<h1>Disk IO Rate</h1>
<br />
Current Disk IO: {{ template "prom_query_drilldown" (args "rate(node _disk_io_time_seconds_total{job='Linux Server'}[5m])") }}
<br />
<br />
<!-- This is div that will be used to display the Graph; the name will be used later -->
<div id="diskIoGraph"></div>
<script>
<-- Here we create a new Graph                                -->
<-- Note that                                                 -->
<--   node(required) uses javascript to get the id of the div -->
<--   expr(required) contains the expression to graph         -->
new PromConsole.Graph({   
node: document.querySelector("#diskIoGraph"),
expr: "rate(node_disk_io_time_seconds_total{job='Linux Server'} [5m])"
})
</script>
{{template "prom_content_tail" .}} {{template "tail"}}
```