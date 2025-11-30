# AWS Fargate — Company-Specific Interview Questions & Solutions
*Advanced SDE2 / Backend / Cloud Interview Guide*

---

## Q1. Design a scalable microservice deployment using Fargate for a web API. What components will you use, how will you handle deployment, load balancing, scaling, high availability?

**Solution / Key Points:**
- Use Amazon ECS or EKS with **Fargate** launch type.
- Define a **Task Definition** with container image, CPU, memory, environment variables, ports.
- Deploy a **Service** with desired task count and auto-scaling.
- Use an **Application Load Balancer (ALB)** with health checks for tasks.
- VPC design: private subnets, security groups, NAT / Internet access as needed.
- Enable **auto-scaling** based on CPU, memory, or request metrics.
- Logging & monitoring: CloudWatch Logs/Metrics, alarms.
- For zero-downtime: use rolling updates or blue/green via ALB target groups or pipelines.

---

## Q2. Suppose your team asks: “Should we migrate from EC2 + ECS to Fargate?” What factors would you consider?

**Solution / Key Points:**
- **Operational overhead**: Fargate reduces instance maintenance (patching, scaling, monitoring).
- **Cost & utilization**: Fargate can be cheaper for bursty workloads; EC2 may be cheaper for high constant utilization.
- **Workload patterns**: Fargate fits variable, event-driven workloads.
- **Control & customization**: EC2 allows privileged containers, GPUs, custom OS tweaks.
- **Security & isolation**: Fargate offers task-level isolation.
- Recommendation: Hybrid approach is possible; choose Fargate for microservices and unpredictable workloads, EC2 for high-performance or specialized workloads.

---

## Q3. How would you implement zero-downtime deployments / rolling updates / blue-green deployments with Fargate?

**Solution / Approach:**
- ECS Service rolling update: update Task Definition; ECS replaces old tasks gradually.
- Use ALB health checks to ensure new tasks are ready before traffic shifts.
- For blue-green: deploy a new service + target group; shift traffic using ALB rules.
- CI/CD pipelines (CodeDeploy, CloudFormation, Terraform, CDK) can manage deployment and rollback.
- Consider cold starts, session handling, and external shared state (DB/EFS) during deployments.

---

## Q4. How would you design a backend worker system for unpredictable SQS workloads using Fargate?

**Solution / Design:**
- Fargate tasks poll messages from SQS.
- Auto-scale tasks based on queue depth.
- Task Definition: worker container, env vars, IAM role for queue and DB access.
- Use stateless tasks where possible; external storage for persistence.
- Logging & monitoring via CloudWatch.
- Dead-letter queue for failed messages.
- Benefit: pay only for running tasks, handles bursty workloads automatically.

---

## Q5. Drawbacks / Limitations of Fargate vs EC2/EKS. When would you avoid Fargate?

**Solution / Key Points:**
- Less control over infrastructure (cannot run privileged containers, custom OS tweaks, GPUs).
- Higher cost for consistently high-load workloads compared to EC2 reserved/spot.
- Storage & I/O constraints: ephemeral storage limited; EFS adds latency.
- Networking limits: subnet IP availability, ENI allocation.
- Debugging less flexible than EC2.
- Avoid Fargate when needing specialized hardware, privileged containers, or stable high-load workloads where EC2 is more cost-effective.

---

## Q6. How would you secure a Fargate-based microservices deployment?

**Solution / Best Practices:**
- Use **IAM Task Roles** (or IRSA for EKS) with least privilege.
- Manage secrets using **Secrets Manager** or **SSM Parameter Store**.
- Deploy tasks in **private subnets**; configure Security Groups & NACLs.
- Encrypt data at rest and in transit (EFS, S3, RDS).
- Enable logging & monitoring (CloudWatch, VPC flow logs).
- Use minimal, scanned container images to reduce attack surface.

---

## Q7. Company Scenario: SaaS Platform with Bursty Load

**Task:** Design a containerized backend for variable load, including API and worker jobs, with cost-efficiency and minimal maintenance.

**Solution Sketch:**
1. ECS + Fargate for APIs and workers.
2. API Service: ALB + auto-scaling based on CPU/request metrics.
3. Worker Service: SQS-triggered tasks, auto-scale on queue depth.
4. ECR for images; CI/CD for deployment.
5. Logging & monitoring: CloudWatch.
6. Secrets: Secrets Manager / SSM injected at runtime.
7. Persistent storage: RDS / DynamoDB / S3 / EFS.
8. Deploy strategies: rolling updates; blue/green if critical.
9. Security: least-privilege IAM, private networking, encrypted storage.
10. Cost optimization: right-sized containers, pay-as-you-go.

---

## ✅ Notes

- These questions combine **practical design, trade-offs, and company-level reasoning**.
- Focus on **scalability, cost, reliability, and security**.
- Be ready to discuss alternative designs and justify your choices.
- Include metrics and monitoring in your answers — interviewers value observability planning.

