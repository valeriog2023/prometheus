# Recording Rules


Recording rules in Prometheus:  
*  allow you to precompute the values of expressions and queries
and save the results in their own separate set of time-series data(metric).
*   are evaluated on a configurable schedule,
*   useful when you have complex or expensive queries that are run very frequently.
    as it allows to simply pulling the pre-computed data
*   they are configured using YAML, and you place YAML files in a location specified 
    under the rule_files section in prometheus.yml
*   when changed, need service reload


# Examples

## Implement a Recording Rule to Pre-Calculate CPU Usage

* Create a directory to store rules files: ```sudo mkdir -p /etc/prometheus/rules```
* Edit the Prometheus config and add a rules location:
```
sudo vi /etc/prometheus/prometheus.yml
...

rule_files:
  - "/etc/prometheus/rules/*.yml"

```
*  Create a rules file for a server metrics and a rule to monitor CPU.  
   Note that some parameters are defined per group of rules (e.g. interval)
```
sudo vi /etc/prometheus/rules/rule1.yml

groups:
- name: <group1_rule>
  # How often rules in the group are evaluated.
  [ interval: <duration> | default = global.evaluation_interval ]
  # Limit the number of alerts an alerting rule and series a recording rule can produce. 0 is no limit.
  [ limit: <int> | default = 0 ]

  rules:
      # name of the new metric
    - record: <server1>:cpu_usage
    
      # promql expression to run               
      expr: sum(rate(node_cpu_seconds_total{instance='<server1>',mode!='idle'}[5m])) * 100 / 2
      
      # Labels (optional) to add or overwrite before storing the result.
      labels:
       [ <labelname>: <labelvalue> ]

    - record: <server1>:memory_usage
      expr: 100 - (node_memory_MemAvailable_bytes{instance='<server1>'} / node_memory_MemTotal_bytes{instance='<server1>'}) * 100   
```
*  You can verify if a rule file is correct with: ```promtool check rules /path/to/example.rules.yml```
*  Restart prometheus server: ```sudo systemctl restart prometheus```

*  You can now check that the new metrics <server1>:memory_usage, <server1>:cpu_usage are available