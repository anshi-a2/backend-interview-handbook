# Amazon — AWS Fargate Interview Questions

## Q1. How would you design a highly available microservice architecture using Fargate for an Amazon-scale service?

### Answer Outline
- Use **ECS with Fargate** for serverless container orchestration.  
- Task definitions with CPU/memory, environment variables, IAM task roles.  
- ALB for routing traffic to Fargate tasks; health checks enabled.  
- Multi-AZ deployment for high availability.  
- Autoscaling based on CPU/memory or custom CloudWatch metrics.  
- Stateless services, persistent data in DynamoDB / S3 / RDS.  
- Logging/monitoring: CloudWatch logs and metrics, alarms.  
- Zero-downtime deploys: ECS rolling updates or blue/green via ALB target groups.

---

## Q2. Amazon handles unpredictable spikes in traffic (e.g., Prime Day). How would you implement Fargate-based autoscaling for such scenarios?

### Answer Outline
- Use **ECS Service Auto Scaling** based on custom metrics (requests per second, queue depth).  
- Pre-warm tasks for known spike windows or use predictive scaling.  
- Use SQS for decoupling tasks and allowing burstable background processing.  
- Monitor metrics and apply scaling policies: scale-out rapidly, scale-in slowly.  
- Ensure capacity is spread across multiple subnets/AZs to handle failures.  

---

## Q3. How would you secure Amazon’s container workloads on Fargate?

### Answer Outline
- Use **IAM Task Roles** with least privilege.  
- Inject secrets via **AWS Secrets Manager** or **SSM Parameter Store**.  
- Private subnets for internal services, proper Security Groups and NACLs.  
- Encrypt storage (S3, RDS, EFS) and communication (TLS).  
- Use minimal container images; scan for vulnerabilities.  
- CloudWatch for monitoring, logging, and alarms.

---

## Q4. Scenario: Background job processing system for Amazon order events.

### Answer Outline
- Fargate tasks poll **SQS** queue for order events.  
- Auto-scale based on queue length.  
- Tasks are stateless; store results in DynamoDB/S3.  
- Dead-letter queue for failed events.  
- CloudWatch metrics for latency, failure rates, processing throughput.  

---

## Q5. Trade-offs of using Fargate vs EC2 for Amazon-scale workloads.

### Answer Outline
- **Pros:** No infrastructure management, automatic scaling, task-level isolation, cost-efficient for bursty workloads.  
- **Cons:** Limited control over host, no GPU support, storage limits, cost may be higher for consistently high-utilization workloads.  
- **When to use EC2:** GPU-heavy workloads, long-running stable workloads, host-level customizations.  
