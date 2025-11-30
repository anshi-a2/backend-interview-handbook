# Google — AWS Fargate Interview Questions

## Q1. Google Cloud workloads may be containerized but run hybrid cloud. How would you migrate or run Fargate-based workloads?

### Answer Outline
- ECS or EKS with Fargate for AWS workloads.  
- Use container images stored in ECR (or Artifact Registry) compatible with multi-cloud.  
- Stateless workloads with persistent data in S3/DynamoDB.  
- CI/CD pipelines for container deployment and updates.  

## Q2. How would you design autoscaling for Google-scale analytics services?

### Answer Outline
- Fargate tasks for batch or streaming workloads.  
- Auto-scale tasks using CloudWatch custom metrics (data processed, queue depth).  
- Use SQS/Kinesis for buffering and decoupling workloads.  
- CloudWatch logs/metrics for monitoring throughput and failures.  

## Q3. Google-level security for container workloads?

### Answer Outline
- Least-privilege IAM Task Roles.  
- Secrets Manager for sensitive credentials.  
- VPC segmentation, private subnets, and security groups.  
- Encrypt storage and communications.  
- Logging and monitoring with alerts.
