
# Prometheus, Grafana, Jenkins Integration for Host and Jenkins Monitoring

This project provides an automated monitoring infrastructure setup using **Prometheus** and **Grafana** with **Jenkins CI/CD** integration. It includes automatic deployment, Jenkins monitoring, alert management, and Slack notifications.

## Project Structure

### Project Overview

This solution automates the setup and monitoring of infrastructure components with a focus on:

- Automated deployment of the entire monitoring stack
- Jenkins monitoring using Prometheus and CloudBees disk usage plugins
- Real-time alerts delivered to Slack
- Configuration validation through the CI/CD pipeline

The project is divided into three main components:

1. **Vagrant:**
   - Provisions the Linux VM to serve as the host environment.
   - Ensures a consistent, repeatable environment for Jenkins, Prometheus, and Grafana.

2. **Jenkins:**
   - Manages your CI/CD pipeline using Docker.
   - Automates the setup of monitoring tools (Prometheus, Grafana, AlertManager, Node Exporter).

3. **Prometheus-Grafana:**
   - Contains configurations for Prometheus, Grafana, AlertManager, and dashboards.
   - Provides monitoring, visualization, and alerting.

### Project Structure:

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

## ⚙️ Installation & Setup

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

### 📊 Creating Dashboards in Grafana:

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

This method ensures that users can take advantage of an established, ready-to-use dashboard while also following your project's installation instructions.

---
