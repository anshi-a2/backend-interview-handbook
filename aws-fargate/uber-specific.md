# Uber — AWS Fargate Interview Questions

## Q1. Uber handles high-volume, real-time ride requests. How would you design a Fargate-based backend to handle unpredictable spikes?

### Answer Outline
- ECS + Fargate tasks for ride-matching microservices.  
- Auto-scale tasks based on API request metrics or queue length (SQS/Kafka).  
- Multi-AZ deployment to handle failures.  
- ALB or NLB for routing requests.  
- Use stateless microservices; persistent storage in RDS/DynamoDB/S3.  
- CloudWatch for monitoring latency, error rates, and scaling events.  

## Q2. How would you design background job processing for trip analytics and fare calculations?

### Answer Outline
- Queue-based architecture (SQS or Kafka).  
- Fargate worker tasks auto-scale based on queue depth.  
- Idempotent processing, retry logic, and DLQ for failed tasks.  
- CloudWatch metrics for monitoring throughput and failures.  

## Q3. Security and compliance for Uber-scale workloads?

### Answer Outline
- IAM Task Roles for least-privilege access.  
- Secrets in Secrets Manager / SSM Parameter Store.  
- Private subnets, security groups, VPC isolation.  
- Data encryption in transit (TLS) and at rest (S3, DynamoDB, RDS).  
- Logging & monitoring: CloudWatch, CloudTrail, VPC Flow Logs.  

## Q4. Cost optimization for Uber’s Fargate usage?

### Answer Outline
- Right-size tasks (CPU/memory) based on workload.  
- Use auto-scaling to only run tasks when needed.  
- For predictable workloads, consider reserved capacity or EC2 hybrid.  
- Monitor CloudWatch metrics to detect over-provisioning.
