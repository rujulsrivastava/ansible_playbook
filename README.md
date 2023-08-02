# ansible_playbook
1. Installations
﻿sudo apt-get install -y gcc python3-dev libkrb5-dev && \ sudo apt-get install python3-pip -y && \
pip3 install --upgrade pip && \
pip3 install pywinrm && \
sudo apt install ansible -y
check with ansible --version

get node_exporter : wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

2. Create project directory and structure
mkdir luganodes
write inventory.ini, playbook.yml
inventory.ini
[linux_hosts]
192.168.207.130 ansible_user=raven ansible_connection=winrm ansible_winrm_server_cert_validation=ignore ansible_password=Winner@1
playbook.yml
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

mkdir ansibleconfig
ansible.cfg :

install prometheus: sudo snap install prometheus
check: sudo systemctl status snap.prometheus.prometheus
http://localhost:9090


Install grafana:
wget -q -O - https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/grafana-archive-keyring.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana

sudo snap start grafana
check: sudo snap services grafana


View via http://localhost:3000. Initially, the default username and password are both set to "admin." 
Once logged into Grafana, you'll need to configure data sources to connect to Prometheus. Click on "Configuration" in the left sidebar and select "Data Sources." Then, click on "Add data source," choose "Prometheus," and configure the URL to point to your Prometheus server running on localhost:9090. Save the configuration.

