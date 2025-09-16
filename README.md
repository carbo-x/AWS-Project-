 AWS Auto Scaling Group with Elastic Load Balancer Project

 Project Overview
This project demonstrates the implementation of a scalable and highly available web application infrastructure using AWS services including EC2, Auto Scaling Groups (ASG), and Elastic Load Balancer (ELB).

 Architecture Diagram

                                                   Internet Gateway
                                                        ↓
                                                 Application Load Balancer
                                                        ↓
                                                    Auto Scaling Group
                                                                ├── EC2 Instance 1 (Availability Zone A)
                                                                ├── EC2 Instance 2 (Availability Zone B)
                                                                └── EC2 Instance N (Auto-scaled based on demand)

 Features
- High Availability: Instances distributed across multiple Availability Zones
- Auto Scaling: Automatically scales in/out based on CPU utilization
- Load Distribution: Traffic evenly distributed across healthy instances
- Health Monitoring: Continuous health checks and automatic replacement of unhealthy instances

AWS Services Used
- EC2: Virtual servers hosting the application
- Auto Scaling Groups: Automated instance management
- Elastic Load Balancer: Traffic distribution and health checks
- CloudWatch: Monitoring and alerting
- VPC: Network isolation and security

Configuration Details

 EC2 Instances
- AMI: Amazon Linux 2
- Instance Type: t2.micro (Free tier eligible)
- Storage: 8GB gp2 EBS volume
- Security Groups: 
  - HTTP (Port 80) - Open to Load Balancer
  - SSH (Port 22) - Restricted to your IP

Load Balancer Configuration
- Type: Application Load Balancer (Layer 7)
- Scheme: Internet-facing
- Listeners: HTTP on port 80
- Target Group: 
  - Protocol: HTTP
  - Port: 80
  - Health Check: HTTP GET /
- Health Check Settings:
  - Healthy threshold: 2
  - Unhealthy threshold: 2
  - Timeout: 5 seconds
  - Interval: 30 seconds

Auto Scaling Group Settings
- Desired Capacity: 2 instances
- Minimum Size: 1 instance
- Maximum Size: 5 instances
- Scaling Policies:
  - Scale Out: CPU utilization > 70% for 2 consecutive periods
  - Scale In: CPU utilization < 30% for 2 consecutive periods
- Cooldown Period: 300 seconds

Implementation Steps

 1. Launch Template Creation
user-data.sh

2. Security Group Configuration
- Web Server Security Group:
  - Inbound: HTTP (80) from Load Balancer Security Group
  - Inbound: SSH (22) from your IP address
- Load Balancer Security Group:
  - Inbound: HTTP (80) from 0.0.0.0/0
  - Inbound: HTTPS (443) from 0.0.0.0/0 (if SSL configured)

3. Network Configuration
- VPC: Default VPC or custom VPC
- Subnets: At least 2 public subnets in different AZs
- Internet Gateway: Attached to VPC for internet access

Testing and Validation

Load Testing
```bash
# Use Apache Bench to generate load
ab -n 10000 -c 100 http://your-load-balancer-dns-name/

# Monitor scaling activity
aws autoscaling describe-scaling-activities --auto-scaling-group-name your-asg-name
```

Health Check Verification
- Monitor target group health in EC2 console
- Check CloudWatch metrics for instance health
- Verify automatic replacement of failed instances

Monitoring and Alerting

Key CloudWatch Metrics
- ASG Metrics:
  - Group Desired Capacity
  - Group In Service Instances
  - Group Total Instances
- ELB Metrics:
  - Request Count
  - Target Response Time
  - Healthy/Unhealthy Host Count
- EC2 Metrics:
  - CPU Utilization
  - Network In/Out
  - Status Check Failed

Recommended Alarms
- High CPU utilization (>80%)
- Unhealthy target count (>0)
- ELB 5XX errors
- Instance status check failures

Cost Optimization
- Use t2.micro instances (Free tier eligible)
- Monitor and adjust scaling policies
- Set up billing alerts
- Use Spot Instances for non-critical workloads (optional)

 Troubleshooting

Common Issues
1. Instances failing health checks
   - Check security group rules
   - Verify web server is running
   - Check application logs

2. Auto Scaling not triggering
   - Verify CloudWatch metrics are being published
   - Check scaling policies and thresholds
   - Ensure cooldown periods haven't been breached

3. Load Balancer returning 503 errors
   - Check target group health
   - Verify instance security groups
   - Check instance capacity

Clean Up
To avoid ongoing charges, clean up resources in this order:
1. Delete Auto Scaling Group
2. Delete Load Balancer
3. Delete Target Groups
4. Terminate any remaining EC2 instances
5. Delete Launch Template
6. Delete Security Groups (if custom)
