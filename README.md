# Prometheus & Grafana with Jenkins Integration for Automated Monitoring

This project provides an automated monitoring infrastructure setup using Prometheus and Grafana with Jenkins CI/CD integration. It includes automatic deployment, Jenkins monitoring, alert management, and Slack notifications.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Project Structure](#project-structure)
3. [Pipeline Flow Overview](#pipeline-flow-overview)
4. [Installation & Setup](#installation--setup)
5. [Creating Dashboards in Grafana](#creating-dashboards-in-grafana)
6. [Monitoring Components](#monitoring-components)
7. [Alert Rules Configuration](#alert-rules-configuration)
8. [Data Collection Flow](#data-collection-flow)
9. [Contact Information](#Contact-Information)

## Project Overview

This solution automates the setup and monitoring of infrastructure components with a focus on:

- Automated deployment of the entire monitoring stack
- Jenkins monitoring using Prometheus and CloudBees disk usage plugins
- Real-time alerts delivered to Slack
- Configuration validation through the CI/CD pipeline

The project is divided into three main components:

- **Vagrant**:
  - Provisions the Linux VM to serve as the host environment.
  - Ensures a consistent, repeatable environment for Jenkins, Prometheus, and Grafana.

- **Jenkins**:
  - Manages your CI/CD pipeline using Docker.
  - Automates the setup of monitoring tools (Prometheus, Grafana, AlertManager, Node Exporter).

- **Prometheus-Grafana**:
  - Contains configurations for Prometheus, Grafana, AlertManager, and dashboards.
  - Provides monitoring, visualization, and alerting.

## Project Structure



```
├── jenkins
│   ├── docker-compose.yml
│   └── sre.Jenkinsfile
├── prometheus-grafana
│   ├── alertmanager
│   │   └── alertmanager.yml
│   ├── docker-compose.yml
│   ├── grafana
│   │   └── provisioning
│   └── prometheus
│       ├── alert_rules.yml
│       └── prometheus.yml
├── README.md
└── vagrant
    └── Vagrantfile
```

## Pipeline Flow Overview

This project includes a CI/CD pipeline managed by Jenkins to automate the setup and monitoring of infrastructure components. The pipeline is designed to automate several tasks, including YAML linting, security checks, container setup, and deployment of Prometheus, Grafana, and Node Exporter. Below is the flow of the pipeline:

 Install YAML Linter:

   In this stage, the pipeline installs yamllint, a tool used to lint YAML files in the repository. This ensures that the YAML configurations are properly formatted and follow best practices.

 Check YAML Files:

   After installing yamllint, the pipeline proceeds to lint all the YAML files in the repository. This step helps ensure that no syntax errors exist in configuration files like Docker Compose and Kubernetes manifests.

 Install Checkov:

   The pipeline installs Checkov, a static code analysis tool for Terraform, Kubernetes, and other cloud-related configuration files. Checkov helps identify security misconfigurations in infrastructure-as-code files.

 Run Checkov:

   In this stage, Checkov is executed to analyze all the files in the repository and output a security report. If any issues or misconfigurations are found, the pipeline will provide a detailed summary.

 Install Prometheus, Grafana, and Node Exporter:

   This step brings up the monitoring stack, including Prometheus, Grafana, and Node Exporter, using Docker Compose. It ensures that the monitoring services are up and running, collecting metrics from the Jenkins          instance and the VM host.

 Check Running Containers:

   The pipeline verifies that all the necessary containers (Prometheus, Grafana, Node Exporter) are running by executing the docker ps command. This step ensures that the services are properly started and available.

Post-Pipeline Notifications

After the pipeline execution:

 Success: If the pipeline completes successfully, a notification is sent to the Slack channel (#all-sre) with a "success" status.

 Failure: If any step fails, a "failure" notification is sent to the same Slack channel, indicating the issue in the pipeline.

---

##  Installation & Setup

### Prerequisites

Before setting up, ensure that you have the following installed:

- **Vagrant & Virtualbox**: To configure and run the VM (Linux-Ubuntu:22.04 image).
- **Docker**: To run containers for Prometheus, Grafana, Jenkins, AlertManager, and Node Exporter.
- **Docker Compose**: For easier management of multi-container Docker applications.
- **Slack Webhook URL**: For Slack notifications when an alert is triggered.
- **Jenkins runner on the VM**: For running and automating pipeline stages.

### 🚀 Steps to Get Started

1. **Clone the repository:**

   ```bash
   git clone https://github.com/MatanKaufman1/sre-sec-project.git
   cd sre-sec-project
   ```

2. **Start the VM with Vagrant:**

   ```bash
   cd vagrant
   vagrant up
   ```

3. **SSH into the VM:**

   ```bash
   vagrant ssh
   ```

4. **Create the Jenkins directory:**

   ```bash
   mkdir ~/jenkins
   ```

5. **Copy the docker-compose.yml file from the shared folder:**

   Assuming your project is synced into `/vagrant` (Vagrant’s default):

   ```bash
   cp /vagrant/jenkins/docker-compose.yml ~/jenkins/
   ```

6. **Run Jenkins inside the VM:**

   ```bash
   cd ~/jenkins
   docker-compose up -d
   ```

   This starts a Jenkins server at http://localhost:8080.

### Access Jenkins UI

- Navigate to http://localhost:8080
- Complete the initial setup (admin password, plugins, etc.)
- Create or trigger the pipeline job using `sre.Jenkinsfile`

The pipeline will:

- Clone this repository
- Create a `runner/workspace/` directory
- Copy the contents of `prometheus-grafana/` into it
- Automatically bring up Prometheus, Grafana, Node Exporter, and AlertManager using Docker Compose

---

### Creating Dashboards in Grafana:

You can use a pre-built **Node Exporter Full** dashboard from Grafana's community repository. This dashboard provides comprehensive system monitoring, including CPU, memory, disk, and network usage metrics.

To use this dashboard, import it into Grafana by following these steps:

1. Visit the [Node Exporter Full Dashboard on Grafana.com](https://grafana.com/grafana/dashboards/1860-node-exporter-full/).
2. Download the JSON file for the dashboard.
3. Copy the downloaded file into the `prometheus-grafana/grafana/provisioning/dashboards/` folder.

   ```bash
   cp /path/to/downloaded/dashboard.json prometheus-grafana/grafana/provisioning/dashboards/
   ```

4. Restart Grafana to load the new dashboard:

   ```bash
   docker restart grafana
   ```
---
## Monitoring Components

This project collects metrics from two primary sources: the host system (via Node Exporter) and Jenkins. These metrics are stored in Prometheus and visualized in Grafana dashboards.
Host Monitoring (Node Exporter)

Node Exporter collects system-level metrics from the Linux host, including:
CPU Metrics

    CPU usage by mode (user, system, idle, etc.)

    Load averages

    Process counts and states

Memory Metrics

    Total and available memory

    Swap utilization

    Memory allocation details

    Cache usage

Disk Metrics

    Disk space (used/free)

    Disk I/O operations

    Latency and inode usage

Network Metrics

    Traffic (bytes in/out)

    Packet rates

    Errors and drops

    Connection states

System Metrics

    Uptime

    Load

    File descriptor usage

    Available entropy

🛠️ Jenkins Monitoring

Jenkins metrics are collected using the Prometheus plugin for Jenkins and include:
Build Metrics

    Build durations

    Build result rates (success/failure)

    Build queue time

    Number of builds per status

Jenkins Health

    Executor status

    Plugin health

    Resource usage by Jenkins

CloudBees Disk Usage (via plugin)

    Workspace disk usage

    Artifact disk usage

    Total Jenkins disk consumption

Alert Rules Configuration

The monitoring stack includes pre-configured alert rules in Prometheus:
Alert Name	Condition	Severity
NodeExporterDown	Node Exporter is unreachable for 1 minute	Critical
JenkinsJobFailure	Any Jenkins job fails	High
HighCPU	CPU usage > 90% for more than 1 minute	Warning

All alerts are routed to the #all-sre Slack channel using AlertManager.
Data Collection Flow

Here’s how the monitoring pipeline works:

    Node Exporter (port 9100) collects host metrics every 10s.

    Jenkins Prometheus plugin exposes metrics at /prometheus.

    Prometheus (port 9090) scrapes data from Node Exporter & Jenkins.

    AlertManager (port 9093) handles alerts and sends Slack notifications.

    Grafana (port 3000) displays real-time dashboards.

All services run in Docker containers within a shared Docker network, ensuring portability and consistency across environments.

---
## Contact Information

    Maintainer: Matan Kaufman
    Email: matankaufman1@gmail.com

