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


```markdown
### 🌐 Services Utilized
*   **Amazon VPC:** Isolated networking container (containing Public Subnets, Security Groups).
*   **Amazon EC2:** Compute layer hosting the Gunicorn/Flask application server.
*   **Elastic Load Balancing (ALB):** Public ingress traffic distribution and health monitoring.
*   **Amazon CloudWatch:** Telemetry collection, logging (Logs Agent), performance metrics, and automated alerting infrastructure.

---

## ⚖️ AWS Well-Architected Review

This design was prioritized against two critical pillars of the AWS Well-Architected Framework:

### Pillar 1: Operational Excellence (Monitor and Automate)
*   **Best Practice:** Centralize logs and use actionable metrics for automated alerting.
*   **Implementation:** Configured the CloudWatch Logs Agent on EC2 to stream application and error logs to a persistent Log Group. Established CloudWatch Alarms with 1-minute precision to generate high-severity alerts (`In Alarm`) when application latency exceeds performance baselines or when targets go completely offline (`Missing Data`). This ensures a measurable, proactive response to system performance.

### Pillar 2: Cost Optimization (Match Supply and Demand)
*   **Best Practice:** Select the right resources and manage dynamic demand to minimize waste.
*   **Implementation:** Used a single compute node (t3.micro) rather than a clustered Auto Scaling Group for a development workload. To minimize post-testing overhead, the Application Load Balancer was identified as the primary cost driver (~$18–$24/month idle cost) and was deleted immediately following artifact collection. The underlying EC2 node was **Stopped** rather than terminated, dropping its compute charges to zero while safely preserving the application's code and state (EBS data) for future validation at minimal storage cost.

---

## 📸 Core Artifacts & Proof of Engineering

### 1. Centralized Log Streams
To ensure complete visibility into the health of the application, I configured the CloudWatch logs agent to stream application logs directly from the EC2 instance into a centralized repository.

artifact_03_cloudwatch_log_stream.png 

### 2. High-Latency Incident Trigger & Alarm Breach
To test the reliability of the monitoring framework, traffic loops were simulated against the application. By stopping the backend web server process (`pkill gunicorn`), I intentionally triggered an operational failure. 

To ensure the system flagged the total loss of the application backend, the alarm’s missing data treatment was optimized to **"Treat missing data as bad (breaching)"**, instantly forcing the alarm into a critical state.

_04_cloudwatch_alarm_breach.png

---

## 🛠️ Incident Response & Root Cause Analysis (RCA)

When a system failure occurred, the following technical troubleshooting runbook was executed to restore the service:

1.  **Identify State:** CloudWatch Alarm triggered an `Insufficient_Data` / `In_Alarm` state change due to a non-responsive target group backend.
2.  **SSH / Session Re-entry:** Established a secure connection to the instance. Discovered a standard terminal timeout had occurred, requiring a clean shell initialization.
3.  **Directory Diagnostics:** Running standard global pathing threw a `ModuleNotFoundError: No module named 'app'`. Used `find / -name "app.py" 2>/dev/null` to locate the isolated root application path at `/app`.
4.  **Service Restoration:** Navigated to the root module directory and successfully re-initialized the production server using:
    ```bash
    cd /app && sudo gunicorn --bind 0.0.0.0:80 app:app
    ```
5.  **Recovery Verification:** Traffic loops successfully resumed handling requests, and CloudWatch metrics returned to a nominal green **OK** baseline.
