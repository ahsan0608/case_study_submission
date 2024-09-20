## Configuration Management

### 1. Ansible Command to Display Configuration for a Host
```bash
ansible <hostname> -m setup
```

### 2. Configure Cron Job for `logrotate`
Cron job that runs `logrotate` every 10 minutes between 2 AM and 4 AM
```bash
*/10 2-4 * * * /usr/sbin/logrotate /etc/logrotate.conf
```

### 3. Deploy NTPD Package, Nagios to Servers
Heres the ansible project [Ansible deploy ntpd](https://github.com/ahsan0608/case_study_submission/tree/main/ansible_deploy_ntpd_nagios).
This project is designed to:
1. Deploy the NTP service with a custom configuration on a set of servers.
```bash
ansible-playbook playbook.yml --tags ntp
```
2. Deploy Nagios monitoring templates to monitor the NTP process on these servers.
```bash
ansible-playbook playbook.yml --tags nagios
```

---

## Docker/Kubernetes

### 1. Docker Compose for NGINX Server where logs persist between container restarts, and it uses a custom network bridge with the subnet `172.20.8.0/24`

```yaml
version: '3'
services:
  nginx:
    image: nginx:latest
    container_name: nginx_server
    restart: always
    volumes:
      - ./nginx-log:/var/log/nginx
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      custom_bridge:
        ipv4_address: 172.20.8.2
networks:
  custom_bridge:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.8.0/24
```

### 2. Kubernetes Command to Identify Pod Restart Reason in Prod

```bash
kubectl describe pod <pod-name> -n production
```
This will display the events, including the reason for the pod restarts.

### 3. Reasons for Random Pod Restarts
The Java application pod may be restarting due to resource constraints:
- The memory request and limit (1000 & 1500) could be insufficient, causing the pod to reach its memory limits and be OOMKilled.
- The CPU request and limit (1000 & 2000) could be too low for the app.
- The Xmx of 1000M may be causing issues when combined with Kubernetes memory limits, leading to restarts.

We can check the events and logs from the pod for further investigation

---

## Helm

### Elasticsearch Deployment Using Helm
I used the provided **elasticsearch_helm** template to create a k8s deployment for Elasticsearch. The Helm chart is configured to deploy a StatefulSet that includes all the necessary configurations for Elasticsearch, such as resource requests, limits, storage, cluster settings etc. I deployed it using the following command:
```bash
helm install customer-abc-elasticsearch ./elasticsearch -f ./elasticsearch/envs/qa/customer-abc-elasticsearch/values.yaml
```

Here’s a link to the Helm project repository:
- **Helm Project (Elasticsearch Deployment)**: [Helm Project Repository Link](https://github.com/ahsan0608/case_study_submission/tree/main/elasticsearch_helm)

After deploying Elasticsearch, I captured the YAML output of the deployment and a screenshot of the running pods using

```bash
kubectl get statefulset elasticsearch-cluster -o yaml
```

<img width="795" alt="Screenshot 2024-09-20 at 11 49 57 PM" src="https://github.com/user-attachments/assets/2c6f84ef-6b4b-43ee-a315-557c46bb4df9">
---

## Metrics

### 1. How Prometheus Works

Prometheus scrapes metrics at predefined intervals and stores them in a time-series database with high precision and offering a flexible query language (PromQL) for analyzing and visualizing data.

### 2. Creating Custom Prometheus Alerts for Kubernetes Monitoring

In Prometheus, we define custom alerts in an alert rule configuration file. Prometheus uses these rules to generate alerts when certain conditions are met.

Here’s an example of a custom alert rule to monitor high CPU usage for a container in Kubernetes:

**Alert Rule Configuration:**
```yaml
groups:
  - name: prom-example
    rules:
      - alert: HighCPUUsage
        expr: sum(rate(container_cpu_usage_seconds_total[5m])) by (container) > 80
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage detected in container {{ $labels.container }}"
          description: "{{ $labels.container }} CPU usage is above 80% for the last 5 minutes."
```

In this rule, Prometheus checks if the CPU usage for a container exceeds 80% over a 5-minute window. If it does, an alert is triggered with a "critical" severity.

### 3. Prometheus Query for Grafana to Show Usage Trend of an Application Metric (Counter)

We can use the `rate()` function to calculate the average increase per second over a given time range. For example this query calculates the rate of change in the `http_requests_total` counter over the last 5 minutes

Here’s the query:
```promql
rate(http_requests_total[5m])
```

## Databases

### 1. Cassandra Query Returns Inconsistent Results
In Cassandra, writes are eventually propagated across replicas, but reads might not always reflect the latest state due to the consistency level set during the read/write operations. The inconsistency in Cassandra query results seems due to eventual consistency. 
To mitigate this we can use a higher consistency level, such as `QUORUM` or `ALL`, for read/write operations to ensure stronger consistency.

### 2. MongoDB Sharding Steps
To shard the `sanfrancisco.company_name` collection in MongoDB:
1. Enable sharding for the database:
   ```js
   sh.enableSharding("sanfrancisco")
   ```
2. Shard the `company_name` collection based on `_id`:
   ```js
   sh.shardCollection("sanfrancisco.company_name", { "_id": 1 })
   ```
3. Add the second replica set (`replicaset_2`) to the sharded cluster to ensure the new collection data is properly distributed across both replica sets.



