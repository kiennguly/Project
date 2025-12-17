# CloudFront + S3 Frontend + ALB + EC2 Auto Scaling Backend + WAF + Route 53
Production-Ready Reference Architecture (Static SPA + Scalable API)

---

## 1. Overview

This project documents a practical AWS web architecture where:

- **Frontend** is a static website (SPA) hosted on **Amazon S3** and delivered globally by **Amazon CloudFront**.
- **Backend** is a scalable application/API running on **Amazon EC2** behind an **Application Load Balancer (ALB)** and managed by an **Auto Scaling Group (ASG)**.
- **Security** is strengthened using **AWS WAF** at the edge (CloudFront) and least-privilege access patterns (private S3 via OAC, restricted backend entry).
- **DNS & routing** is managed using **Amazon Route 53** for clean domain mapping (e.g., `app.example.com`).

This setup is widely used for real products: marketing sites + SPA frontends, admin dashboards, and scalable REST APIs.

---

## 2. High-Level Architecture

### Request Flow (User → Edge → Origins)

1. User queries DNS via **Route 53** (`app.example.com`).
2. Route 53 routes traffic to **CloudFront** (Alias/AAAA).
3. **CloudFront + WAF** receives the request at the nearest edge location.
4. CloudFront routes requests based on path patterns:
   - `/*` → **S3 (Static assets)** via **OAC** (private S3)
   - `/api/*` → **ALB (Backend)** → **EC2 Auto Scaling Group**
5. Metrics/logs are captured for observability (CloudFront/WAF/ALB/EC2/ASG).

---

## 3. Components and Responsibilities

### 3.1 Route 53 (DNS)
**Role**
- Authoritative DNS for your domain (e.g., `example.com`).
- Routes `app.example.com` to CloudFront using Alias records.

**Benefits**
- Highly available DNS.
- Clean separation: domain management independent from app compute.

---

### 3.2 CloudFront (CDN / Edge)
**Role**
- Global edge delivery for frontend assets.
- Smart caching, compression, HTTP/2/3 support.
- Path-based routing to multiple origins (S3 + ALB).

**Benefits**
- Lower latency globally (edge caching).
- Reduces load on origins (S3/ALB) and cuts bandwidth costs.
- Built-in resilience: edge locations can absorb spikes.

---

### 3.3 AWS WAF (Edge Security)
**Role**
- Web Application Firewall attached to CloudFront.
- Filters malicious traffic before it reaches your origins.

**Common Rule Sets**
- AWS Managed Rules (OWASP Top 10 patterns)
- Bot control / rate limiting
- Geo restriction (if needed)
- IP reputation lists

**Benefits**
- Stops attacks early (SQLi/XSS/bots) at edge.
- Protects ALB/EC2 from unnecessary traffic.
- Supports compliance/security posture.

---

### 3.4 S3 (Static Frontend Origin)
**Role**
- Stores frontend build artifacts: HTML/CSS/JS/images.
- Configured **private**: no public read.
- CloudFront reads objects using **Origin Access Control (OAC)**.

**Benefits**
- Extremely cost-effective hosting.
- Highly durable and highly available.
- Simplifies CI/CD (upload new build → invalidate cache).

---

### 3.5 ALB (Backend Entry Point)
**Role**
- Receives requests routed from CloudFront for `/api/*`.
- Terminates TLS (HTTPS) and forwards to EC2 targets.
- Performs health checks and distributes traffic.

**Benefits**
- Layer 7 routing (host/path-based).
- Smooth scaling: no client-side awareness of instances.
- Health-aware load distribution.

---

### 3.6 EC2 Auto Scaling Group (Compute Layer)
**Role**
- Runs backend service (REST API / app server).
- Automatically scales in/out based on load (CPU/ReqCount/Latency).
- Spreads instances across multiple AZs for high availability.

**Benefits**
- Elastic capacity (handles spikes).
- Improved uptime with multi-AZ redundancy.
- Cost control: scale down when idle.

---

## 4. Why This Architecture Is Practical (Real-World Applicability)

### 4.1 Fits Most Modern Web Products
- SPA frontends (React/Vue/Angular)
- Admin dashboards
- Public landing pages + private API
- Event-driven / microservice backends behind an API

