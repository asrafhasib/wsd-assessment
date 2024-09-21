# wsd-assessment

#Configuration management#

1)	Which ansible command can display all ansible_ configuration for a host?
   
   Answer: Displaying Ansible configuration for a host command: # ansible -m setup hostname
  	
2)	Please configure a cron job that runs logrotate on all machines every 10 minutes between 2h - 4h.
   
   Answer: Please check the snapshot with the deployment file.

3)	Please deploy ntpd package to the following 3 servers:
   
   Answer: For the lab environment limitation, I used only one VM to apply for those jobs & here we can get the idea to run all jobs in the same ways. please check the snapshot and deployment file.


#Docker/Kubernetes#

Suggested environment: Ubuntu 20 LTS, docker 19 or above
1)	Prepare a docker-compose for a nginx server.
Requirements:
•	nginx logs need to survive between nginx container restarts
•	docker should use network bridge subnet 172.20.8.0/24

Answer: Please check the docker-compose.yml file & snapshot to get the answer.

2)	Which Kubernetes command you will use to identify the reason for a pod restart in the project "internal" under namespace "production".
 
   Answer: kubectl describe pod [pod name] -n production

2)	Consider the followings:
POD NAME                                       CPU(cores)         MEMORY(bytes)
java-app-7d9d44ccbf-lmvbc   java-app                  3m           951Mi
java-app-7d9d44ccbf-lmvbc   java-app-logrotate        1m           45Mi
java-app-7d9d44ccbf-lmvbc   java-app-fluentd          1m           84Mi
java-app-7d9d44ccbf-lmvbc   mongos                    4m           62Mi

Application pod has the following resource quota:
•	Memory request & limit: 1000 & 1500
•	CPU request & limit: 1000 & 2000
•	Xmx of 1000M
Java-app keep restarting at random.  From Kubernetes configuration perspective, what are the possible reasons for the pod restarts?

   Answer: Here are some possible reasons for the Java app pod restarts:
         .	The Xmx setting (1000M) is close to the memory request (1000Mi), leaving little room for other processes.
         .	CPU limit may be reached during peak times.
         .	Memory limit exceeded: The total memory usage (951Mi + 45Mi + 84Mi + 62Mi = 1142Mi) exceeds the memory limit of 1500Mi. when it reached the threshold of 80% quotas.

#Helm#

1. Please use the accompanied elasticsearch helm template to create a Kubernetes deployment of elasticsearch. Provide a screenshot & deployment yaml of the resultant deployment in Kubernetes.

   Answer: Please check the screenshot & deployment yaml to get the answer.


#Metrics#

1)	Explain how Prometheus work.

  	 Answer: Answer: Prometheus is an open-source systems monitoring and alerting toolkit 
     How Prometheus Works:
     .	Target Discovery: Prometheus automatically discovers services (called "targets") from which it collects metrics. Targets expose an HTTP endpoint (usually /metrics) in a specific format that Prometheus understands.
     .	Metrics Collection: Prometheus scrapes metrics from these endpoints at regular intervals. It pulls the data (metrics) instead of waiting for it to be pushed.
     .	Storage: The scraped metrics are stored in a time-series database along with timestamps.
     .	Querying: Users or other systems can query the collected data using PromQL (Prometheus Query Language). This data can be used for real-time monitoring and analysis.
     .	Alerting: Based on defined thresholds, Prometheus evaluates rules and triggers alerts when specific conditions are met.


3)	How do you create custom Prometheus alerts and alerting rules for Kubernetes monitoring? Provide an example alert rule and its configuration.

answer: To create  custom Prometheus alerts and alerting rules  for monitoring Kubernetes,  we follow these key steps:

.  Define the Alerting Rule : Write an alerting rule in a YAML file using  PromQL .
.  Configure Prometheus : Add the alerting rule to Prometheus by referencing it in the configuration.
.  Alertmanager Integration : Configure  Alertmanager  to receive and handle alerts (e.g., sending notifications to email, Slack, etc.).

Steps:

 # 1. Create an Alerting Rule

Alerting rules evaluate metrics collected by Prometheus and trigger alerts based on certain conditions.

