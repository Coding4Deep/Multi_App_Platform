<div align="center">

#  **DevOps Multi Application Dashboard**

[![CI/CD](https://img.shields.io/badge/Jenkins-CI%2FCD-red?logo=jenkins)](https://www.jenkins.io)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-blue?logo=kubernetes)](https://kubernetes.io)
[![Docker](https://img.shields.io/badge/Docker-Container-blue?logo=docker)](https://www.docker.com)
[![React](https://img.shields.io/badge/Frontend-React-61DAFB?logo=react)](https://reactjs.org)
[![MongoDB](https://img.shields.io/badge/Database-MongoDB-4EA94B?logo=mongodb)](https://www.mongodb.com)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-336791?logo=postgresql)](https://www.postgresql.org)
[![RabbitMQ](https://img.shields.io/badge/Messaging-RabbitMQ-FF6600?logo=rabbitmq)](https://www.rabbitmq.com)
[![Memcached](https://img.shields.io/badge/Cache-Memcached-000000?logo=memcached)](https://memcached.org)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?logo=prometheus)](https://prometheus.io)
[![Grafana](https://img.shields.io/badge/Visualization-Grafana-F46800?logo=grafana)](https://grafana.com)
[![License](https://img.shields.io/github/license/your-org/devops-dashboard)](./LICENSE)

</div>

---

## 🌐 Project Overview

The **DevOps Application Dashboard** is a unified interface for accessing and monitoring multiple microservices deployed on Kubernetes (EKS). It provides developers, DevOps engineers, and testers a **single-click gateway** to all services along with **real-time health monitoring**.

> Built with end-to-end CI/CD pipelines, full Kubernetes deployment automation, and observability stack – it is your DevOps command center.

---

## 🧭 Key Features

- 🔄 **Central Dashboard** for navigating all deployed services
- ⚙️ Fully automated **CI/CD pipeline with Jenkins**
- 🐳 **Dockerized Microservices** with push to private/public registries
- ☸️ **Kubernetes (EKS)** deployment with autoscaling and HPA
- 🛡️ Uses **Secrets** and **ConfigMaps** for secure configuration
- 📊 Real-time Monitoring via **Prometheus + Grafana**
- 💾 Integrated with **MongoDB**, **PostgreSQL**, **RabbitMQ**, and **Memcached**

---

## 🏗️ Project Architecture

```mermaid
graph TD
  A[Developer Pushes Code to GitHub] --> B[Jenkins CI/CD Pipeline]
  B --> C[Test, Build, Package]
  C --> D[Push Artifact to S3/Nexus/Github]
  C --> E[Dockerize & Push Image to Registry]
  E --> F["Deploy to Kubernetes (EKS)"]
  F --> G["Applications + RabbitMQ + DBs + Cache"]
  G --> H["Ingress + Autoscaling + Secrets"]
  H --> I["Dashboard Exposes All Apps"]
  I --> J["User Access & Health Monitoring"]

