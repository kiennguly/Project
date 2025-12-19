# Scalable and Highly Available 3-Tier Image Sharing Application on AWS

## 1. Overview
This project describes the architecture of a three-tier image sharing application designed on AWS.  
The solution focuses on:
- **High availability**
- **Horizontal scalability**
- **Minimum application refactoring**

The architecture preserves the existing application structure (front-end, application logic, and database) while migrating core components to managed AWS services to improve reliability and operational efficiency.

---

## 2. Problem Statement
The original application consists of:
- One Amazon EC2 instance for the front-end layer
- One Amazon EC2 instance for the application layer
- One Amazon EC2 instance running MySQL for the database

This design presents challenges:
- Single points of failure
- Limited scalability
- Manual operational overhead

The goal is to redesign the architecture to be scalable and highly available, while requiring the least amount of change to the existing application.

---

## 3. High-Level Architecture
The solution adopts a classic three-tier architecture using AWS managed services:

- **Front-end tier**: AWS Elastic Beanstalk (Load Balanced, Multi-AZ)
- **Application tier**: AWS Elastic Beanstalk (Load Balanced, Multi-AZ)
- **Database tier**: Amazon RDS for MySQL (Multi-AZ)
- **Image storage and delivery**: Amazon S3

---

## 4. Architecture Diagram
The following diagram illustrates the overall system design:
![Architecture](project11.png)
# Architecture Explanation: Image Sharing Platform on AWS

## 5. Component Breakdown

### 5.1 Front-end Tier
Implemented using AWS Elastic Beanstalk with a load-balanced environment.
* **Deployment:** Runs across multiple Availability Zones (AZs).
* **Scaling:** Automatically scales EC2 instances based on traffic.
* **Reliability:** Improves fault tolerance and reduces operational overhead.

**Benefits:**
* No code changes required.
* Built-in health checks and self-healing.
* Seamless scaling during traffic spikes.

### 5.2 Application Tier
Deployed as a separate Elastic Beanstalk environment.
* **Logic:** Handles business logic such as authentication, image metadata processing, and API requests.
* **Infrastructure:** Load balanced and auto-scaled across multiple AZs.

**Benefits:**
* Independent scaling from the front-end.
* Maintains existing application logic.
* Simplified deployment and monitoring.

### 5.3 Database Tier
Uses **Amazon RDS for MySQL** with Multi-AZ deployment.
* **Replication:** Provides synchronous replication between primary and standby instances.
* **Failover:** Automatic failover in case of instance or AZ failure.



**Benefits:**
* High availability without application changes.
* Managed backups and patching.
* Reduced risk of data loss.

### 5.4 Image Storage (Amazon S3)
All user-uploaded images are stored in Amazon S3.
* **Delivery:** Images are served directly from S3 to users.
* **Storage:** Application stores only metadata and object references.

**Benefits:**
* Virtually unlimited scalability.
* High durability ($99.999999999\%$).
* Reduced load on application servers.

---

## 6. Request Flow
1. **Access:** User accesses the application via a domain name.
2. **DNS:** Amazon Route 53 resolves the DNS request.
3. **Routing:** Traffic is routed to the front-end Application Load Balancer (ALB).
4. **Forwarding:** Front-end servers forward API requests to the application tier ALB.
5. **Processing:** Application servers:
   * Read/write metadata to Amazon RDS.
   * Upload and retrieve images from Amazon S3.
6. **Delivery:** Images are delivered directly to users from S3.



---

## 7. Scalability and High Availability

### Scalability
* **Compute:** Front-end and application tiers scale horizontally using Auto Scaling.
* **Storage:** Amazon S3 scales automatically with demand.
* **Database:** Database tier scales vertically and supports read replicas if needed.

### High Availability
* Multi-AZ Elastic Beanstalk environments.
* Multi-AZ RDS deployment with automatic failover.
* Architecture designed with No Single Point of Failure (NSPF).

---

## 8. Security Considerations
* **Access Control:** EC2 instances use IAM roles with least-privilege permissions.
* **Storage Security:** S3 buckets use bucket policies and pre-signed URLs.
* **Network:** Database access is restricted to application-specific security groups.
* **Data Encryption:**
  * **At rest:** RDS, S3.
  * **In transit:** HTTPS/TLS.

---

## 9. Cost Optimization
* **Compute:** Auto Scaling ensures a "pay-for-use" model.
* **Storage:** S3 storage tiers can be optimized using lifecycle policies.
* **Database:** RDS Multi-AZ balances cost and availability for production workloads.

---

## 10. Conclusion
This architecture delivers a scalable, highly available, and production-ready image sharing platform while requiring minimal changes to the existing application. By leveraging AWS managed services such as Elastic Beanstalk, RDS Multi-AZ, and Amazon S3, the solution significantly improves reliability, scalability, and operational efficiency.

---

## 11. Future Enhancements
* **CloudFront:** Add Amazon CloudFront for global image delivery and lower latency.
* **Security:** Integrate AWS WAF to protect against web exploits.
* **Automation:** Enable CI/CD pipelines using AWS CodePipeline.
* **Performance:** Introduce read replicas for read-heavy database workloads.
