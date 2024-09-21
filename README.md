
---

# WSD Assessment
**Referring to check the snap & deployment file**

## Configuration Management

### 1) Displaying Ansible configuration for a host
**Question:** Which Ansible command can display all Ansible configuration for a host?  
**Answer:** Use the following command:

ansible -m setup hostname


### 2) Configure a cron job for log rotation
**Question:** Please configure a cron job that runs logrotate on all machines every 10 minutes between 2h - 4h.  
**Answer:** Please refer to the snapshot and deployment file for the configuration.

### 3) Deploy NTPD package on servers
**Question:** Please deploy the NTPD package to the following 3 servers.  
**Answer:** Due to lab environment limitations, I used only one VM to apply this job. You can follow the same approach for all jobs. Please check the snapshot and deployment file.

---

## Docker/Kubernetes

### Suggested Environment
- **OS:** Ubuntu 20 LTS
- **Docker Version:** Docker 19 or above

### 1) Prepare Docker Compose for Nginx Server
**Requirements:**
- Nginx logs need to persist between container restarts.
- Docker should use a network bridge with the subnet `172.20.8.0/24`.

**Answer:** Please refer to the `docker-compose.yml` file and snapshots.
**Note:** Here my IP was conflicted with the provided one that's why I am using the 172.25.8.0/24 range.

### 2) Identify reason for pod restart in Kubernetes
**Question:** Which Kubernetes command will you use to identify the reason for a pod restart in the project "internal" under namespace "production"?  
**Answer:** 
```bash
kubectl describe pod [pod-name] -n production
```

### 3) Analyze pod resource quota for restarts
**Scenario:**  
Application pod resource usage is as follows:

| Pod Name               | CPU (cores) | Memory (bytes) |
|------------------------|-------------|----------------|
| java-app-7d9d44ccbf    | 3m          | 951Mi          |
| java-app-logrotate     | 1m          | 45Mi           |
| java-app-fluentd       | 1m          | 84Mi           |
| mongos                 | 4m          | 62Mi           |

Resource quotas:
- **Memory:** Request 1000Mi, Limit 1500Mi
- **CPU:** Request 1000m, Limit 2000m
- **Xmx:** 1000M

**Question:** Why does the Java app keep restarting?  
**Answer:** Possible reasons include:
- The Xmx setting (1000M) is too close to the memory request (1000Mi), leaving little overhead.
- CPU limit might be exceeded during peak usage.
- Memory limit may be exceeded since total memory usage is 1142Mi, which is close to the limit (1500Mi).

---

## Helm

### 1) Deploy Elasticsearch using Helm
**Question:** Use the accompanying Elasticsearch Helm template to deploy Elasticsearch.  
**Answer:** Please refer to the screenshot and deployment YAML file. 
**note** Due to lab environment limitations, I used only one VM for Master & another one is Worker to apply for this job. we can follow the same approach for all jobs. Please check the snapshot and deployment file.

---

## Metrics

### 1) How Prometheus Works
**Question:** Explain how Prometheus works.  
**Answer:**  
Prometheus is an open-source monitoring and alerting toolkit. Here's how it works:
- **Target Discovery:** Prometheus discovers services (targets) to collect metrics from.
- **Metrics Collection:** Prometheus scrapes metrics from HTTP endpoints at regular intervals.
- **Storage:** Metrics are stored in a time-series database with timestamps.
- **Querying:** Users query data using PromQL for real-time monitoring.
- **Alerting:** Prometheus triggers alerts based on defined thresholds.

### 2) Creating Custom Prometheus Alerts
**Question:** How do you create custom Prometheus alerts for Kubernetes monitoring?  
**Answer:** Follow these steps:
1. **Define Alerting Rule** in a YAML file using PromQL.
2. **Configure Prometheus** by adding the alert rule to its configuration.
3. **Integrate Alertmanager** to handle notifications.

**Example Alert Rule (High CPU Usage Alert):**
```yaml
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
```

### 3) Prometheus Query for Grafana
**Question:** What is the Prometheus query for visualizing the usage trend of an application metric that is a counter?  
**Answer:** Use the `rate()` function:
```promql
rate(my_metric_name[5m])
```
- `rate(my_metric_name[5m])` calculates the average per-second rate of increase over 5 minutes.

---

## Databases

### Cassandra
**Question:** Query to the Cassandra DB cluster returns different results. Why and how can this be avoided?  
**Answer:**
- Ensure all nodes are up and synchronized.
- Use consistency level `QUORUM` or `ALL` for critical reads.
- Perform regular read repairs.
- Use `nodetool repair` for manual repairs.

### MongoDB
**Question:** How do you shard the `sanfrancisco.company_name` collection based on `_id` in MongoDB?  
**Answer:** Follow these steps:
1. Start three MongoDB instances.
2. Connect to one instance and initiate the replica set:
```js
rs.initiate({
  _id: "replicaset_1",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
```
3. Verify the replica set status with `rs.status()`.

---
