# Netflix — AWS Fargate Interview Questions

## Q1. Netflix’s streaming platform must scale globally. How would you use Fargate to handle spikes in viewership?

### Answer Outline
- ECS/EKS + Fargate tasks for microservices.  
- Multi-region deployment with Route53 and ALB/NLB.  
- Auto-scale tasks based on CPU, request rate, or custom metrics.  
- Caching with ElasticCache / DynamoDB for frequently accessed data.  
- Logging and monitoring via CloudWatch + centralized observability tools.  

## Q2. Background analytics jobs (content recommendations) with Fargate?

### Answer Outline
- Use SQS/Kafka for event-driven batch processing.  
- Fargate tasks auto-scale based on queue depth or scheduled intervals.  
- Externalize state to S3, DynamoDB, or RDS.  
- Retry logic, dead-letter queues, and monitoring.  

## Q3. Deployment strategy for zero downtime?

### Answer Outline
- Rolling updates in ECS Service.  
- Blue/Green deployment via ALB target groups.  
- CI/CD pipelines for automation.  
- Health checks and monitoring to ensure only healthy tasks serve traffic.
