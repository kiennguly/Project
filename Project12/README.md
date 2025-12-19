\# ECS on Fargate: Baseline (On-Demand) + Burst (Fargate Spot)



This repository documents a reference architecture for running a web application on \*\*Amazon ECS with AWS Fargate\*\* that must be \*\*highly available (24/7)\*\* while remaining \*\*cost-effective\*\* during \*\*short-lived traffic bursts\*\*.



The core design principle is \*\*capacity separation\*\*:

\- \*\*Baseline capacity\*\* runs on \*\*Fargate (On-Demand)\*\* to guarantee uptime.

\- \*\*Burst capacity\*\* runs on \*\*Fargate Spot\*\* to reduce cost during spikes.



---



\## Design Goals



\- \*\*24/7 Availability (High Availability)\*\*  

&nbsp; Baseline tasks must remain stable and non-interruptible.



\- \*\*Elastic Scaling for Short Bursts\*\*  

&nbsp; The service must scale out quickly when traffic spikes and scale back in when traffic normalizes.



\- \*\*Cost Optimization\*\*  

&nbsp; Use lower-cost capacity only for temporary scaling rather than for steady-state workloads.



---



\## Architecture Diagram (Mermaid)





\# ECS Fargate Architecture: Baseline \& Burst Optimization



\## 🏗️ Components and Responsibilities



\### 1. Application Load Balancer (ALB)

\* \*\*Public Entry Point:\*\* Acts as the public entry point for incoming HTTPS traffic.

\* \*\*Distribution:\*\* Distributes requests to healthy ECS tasks via a target group using IP targets for Fargate.

\* \*\*Resilience:\*\* Improves availability by automatically routing traffic around unhealthy tasks.



\### 2. ECS Service (Service Scheduler)

\* \*\*Lifecycle Management:\*\* Maintains the desired task count at all times.

\* \*\*Auto-Recovery:\*\* Automatically replaces failed tasks to ensure service continuity.

\* \*\*Placement Logic:\*\* Works with capacity providers to decide where tasks are placed based on defined strategies.



\### 3. Baseline Tasks — Fargate On-Demand

\* \*\*Steady-State:\*\* Always running to handle the core application load.

\* \*\*Reliability:\*\* Non-interruptible capacity used to satisfy 24/7 and high availability requirements.

\* \*\*Performance:\*\* Handles normal traffic consistently without risk of reclamation.



\### 4. Burst Tasks — Fargate Spot

\* \*\*Elastic Capacity:\*\* Launched only when traffic spikes occur.

\* \*\*Cost Efficiency:\*\* Significantly cheaper than On-Demand (up to 70-90% discount).

\* \*\*Fault Tolerance:\*\* Interruption-tolerant by design; used exclusively for temporary scale-out capacity.



\### 5. CloudWatch + Service Auto Scaling

\* \*\*Metrics Collection:\*\* CloudWatch monitors key service metrics, including:

&nbsp;   \* ALB request count per target.

&nbsp;   \* Target response time.

&nbsp;   \* ECS CPU and Memory utilization.

\* \*\*Dynamic Scaling:\*\* Auto Scaling policies increase or decrease the desired task count based on real-time demand.



\### 6. Capacity Provider Strategy (Base + Weight)

\* \*\*Base:\*\* Ensures a minimum number of tasks always run on Fargate On-Demand (baseline).

\* \*\*Weight:\*\* Directs additional tasks (scale-out) to Fargate Spot (burst).

\* \*\*Best Practice:\*\* This is the recommended way to mix Fargate and Fargate Spot in a managed, repeatable manner.



---



\## 🚦 Request Flow

1\. \*\*Initiation:\*\* Users send HTTPS requests to the Application Load Balancer.

2\. \*\*Routing:\*\* ALB forwards requests to the ECS Service target group.

3\. \*\*Normal Operations:\*\* Baseline On-Demand tasks serve steady-state traffic.

4\. \*\*Trigger:\*\* When traffic spikes, CloudWatch metrics rise and trigger ECS Service Auto Scaling.

5\. \*\*Expansion:\*\* Auto Scaling increases the desired task count.

6\. \*\*Placement:\*\* The Capacity Provider strategy places new tasks onto Fargate Spot (burst capacity).

7\. \*\*Cool Down:\*\* When traffic drops, Auto Scaling scales in, and burst tasks are terminated first.







---



\## ⚠️ Spot Interruption Behavior

Fargate Spot tasks can be interrupted by AWS when capacity is needed elsewhere. In this architecture:

\* \*\*Scope:\*\* Only burst tasks use Spot, limiting the blast radius of an interruption.

\* \*\*Stability:\*\* Baseline tasks remain on On-Demand, ensuring the service stays available.

\* \*\*Self-Healing:\*\* ECS attempts to restore the desired count by launching replacement tasks (Spot if available, otherwise defaulting to strategy).



---



\## 💡 Why This Design Works



\### High Availability

\* Steady-state capacity is non-interruptible (Fargate On-Demand).

\* Tasks are distributed across multiple Availability Zones (Multi-AZ).

\* ECS Service keeps the baseline desired count stable.



\### Cost Optimization

\* Temporary capacity for short spikes is placed on Fargate Spot.

\* Avoids paying On-Demand rates for capacity that is only needed briefly.

\* System automatically returns to baseline cost after spikes end.



---



\## 🎯 When to Use This Pattern

This pattern is ideal when:

\* You have steady baseline traffic 24/7.

\* You experience short and unpredictable traffic bursts.

\* You need high availability but want to minimize costs during peak periods.



---



\## 📝 Summary

This architecture combines \*\*Fargate On-Demand\*\* for stability, \*\*Fargate Spot\*\* for savings, and \*\*Capacity Providers\*\* to manage them seamlessly via \*\*Base + Weight\*\* logic. With \*\*CloudWatch and Auto Scaling\*\*, you achieve always-on availability with cost-optimized elasticity.

