# Kube-State-Metrics Configs

This repository contains configurations for **kube-state-metrics**, along with instructions on setting up a monitoring stack using **Prometheus** and **Grafana** on a Kubernetes cluster. 

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Provisioning an EC2 Instance for Kubeadm](#provisioning-an-ec2-instance-for-kubeadm)
- [Deploying Prometheus](#deploying-prometheus)
- [Visualizing Prometheus Dashboard](#visualizing-prometheus-dashboard)
- [Deploying Grafana](#deploying-grafana)
- [Importing and Configuring a Dashboard](#importing-and-configuring-a-dashboard)
- [Deploying an Application and Observing Metrics](#deploying-an-application-and-observing-metrics)
- [Contributing](#contributing)
- [License](#license)

## Overview
This project sets up a Kubernetes monitoring stack using **Prometheus**, **kube-state-metrics**, and **Grafana**. The guide walks through provisioning an EC2 instance, setting up Kubernetes with `kubeadm`, deploying Prometheus and Grafana, and visualizing application metrics.

## Prerequisites
Before you begin, ensure you have:
- An AWS account with permissions to create EC2 instances.
- An SSH key pair for secure access to the EC2 instance.
- `kubectl`, `kubeadm`, and `helm` installed on your local machine.
- A configured AWS CLI.

## Provisioning an EC2 Instance for Kubeadm
1. Launch an EC2 instance (Ubuntu 22.04 recommended) with at least **2 vCPUs** and **4GB RAM**.
2. SSH into the instance:
   ```sh
   ssh -i your-key.pem ubuntu@your-instance-ip
   ```
3. Install Docker and Kubernetes components:
   ```sh
   sudo apt update && sudo apt install -y docker.io
   sudo systemctl enable --now docker
   sudo apt install -y kubelet kubeadm kubectl
   ```
4. Initialize the Kubernetes cluster:
   ```sh
   sudo kubeadm init --pod-network-cidr=192.168.0.0/16
   ```
5. Configure `kubectl` for the `ubuntu` user:
   ```sh
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

## Deploying Prometheus
1. Add the Helm repository and update:
   ```sh
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```
2. Install Prometheus using Helm:
   ```sh
   helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
   ```

## Visualizing Prometheus Dashboard
1. Port-forward Prometheus UI:
   ```sh
   kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
   ```
2. Access Prometheus at [http://localhost:9090](http://localhost:9090).

## Deploying Grafana
1. Get Grafana admin credentials:
   ```sh
   kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
   ```
2. Port-forward Grafana:
   ```sh
   kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
   ```
3. Access Grafana at [http://localhost:3000](http://localhost:3000) and log in with `admin` and the retrieved password.

## Importing and Configuring a Dashboard
1. Go to Grafana > Dashboards > Import.
2. Use the **Dashboard ID** for `kube-state-metrics`, such as **315** from [Grafana Dashboards](https://grafana.com/grafana/dashboards/315).
3. Select Prometheus as the data source and import the dashboard.

## Deploying an Application and Observing Metrics
1. Deploy a sample Nginx application:
   ```sh
   kubectl create deployment nginx --image=nginx
   ```
2. Expose the deployment:
   ```sh
   kubectl expose deployment nginx --port=80 --type=NodePort
   ```
3. Verify that Grafana displays metrics for your application.

---
**Author:** Lisa Eileen  
**GitHub:** [LisaEileen](https://github.com/Lisaeileen)
