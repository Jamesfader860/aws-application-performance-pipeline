# aws-application-performance-pipeline
A production-ready AWS monitoring architecture featuring a Python Flask/Gunicorn backend behind an ALB, with centralized CloudWatch log streaming, automated latency breach alerting, and post-testing cost optimization

# 📊 Distributed Application Performance Monitoring & Incident Response Pipeline

## 🚀 Project Overview
This project demonstrates the implementation of a highly available, monitored web application architecture on AWS. By deploying a Python Flask application behind an **Application Load Balancer (ALB)**, I established a robust monitoring pipeline using **Amazon CloudWatch** to track system performance, centralize application logs, and automate real-time incident alerting for latency anomalies and system failures.

Additionally, this project highlights hands-on **Chaos Engineering** principles by intentionally disrupting application availability to validate the alerting infrastructure and executing precise **Cost Optimization** strategies to decommission high-billing cloud resources post-testing.

---

## 🏗️ System Architecture & Blueprint
The infrastructure is engineered utilizing a decoupled, monitored approach:
*   **Compute:** Amazon EC2 instance hosting a WSGI-compliant Flask application served via **Gunicorn** at a root directory configuration (`/app`).
*   **Traffic Management:** An AWS Application Load Balancer (ALB) acting as the public-facing ingress point, distributing traffic and monitoring target health.
*   **Telemetry & Observability:** 
    *   **CloudWatch Logs Agent:** Configured to stream live Gunicorn and system application logs to a centralized log group.
    *   **CloudWatch Alarms:** Set up to evaluate performance metrics (Latency / Missing Data) using a strict 1-minute resolution window.

```text
       ┌────────────────────────────────────────────────────────────────────────┐
       │                       AMAZON CLOUDWATCH (TELEMETRY)                     │
       │                                                                        │
       │    ┌──────────────────┐      ┌────────────────────────────────────┐    │
       │    │  CloudWatch Logs │◄─────┼─ [Stream] Gunicorn/Flask App Logs │    │
       │    └──────────────────┘      └────────────────────────────────────┘    │
       │             ▲                                                          │
       │             │ (1-Min Resolution)                                       │
       │    ┌──────────────────┐                                                │
       │    │ CloudWatch Alarm │◄─── [Breach Trigger: Treat Missing Data As Bad]│
       │    └──────────────────┘                                                │
       └─────────────▲──────────────────────────────────────────────────────────┘
                     │
                     │ (Tracks Target Health & Latency Metrics)
  ===================│============================================================
                     │                 AMAZON VPC
                     │
         [Public Internet Ingress]
                     │
                     ▼
         ┌───────────────────────┐
         │ Application Load      │
         │ Balancer (ALB)        │
         └───────────┬───────────┘
                     │
                     │ (Target Group Routing: Port 80)
                     ▼
         ┌───────────────────────┐
         │  Amazon EC2 Instance  │
         │  (t3.micro Node)      │
         │ ┌───────────────────┐ │
         │ │ Production Server │ │
         │ │ Gunicorn / WSGI   │ │
         │ └─────────┬─────────┘ │
         │           ▼           │
         │ ┌───────────────────┐ │
         │ │ Flask App         │ │
         │ │ Path: /app/app.py │ │
         │ └───────────────────┘ │
         └───────────────────────┘
  ================================================================================
  
       ┌────────────────────────────────────────────────────────────────────────┐
       │                💸 POST-PROJECT COST OPTIMIZATION PHASE                 │
       │                                                                        │
       │  1. [DELETED]  Application Load Balancer ──► Saved ~$20.00/month idle  │
       │  2. [STOPPED]  Amazon EC2 Instance       ──► Preserved code at $0/hr  │
       │  3. [RELEASED] Idle Elastic IP Addresses ──► Eliminated unattached fees│
       └────────────────────────────────────────────────────────────────────────┘