### 4.2 Strong Separation of Concerns
- Frontend is serverless (S3 + CloudFront).
- Backend is scalable compute (ALB + ASG).
- Security is centralized at the edge (WAF).

### 4.3 Handles Traffic Spikes Cleanly
- CloudFront caches and absorbs huge bursts.
- ALB distributes load to healthy targets.
- ASG scales automatically.

---

## 5. Benefits Summary (Professional Breakdown)

### Performance
- Edge caching reduces Time-To-First-Byte (TTFB).
- Compression and CDN delivery optimize bandwidth and latency.

### Scalability
- Backend scales horizontally with ASG.
- Frontend scales automatically by design (S3 + CloudFront).

### Availability
- Multi-AZ EC2 fleet.
- CloudFront global edge network.
- S3 durability + availability.

### Security
- WAF blocks common threats at the edge.
- Private S3 via OAC prevents direct bucket access.
- Backend can be hardened to reduce bypass risk.

### Cost Efficiency
- Static hosting is extremely cheap.
- Autoscaling prevents over-provisioning.
- CDN caching lowers origin egress and compute costs.

### Operational Simplicity
- Clear component boundaries.
- Easy CI/CD for frontend (upload to S3 + invalidate CloudFront).
- Standard monitoring through CloudWatch.

---

## 6. Hardening Recommendations (Pro-Level)

### 6.1 Prevent “Bypass ALB via Direct Access”
If ALB is internet-facing, attackers can try to hit it directly (bypassing CloudFront/WAF).

**Mitigation Options**
1) **CloudFront → ALB Origin Custom Header**
- CloudFront adds a secret header (e.g., `X-Origin-Verify: <secret>`)
- ALB listener rules deny requests missing/wrong header (return 403)

2) **Restrict ALB Security Group**
- Allow inbound only from trusted sources (advanced; depends on IP strategy/prefix lists)

Recommended in documentation:
- Use **custom header + ALB listener rule** as a clear, auditable control.

---

### 6.2 Secure S3 with OAC (No Public Access)
- Block public access at bucket level.
- Allow reads only from the specific CloudFront distribution using OAC.
- Prevents direct hotlinking and data exposure.

---

## 7. Example CloudFront Behaviors

| Path Pattern | Origin | Purpose | Cache Strategy |
|------------:|--------|---------|----------------|
| `/*`        | S3     | Frontend assets | Cache enabled (long TTL for hashed files) |
| `/api/*`    | ALB    | Backend API | Cache disabled or short TTL |

Notes:
- Use hashed filenames for JS/CSS for safe long caching.
- Disable caching for dynamic API responses unless explicitly desired.

---

## 8. Monitoring & Observability

### Metrics to Watch
- **CloudFront**: cache hit ratio, 4xx/5xx, latency
- **WAF**: blocked requests, rule match count
- **ALB**: target response time, HTTPCode_Target_5XX, unhealthy host count
- **ASG/EC2**: CPUUtilization, memory (via agent), network in/out

### Logging
- CloudFront access logs (optional)
- ALB access logs to S3 (recommended)
- WAF logs (optional but useful for security investigations)

---

## 9. Deployment Notes (High-Level)

Frontend:
1. Build SPA
2. Upload artifacts to S3
3. Invalidate CloudFront cache (paths: `/index.html`, `/*` as needed)

Backend:
1. Bake AMI or use Launch Template (UserData to install app)
2. Configure Target Group + Health Checks
3. Attach ASG to Target Group
4. Enable Auto Scaling policies (CPU/ALB request count)

---

## 10. Use Cases

- Company landing page + scalable API
- Student project portfolio with production-like architecture
- Internal dashboard with secure edge + scalable backend
- Multi-tenant SaaS frontend + service layer

---

## 11. Appendix: Mermaid Diagram (Optional)

```mermaid
flowchart LR
  U[User / Browser] -->|DNS| R53[Route 53]
  R53 -->|Alias| CF[CloudFront]
  WAF[AWS WAF] -. attached .- CF
  CF -->|/*| S3[(S3 Static Assets - Private)]
  CF -. OAC .-> S3
  CF -->|/api/*| ALB[ALB]
  ALB --> TG[(Target Group)]
  TG --> ASG[EC2 Auto Scaling]
  ASG --> EC2A[(EC2 A)]
  ASG --> EC2B[(EC2 B)]
