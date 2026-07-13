# Highly Available & Secure Static Website Hosting on AWS
A production-style AWS architecture that hosts a static website behind a load-balanced, auto-scaling, HTTPS-secured infrastructure — built to reflect how a real company would deploy a public-facing site, not just a "hello world" demo.
 
---
 
## Table of Contents
- [The Problem](#the-problem)
- [The Solution](#the-solution)
- [Architecture](#architecture)
- [AWS Services Used & Why](#aws-services-used--why)
- [Deployment Walkthrough](#deployment-walkthrough)
- [Security Considerations](#security-considerations)
- [Challenges & How I Solved Them](#challenges--how-i-solved-them)
- [What I'd Improve Next](#what-id-improve-next)
- [Skills Demonstrated](#skills-demonstrated)
- [Contact](#contact)
---
 
## The Problem
 
Imagine a small business or startup that needs to launch its marketing website. On paper this sounds simple — upload some HTML and go live. In practice, a real company can't just drop files on a single server, because that approach fails the moment:
 
- The server goes down and there's no failover → **the site disappears**
- Traffic spikes (a product launch, a press mention) → **the server crashes**
- The site runs on plain HTTP → **browsers flag it as "Not Secure," damaging trust**
- The server is publicly exposed with no network segmentation → **it's an easy attack target**
- Traffic grows and no one is watching capacity → **manual scaling becomes a bottleneck**
This project simulates solving that exact problem: hosting a static website the way a company actually would, with **availability, security, and scalability** built in from the start — not bolted on later.
 
## The Solution
 
I designed and deployed a **highly available, auto-scaling, HTTPS-secured web hosting environment** on AWS using a custom VPC, load-balanced EC2 instances across multiple Availability Zones, and a managed SSL certificate tied to a real domain via Route 53.
 
The result: a static site that can survive an instance failure, scale automatically under load, and serve traffic securely over `https://` on a custom domain — with no single point of failure in the compute layer.
 
## Architecture
 
```
                                   Internet
                                      |
                              Route 53 (DNS)
                                      |
                          ACM SSL Certificate (HTTPS)
                                      |
                        Application Load Balancer (Public Subnets)
                                      |
                        ---------------------------------
                        |                                 |
                Auto Scaling Group                 Auto Scaling Group
                EC2 (Private Subnet - AZ 1)         EC2 (Private Subnet - AZ 2)
                        |                                 |
                        ---------------------------------
                                      |
                               NAT Gateway (Public Subnet)
                                      |
                              Outbound Internet Access
```
 
**Key design choice:** the web servers themselves sit in **private subnets**, not exposed directly to the internet. All inbound traffic is forced through the Application Load Balancer, and outbound traffic (e.g., OS updates) is routed through a NAT Gateway. This mirrors how production environments are actually segmented.
 
## AWS Services Used & Why
 
| Service | Why I chose it |
|---|---|
| **VPC (Public + Private Subnets)** | Segments the network so web servers aren't directly internet-facing — only the load balancer is. This is a foundational security pattern in real production environments. |
| **Security Groups** | Enforced least-privilege access — the ALB only accepts HTTP/HTTPS from the internet, and EC2 instances only accept traffic from the ALB's security group, not the open internet. |
| **NAT Gateway** | Lets private-subnet instances reach the internet for updates/patches without being reachable *from* the internet — outbound only. |
| **EC2 + Auto Scaling Group** | Instead of a single server (a single point of failure), the ASG automatically replaces unhealthy instances and scales capacity up or down based on demand — directly solving the "traffic spike crashes the site" problem. |
| **Application Load Balancer** | Distributes traffic across multiple Availability Zones and integrates natively with the ASG and health checks, so a failed instance is silently routed around instead of causing downtime. |
| **Route 53** | Registered and managed a real custom domain instead of relying on an AWS-generated URL — this is what a real business would need for branding and credibility. |
| **AWS Certificate Manager (ACM)** | Issued a free, auto-renewing SSL/TLS certificate so the site serves over HTTPS, avoiding browser security warnings and protecting data in transit. |
| **HTTPS Listener on the ALB** | Terminates SSL at the load balancer, so encryption is handled centrally rather than configured on every individual server. |
 
> **Note:** This project does not use RDS, since the site is fully static (HTML/CSS/JS) and has no dynamic data layer or backend database requirement.
 
## Deployment Walkthrough
 
1. Created a custom VPC with public and private subnets across two Availability Zones for fault tolerance
2. Configured route tables and an Internet Gateway for public subnet internet access
3. Deployed a NAT Gateway in the public subnet to give private instances outbound-only internet access
4. Created security groups following least privilege — ALB open to 80/443 from the internet, EC2 instances only open to the ALB's security group
5. Launched EC2 instances in the private subnets and configured them to serve the static website
6. Created a Launch Template and an Auto Scaling Group to maintain availability and handle scaling
7. Deployed an Application Load Balancer across the public subnets
8. Registered a domain name in Route 53 and created a record set pointing to the ALB
9. Requested and validated an SSL certificate through AWS Certificate Manager
10. Configured an HTTPS listener on the ALB using the ACM certificate, and redirected HTTP → HTTPS
## Security Considerations
 
- Web servers are never directly exposed to the internet — all traffic passes through the ALB
- Security groups are scoped tightly rather than left open (`0.0.0.0/0` only where genuinely required, e.g., the ALB's HTTP/HTTPS listeners)
- HTTPS is enforced end-to-end for anything reaching the public internet
- Outbound-only internet access for private instances via NAT Gateway reduces the attack surface
## Challenges & How I Solved Them
 
*(This is a great section to personalize — swap in what actually gave you trouble. A couple of common ones for this exact build:)*
 
- **Health checks failing on new instances:** Initially the ASG kept cycling instances because the ALB health check path didn't match how the web server was serving content — resolved by aligning the health check path/port with the actual running service.
- **Certificate validation delay:** ACM validation via DNS took time to propagate through Route 53 — solved by using DNS validation records directly instead of email validation, which is faster and automatable.
## What I'd Improve Next
 
Being transparent about next steps shows maturity, not weakness — here's what I'd add to take this from a portfolio project to a fully production-grade setup:
 
- **CloudFront (CDN):** Cache content at edge locations for faster global load times and reduced load on origin servers
- **Infrastructure as Code:** Rebuild this using Terraform or CloudFormation instead of manual console configuration, for repeatability and version control
- **CI/CD Pipeline:** Automate deployments with CodePipeline/CodeBuild or GitHub Actions so site updates deploy without manual intervention
- **AWS WAF:** Add a Web Application Firewall in front of the ALB for protection against common exploits (SQL injection, XSS, bot traffic)
- **CloudWatch Monitoring & Alarms:** Set up dashboards and alerts for instance health, latency, and traffic anomalies
- **Cost Optimization / Budget Alarms:** Add AWS Budgets to monitor spend, and evaluate S3 + CloudFront static hosting as a lower-cost alternative for pure static content
- **Multi-Region Failover:** For true disaster recovery, replicate the architecture into a second region with Route 53 failover routing
## Skills Demonstrated
 
`AWS VPC Design` `Network Segmentation` `EC2` `Auto Scaling` `Application Load Balancer` `Route 53 / DNS Management` `SSL/TLS (ACM)` `Security Group Configuration` `NAT Gateway` `High Availability Architecture` `Cloud Security Fundamentals`
 
## Contact
 
**[Your Name]**
[LinkedIn] · [Email] · [Portfolio/GitHub]
 
---
 
*This project was built as a hands-on exercise in designing AWS infrastructure the way a real business would need it — prioritizing availability, security, and scalability over a minimal "it works" setup.*
