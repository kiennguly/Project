\# Scalable and Highly Available 3-Tier Image Sharing Application on AWS



\## 1. Overview

This project describes the architecture of a \*\*three-tier image sharing application\*\* designed on AWS.  

The solution focuses on:

\- \*\*High availability\*\*

\- \*\*Horizontal scalability\*\*

\- \*\*Minimum application refactoring\*\*



The architecture preserves the existing application structure (front-end, application logic, and database) while migrating core components to \*\*managed AWS services\*\* to improve reliability and operational efficiency.



---



\## 2. Problem Statement

The original application consists of:

\- One Amazon EC2 instance for the \*\*front-end layer\*\*

\- One Amazon EC2 instance for the \*\*application layer\*\*

\- One Amazon EC2 instance running \*\*MySQL\*\* for the database



This design presents challenges:

\- Single points of failure

\- Limited scalability

\- Manual operational overhead



The goal is to redesign the architecture to be \*\*scalable and highly available\*\*, while requiring the \*\*least amount of change\*\* to the existing application.



---



\## 3. High-Level Architecture

The solution adopts a classic \*\*three-tier architecture\*\* using AWS managed services:



\- \*\*Front-end tier\*\*: AWS Elastic Beanstalk (Load Balanced, Multi-AZ)

\- \*\*Application tier\*\*: AWS Elastic Beanstalk (Load Balanced, Multi-AZ)

\- \*\*Database tier\*\*: Amazon RDS for MySQL (Multi-AZ)

\- \*\*Image storage and delivery\*\*: Amazon S3



---



\## 4. Architecture Diagram

The following diagram illustrates the overall system design:

!\[Architecture](project11.png)

```mermaid

flowchart LR

&nbsp; U\[Users] --> R53\[Amazon Route 53]

&nbsp; R53 --> ALB1\[Front-end ALB]



&nbsp; subgraph VPC\[AWS VPC - Multi AZ]

&nbsp;   subgraph FE\[Front-end Tier - Elastic Beanstalk]

&nbsp;     ALB1 --> FEASG\[EC2 Auto Scaling Group]

&nbsp;   end



&nbsp;   subgraph APP\[Application Tier - Elastic Beanstalk]

&nbsp;     ALB2\[Application ALB] --> APPASG\[EC2 Auto Scaling Group]

&nbsp;   end



&nbsp;   subgraph DB\[Database Tier - RDS MySQL Multi-AZ]

&nbsp;     RDSW\[Primary]

&nbsp;     RDSS\[Standby]

&nbsp;     RDSW <-.-> RDSS

&nbsp;   end

&nbsp; end



&nbsp; FEASG --> ALB2

&nbsp; APPASG --> RDSW

&nbsp; APPASG --> S3\[Amazon S3 - Image Storage]

&nbsp; U --> S3