For example, let's say we want to create a custom alert if the  CPU usage  of a Kubernetes pod exceeds  80%  for more than 5 minutes.

 Example Alert Rule: 

 yaml
groups:
  - name: k8s-monitoring
    rules:
      - alert: HighPodCPUUsage
        expr: 100 * (sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) BY (pod_name)) / sum(kube_pod_container_resource_requests_cpu_cores) BY (pod_name) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected for pod {{ $labels.pod_name }}"
          description: "CPU usage for pod {{ $labels.pod_name }} is above 80% for more than 5 minutes."
 

 # Explanation:
-  alert : Name of the alert (`HighPodCPUUsage`).
-  expr : PromQL expression. This query calculates CPU usage by dividing the CPU usage rate of a pod by its requested CPU cores and then multiplies by 100 to get a percentage. If it exceeds 80%, the alert triggers.
-  for : Ensures that the condition must persist for at least 5 minutes before the alert is triggered.
-  labels : Adds additional metadata to the alert. `severity: warning` indicates the severity level.
-  annotations : Provide additional information about the alert, like a summary and description.

 # Add the Rule to Prometheus Configuration

Save the alert rule in a file, e.g., `k8s-alert-rules.yaml`.

Next, update  our Prometheus configuration to load this alert rule.

Edit  our `prometheus.yml` to reference the alert rule file:

 yaml
rule_files:
  - 'k8s-alert-rules.yaml'
 

Now Prometheus knows to load and apply the alerting rule.

 # Apply the Configuration in Kubernetes

If  we're using Prometheus in Kubernetes,  we'll typically deploy it using Helm or a Prometheus Operator.  we need to ensure that  our alerting rules are loaded by the Prometheus instance running in  the Kubernetes cluster.

- If using  Helm ,  we can add our custom rule file to the `rules` directory or pass it through the `values.yaml` file.
  
- If using  Prometheus Operator, create a `PrometheusRule` custom resource that defines the rules. Example:

 yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: k8s-monitoring-rules
  namespace: monitoring
spec:
  groups:
  - name: k8s-monitoring
    rules:
    - alert: HighPodCPUUsage
      expr: 100 * (sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) BY (pod_name)) / sum(kube_pod_container_resource_requests_cpu_cores) BY (pod_name) > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected for pod {{ $labels.pod_name }}"
        description: "CPU usage for pod {{ $labels.pod_name }} is above 80% for more than 5 minutes."
 

Apply this resource to our Kubernetes cluster:   kubectl apply -f k8s-alert-rules.yaml

   
4)	What is the Prometheus query you can use in Granfana to properly show usage trend of an application metric that is a counter?

Answer: To properly visualize the usage trend of a counter metric in Grafana using Prometheus, we should use the rate() or irate() function. 
These functions compute the per-second average rate of increase for a counter metric over a specified time window.

Prometheus Query:  rate(my_metric_name[5m])

•  rate(your_metric_name[5m]): This calculates the average per-second rate of increase for the counter my_metric_name over the last 5 minutes.
•  irate(): Use irate() for a more instant rate over the last two data points, but for trends, rate() is more stable.

#Databases#

Suggested environment: Cassandra 4.0 or above, mongo 4.4.0 or above
1)	Cassandra
Query to db cluster returns different result each time.  Users reported query result has data records that they deleted days ago.  
Explain what the likely reason for the behavior and how to avoid it.

  Answer: To avoid inconsistent query results:
        .	Ensure all nodes are up and synchronized
        .	Use consistency level QUORUM or ALL for critical reads
        .	Perform read repair regularly
        .	Use nodetool repair for manual repairs


2)	Mongo
We have mongodb replicaset_1 with the following db and collections with some records....
A sample record from company_name:
 

Performance is bad as the hardware of replicaset_1 is not capable to handle the database sanfrancisco.  We added a new replicaset_2.  
Please provide all steps required to shard the collection sanfrancisco.company_name based on _id.

Answer: To set up a MongoDB ReplicaSet:
1.	Start three MongoDB instances
2.	Connect to one instance and initiate the replica set:
  	 rs.initiate({
  _id: "replicaset_1",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})

 
Verify the replica set status:  rs.status()










