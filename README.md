
# Prometheus, Grafana, Jenkins Integration with Host Monitoring and Slack Alerts

This project provides an automated monitoring infrastructure setup using Prometheus and Grafana with Jenkins CI/CD integration.
It includes automatic deployment, Jenkins monitoring, alert management, and Slack notifications.
## Project Structure

The project is divided into two main components:

- Jenkins: This part manages your CI/CD pipeline using Docker.
- Prometheus-Grafana: This section contains configurations for Prometheus, Grafana, and AlertManager.

Project Overview
This solution automates the setup and monitoring of infrastructure components with a focus on:

Automated deployment of the entire monitoring stack
Jenkins monitoring using Prometheus and CloudBees disk usage plugins
Real-time alerts delivered to Slack
Configuration validation through CI/CD pipeline

---

    ├── jenkins
    │   ├── docker-compose.yml
    │   └── sre.Jenkinsfile
    ├── prometheus-grafana
    │   ├── alertmanager
    │   │   └── alertmanager.yml
    │   ├── docker-compose.yml
    │   ├── grafana
    │   │   └── provisioning
    │   └── prometheus
    │       ├── alert_rules.yml
    │       └── prometheus.yml
    ├── README.md
    └── vagrant
        └── Vagrantfile
---

## Prerequisites

Before setting up, ensure that you have the following installed:

- **Docker**: To run containers for Prometheus, Grafana, Jenkins, and Alertmanager.
- **Docker Compose**: For easier management of multi-container Docker applications.
- **Jenkins**: For running CI/CD pipelines and exposing Prometheus metrics.
- **Slack Webhook URL**: For Slack notifications when an alert is triggered.

## Architecture Overview

The setup includes the following services:

- **Prometheus**: Collects metrics from Jenkins, the VM host, and other services.
- **Grafana**: Visualizes the collected metrics through dashboards.
- **Jenkins**: Exposes metrics for Prometheus to scrape.
- **Alertmanager**: Triggers Slack notifications based on alerts defined in Prometheus.
- **Node Exporter**: Collects system-level metrics (CPU, memory, disk, network) from the VM host for Prometheus to scrape.

## Setup

### Step 1: Docker Compose Setup

We use Docker Compose to manage the services. Ensure you have `docker-compose.yml` configured correctly.

Example for `docker-compose.yml`:

```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    networks:
      - prometheus-grafana_monitoring
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    container_name: grafana
    networks:
      - prometheus-grafana_monitoring
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/etc/grafana/dashboards

  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    networks:
      - jenkins_monitoring
    ports:
      - "8080:8080"
      - "50000:50000"
    environment:
      - JENKINS_OPTS="--httpPort=8080"
    volumes:
      - jenkins_home:/var/jenkins_home

  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    networks:
      - alertmanager_network
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    networks:
      - prometheus-grafana_monitoring
    ports:
      - "9100:9100"

networks:
  prometheus-grafana_monitoring:
    driver: bridge
  jenkins_monitoring:
    driver: bridge
  alertmanager_network:
    driver: bridge

volumes:
  jenkins_home:
```

### Step 2: Configure Prometheus

Prometheus configuration (`prometheus.yml`) defines the scrape targets for monitoring Jenkins, the VM host, and other services.

Example configuration for Prometheus:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'jenkins'
    static_configs:
      - targets: ['jenkins:8080']
    metrics_path: '/prometheus'
    scheme: 'http'

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

Ensure that your Prometheus service is correctly configured to scrape the desired endpoints.

### Step 3: Configure Grafana

Grafana is configured to connect to Prometheus as a data source.

Example Grafana configuration for data source (in `grafana/provisioning/datasources/datasource.yaml`):

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

### Step 4: Set Up Alertmanager for Slack Notifications

1. **Install Alertmanager** in the `docker-compose.yml` file.
2. Create an `alertmanager.yml` file to configure Alertmanager for Slack notifications:

Example `alertmanager.yml` for Slack alerts:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 3h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/your/webhook/url'
        channel: '#your-channel'
        send_resolved: true
```

3. Replace `'https://hooks.slack.com/services/your/webhook/url'` with your actual Slack Webhook URL.

### Step 5: Jenkins Prometheus Exporter

In order to scrape Jenkins metrics, use the Prometheus metrics plugin in Jenkins. The plugin exposes Jenkins metrics at the `/prometheus` endpoint.

1. Install the **Prometheus Metrics Plugin** in Jenkins.
2. Access the metrics at `http://<jenkins_host>:8080/prometheus`.

### Step 6: Starting the Setup

After configuring the services, you can start everything using Docker Compose:

```bash
docker-compose up -d
```

This will start the Prometheus, Grafana, Jenkins, Alertmanager, and Node Exporter containers. You can now access:

- **Grafana**: `http://localhost:3000` (default username: `admin`, password: `admin`)
- **Prometheus**: `http://localhost:9090`
- **Jenkins**: `http://localhost:8080`
- **Alertmanager**: `http://localhost:9093`

## Dashboards

Once everything is up and running, you can configure Grafana dashboards to visualize Jenkins, VM host, and other system metrics. Example dashboards include:

- **Jenkins Dashboard**: Displays Jenkins job statistics, build health, and performance metrics.
- **Node Exporter Dashboard**: Displays system health metrics such as CPU usage, memory usage, disk IO, and network statistics.
- **VM Host Monitoring Dashboard**: Displays metrics of the VM itself, including CPU, memory, and disk usage.

You can import or create custom dashboards in Grafana using the UI.

## Alerting

Prometheus will trigger alerts based on defined rules. For example, if Jenkins or the VM host's CPU usage goes above a certain threshold, **Alertmanager** will notify the specified Slack channel.

Example Prometheus alert rule (`prometheus.rules`):

```yaml
groups:
  - name: jenkins_alerts
    rules:
      - alert: JenkinsDown
        expr: up{job="jenkins"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Jenkins is down"
          description: "Jenkins has been down for more than 5 minutes."

  - name: vm_host_alerts
    rules:
      - alert: HighCPUUsage
        expr: node_cpu_seconds_total{mode="idle"} < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage on VM host"
          description: "CPU usage on the VM host has been over 90% for the last 5 minutes."
```

## Troubleshooting

If you encounter issues, here are some steps to troubleshoot:

- Ensure that all containers are up by running `docker ps`.
- Check container logs for errors: `docker logs <container_name>`.
- Verify network configurations by running `docker network ls`.

## Conclusion

This setup provides a powerful monitoring and visualization solution for Jenkins, the VM host, and other system metrics using Prometheus and Grafana. The integration with **Alertmanager** ensures that any critical alerts are promptly sent to Slack for immediate action.
