
# Prometheus, Grafana, Jenkins Integration for Host and Jenkins Monitoring

This project provides an automated monitoring infrastructure setup using Prometheus and Grafana with Jenkins CI/CD integration.
It includes automatic deployment, Jenkins monitoring, alert management, and Slack notifications.
## Project Structure
---
Project Overview
This solution automates the setup and monitoring of infrastructure components with a focus on:

Automated deployment of the entire monitoring stack
Jenkins monitoring using Prometheus and CloudBees disk usage plugins
Real-time alerts delivered to Slack
Configuration validation through CI/CD pipeline

The project is divided into three main components:

    Vagrant:

        Provisions the Linux VM to serve as the host environment.
        Ensures a consistent, repeatable environment for Jenkins, Prometheus, and Grafana.

    Jenkins:

        Manages your CI/CD pipeline using Docker.
        Automates the setup of monitoring tools (Prometheus, Grafana, AlertManager, Node Exporter).

    Prometheus-Grafana:

        Contains configurations for Prometheus, Grafana, AlertManager, and dashboards.
        Provides monitoring, visualization, and alerting.

---
## Projet structure:

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
# ⚙️ Installation & Setup
## Prerequisites

Before setting up, ensure that you have the following installed:
- **Vgrant & Virtualbox**: to configure and run the vm (linux-ubuntu:22.04 image)
- **Docker**: To run containers for Prometheus, Grafana, Jenkins, Alertmanager and Node exporter.
- **Docker Compose**: For easier management of multi-container Docker applications.
- **Slack Webhook URL**: For Slack notifications when an alert is triggered.
- **Jenkins runner on the vm**: For run and automate stages

🚀 Steps to Get Started

 Clone the repository:

    git clone https://github.com/MatanKaufman1/sre-sec-project.git
    cd your-repo-name

Start the VM with Vagrant:

    cd vagrant
    vagrant up

SSH into the VM:

    vagrant ssh

create an jenkins directory:
    
    mkdir jenkins

Copy the docker-compose.yml file from the shared folder:

Assuming your project is synced into /vagrant (Vagrant’s default):

    cp /vagrant/jenkins/docker-compose.yml ~/jenkins/

Run Jenkins inside the VM:

    cd ~/jenkins
    docker-compose up -d
This starts a Jenkins server at http://localhost:8080.

Access Jenkins UI

    Navigate to http://localhost:8080

    Complete the initial setup (admin password, plugins, etc.)

    Create or trigger the pipeline job using sre.Jenkinsfile

The pipeline will:

    Clone this repository

    Create a runner/workspace/ directory

    Copy the contents of prometheus-grafana/ into it

    Automatically bring up Prometheus, Grafana, Node Exporter, and Alertmanager using Docker Compose

## 📊 Creating Dashboards in Grafana:
You can use a pre-built Node Exporter Full dashboard from Grafana's community repository.
This dashboard provides comprehensive system monitoring, including CPU, memory, disk, and network usage metrics.
To use this dashboard, import it into Grafana by following these steps:

    Visit the Node Exporter Full Dashboard on Grafana.com.
    
Download the file and paste it:

        prometheus-grafana/grafana/provisioning/dashboard/ folder 

This method ensures that users can take advantage of an established,
ready-to-use dashboard while also following your project's installation instructions.


