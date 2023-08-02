# Ansible Playbook for Deploying Node Exporter, Prometheus, and Grafana

## Introduction

This repository contains an Ansible playbook to automate the deployment of Node Exporter on multiple hosts. Additionally, it sets up Prometheus and Grafana on your local machine to monitor the metrics collected by the Node Exporter. The playbook is designed to provide easy setup and configuration, enabling users to quickly start monitoring their systems.

## Prerequisites

Before running the Ansible playbook, ensure you have the following software installed on your local machine:

- Python3 and pip3
- Ansible
- Prometheus (installed via `sudo snap install prometheus`)
- Grafana (installed via `sudo snap install grafana`)

## Installation

1. Install necessary packages:

```bash
sudo apt-get install -y gcc python3-dev libkrb5-dev
sudo apt-get install python3-pip -y
pip3 install --upgrade pip
pip3 install pywinrm
sudo apt install ansible -y
```
2. Download Node Exporter: 

Download the latest release of Node Exporter from the Prometheus GitHub repository:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

##  Project Structure
Create the project directory and organize the files as follows:

## Inventory Configuration
In the `inventory.ini` file, specify the IP addresses, username, and password for the target systems where Node Exporter is to be deployed.

```ini
[linux_hosts]
<IP Address of Target System> ansible_user=<username> ansible_connection=winrm ansible_winrm_server_cert_validation=ignore ansible_password=<password>
```
Replace `your_username` and `your_password` with the appropriate credentials for each target system.

## Ansible Configuration
Create an `ansible.cfg` file under the `ansibleconfig/` directory with the following content:

```cfg
[defaults]
transport = basic
```

This configuration sets the inventory file to be used in the parent directory (`../inventory.ini`).

## Ansible Playbook
The main `playbook.yml` contains the tasks to deploy Node Exporter on the target systems. It also includes tasks for setting up Prometheus and Grafana on the local machine.

```yml
---
- name: Install Node Exporter On RujulLinux
  hosts: linux_hosts
  gather_facts: yes
  tasks:
    - name: Make New Directory
      file:
        path: "/home/raven/NodeExporterFromVM"
        state: directory
      tags:
        - prometheus

    - name: Copy NodeExporter File
      copy:
        src: "/home/raven/luganodes/node_exporter-1.6.1.linux-amd64.tar.gz"
        dest: "/home/raven/NodeExporterFromVM"

    - name: Unarchive NodeExporter File
      unarchive:
        src: "/home/raven/NodeExporterFromVM/node_exporter-1.6.1.linux-amd64.tar.gz"
        dest: "/home/raven/NodeExporterFromVM/"
        remote_src: yes
        creates: "/home/raven/NodeExporterFromVM/node_exporter-1.6.1.linux-amd64/"
```
## Prometheus Configuration
Prometheus is installed via snap, so the configuration file `prometheus.yml` can be placed in the default location (`/var/snap/prometheus/common/prometheus.yml`). The contents of prometheus.yml will include the necessary targets for scraping, including the Node Exporter on each target system:

```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node_exporter"
    scrape_interval: 5s
    static_configs:
      - targets: ["localhost:9100"]
        labels:
          group: "prometheus"
      - targets: ["localhost:8080"]
        labels:
          group: "production"
```

## Grafana Configuration
After installing Grafana, open the Grafana web interface at [http://localhost:3000](http://localhost:3000). Use the default username and password (both set to "admin") to log in. Once logged in, configure data sources to connect to Prometheus. Click on "Configuration" in the left sidebar and select "Data Sources." Then, click on "Add data source," choose "Prometheus," and configure the URL to point to your Prometheus server running on `localhost:9090`. Save the configuration.

## Bonus: Setup Alerting via Telegram
To achieve this, follow these steps:

1. Install and configure the Grafana Telegram Bot plugin.
2. Configure the Telegram notification channel in Grafana.
3. Trigger alerts by simulating threshold breaches in the Node Exporter metrics.

## Video Demonstration
The video demonstration showcases the Grafana dashboard for Node Exporter metrics and the front-ends of both Prometheus (showing all targets) and Grafana (showing alerts if the bonus task is done). The video link [here](https://drive.google.com/file/d/1aMpJQq1RIlrKU7NacMPxTmf1dnJNjCUp/view?usp=sharing)

## Conclusion
With this Ansible playbook, deploying Node Exporter on multiple hosts and setting up Prometheus and Grafana becomes a straightforward process. The included configurations and demonstrations provide a solid foundation for monitoring systems effectively.
