# DevOps CI/CD Pipeline on AWS

A hands-on DevOps project demonstrating an end-to-end CI/CD workflow for a containerized Flask application using **GitHub, Jenkins, Docker, AWS ECR, EC2, Terraform, Prometheus, and Grafana**.

The project automates the process of building a Docker image, pushing it to Amazon ECR, and deploying the application to an EC2 instance through a Jenkins pipeline. Terraform is used to provision the required AWS infrastructure, while Prometheus and Grafana provide system-level monitoring.

---

## 🏗️ Architecture

```text
                         ┌──────────────────┐
                         │     GitHub       │
                         │   Source Code    │
                         └────────┬─────────┘
                                  │
                                  │ Pipeline Trigger
                                  ▼
                         ┌──────────────────┐
                         │     Jenkins      │
                         │                  │
                         │  Build Docker    │
                         │  Image           │
                         └────────┬─────────┘
                                  │
                                  │ Push Image
                                  ▼
                         ┌──────────────────┐
                         │     AWS ECR      │
                         │  Docker Registry │
                         └────────┬─────────┘
                                  │
                                  │ Pull Image
                                  ▼
                    ┌──────────────────────────┐
                    │       AWS EC2            │
                    │                          │
                    │   ┌──────────────────┐   │
                    │   │   Flask App      │   │
                    │   │    Port 3000     │   │
                    │   └──────────────────┘   │
                    │                          │
                    │   ┌──────────────────┐   │
                    │   │  Node Exporter   │   │
                    │   │    Port 9100     │   │
                    │   └────────┬─────────┘   │
                    │            │             │
                    └────────────┼─────────────┘
                                 │
                                 ▼
                         ┌──────────────────┐
                         │    Prometheus    │
                         │    Port 9090     │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │     Grafana      │
                         │    Port 3001     │
                         └──────────────────┘


              Terraform
                  │
                  ├── EC2
                  ├── Security Group
                  ├── IAM Role
                  └── SSH Key Pair
```

---

## 🚀 Project Workflow

The complete workflow is:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Jenkins Pipeline
    │
    ├── Checkout source code
    │
    ├── Build Docker image
    │
    ├── Authenticate with AWS ECR
    │
    ├── Push image to ECR
    │
    └── SSH into EC2
            │
            ├── Pull latest image
            ├── Stop existing container
            └── Start updated container
```

Monitoring runs independently on the EC2 instance:

```text
EC2 System
    │
    ▼
Node Exporter
    │
    ▼
Prometheus
    │
    ▼
Grafana Dashboard
```

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Application | Python, Flask |
| Version Control | Git, GitHub |
| CI/CD | Jenkins |
| Containerization | Docker |
| Cloud | AWS |
| Container Registry | Amazon ECR |
| Compute | Amazon EC2 |
| Infrastructure as Code | Terraform |
| IAM | AWS IAM |
| Monitoring | Prometheus |
| Visualization | Grafana |
| System Metrics | Node Exporter |
| Deployment | SSH |

---

## 📁 Repository Structure

```text
devops-app/
│
├── app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
│
├── infra/
│   └── main.tf
│
├── monitoring/
│   ├── docker-compose.yml
│   └── prometheus.yml
│
├── Jenkinsfile
│
├── screenshots/
│   ├── jenkins-pipeline.png
│   ├── grafana-dashboard.png
│   ├── app-running.png
│   └── ecr-repo.png
│
└── README.md
```

---

## 🔧 Infrastructure with Terraform

Terraform is used to provision the AWS infrastructure required for the application.

The Terraform configuration currently provisions:

- Amazon EC2 instance
- Security group
- SSH key pair
- IAM role for EC2
- IAM instance profile
- ECR read-only permissions for the EC2 instance
- Amazon Linux AMI
- Docker installation through EC2 `user_data`
- AWS CLI installation through EC2 `user_data`

### Initialize Terraform

```bash
cd infra
terraform init
```

### Validate the configuration

```bash
terraform validate
```

### Review the infrastructure plan

```bash
terraform plan
```

### Create the infrastructure

```bash
terraform apply
```

Terraform outputs the public IP address of the EC2 instance after deployment.

---

🐳 Docker

The Flask application is packaged as a Docker image.

The application configuration is located in:


app/
├── app.py
├── Dockerfile
└── requirements.txt


The Dockerfile defines the application environment and startup command.

The image can be built locally with:

```bash
docker build -t devops-app ./app
```

Run the container:

```bash
docker run -d -p 3000:3000 devops-app
```

The application can then be accessed on:

http://localhost:3000

---

 🔄 Jenkins CI/CD Pipeline

Jenkins is used to automate the application build and deployment process.

The pipeline performs the following stages:

1. Checkout

Jenkins pulls the latest source code from GitHub.

2. Docker Build

A Docker image is created from the application Dockerfile.

3. ECR Authentication

Jenkins authenticates with Amazon ECR using AWS credentials configured in Jenkins.

4. Push to ECR

The Docker image is tagged and pushed to the private ECR repository.

5. EC2 Deployment

Jenkins connects to the EC2 instance through SSH and deploys the latest Docker image.

The deployment process replaces the currently running application container with the newly built image.

---

📦 Amazon ECR

Amazon Elastic Container Registry (ECR) is used as the private Docker image registry.

The workflow is:

Jenkins
   │
   │ docker build
   ▼
Docker Image
   │
   │ docker push
   ▼
Amazon ECR
   │
   │ docker pull
   ▼
EC2


The EC2 instance uses an IAM role with ECR read permissions instead of storing AWS access keys directly on the server.

---

 📊 Monitoring

The project includes a monitoring stack consisting of:

- Node Exporter — collects system-level metrics
- Prometheus — stores and queries metrics
- Grafana — visualizes the metrics through dashboards

The monitoring configuration is located in:

```text
monitoring/
├── docker-compose.yml
└── prometheus.yml
```

Monitoring Flow

```text
EC2 Host
   │
   │ System Metrics
   ▼
Node Exporter :9100
   │
   ▼
Prometheus :9090
   │
   ▼
Grafana :3001
```

Prometheus is configured to scrape Node Exporter every 15 seconds.

Grafana can then be used to visualize metrics such as:

- CPU usage
- Memory usage
- Disk usage
- Network activity
- System load

---

🔐 Security & IAM

The project uses AWS IAM to avoid placing long-lived AWS credentials directly on the EC2 server.

The EC2 instance is associated with an IAM role that provides:

```text
AmazonEC2ContainerRegistryReadOnly
```

This allows the instance to authenticate with ECR and pull container images.

Jenkins uses separately configured AWS and SSH credentials for the CI/CD process.

> Note: The security group configuration is intentionally simplified for this learning project. In a production environment, SSH and monitoring ports should be restricted to trusted IP ranges or private networking.

---

📸 Screenshots

 Jenkins Pipeline

![Jenkins Pipeline](screenshots/jenkins-pipeline.png)

 Amazon ECR

![Amazon ECR](screenshots/ecr-repo.png)

Deployed Flask Application

![Deployed Application](screenshots/app-running.png)

Grafana Monitoring Dashboard

![Grafana Dashboard](screenshots/grafana-dashboard.png)

---
💡 Key Learnings

This project helped me gain hands-on experience with:

- Writing and managing Terraform infrastructure
- Provisioning AWS resources using Infrastructure as Code
- Creating Docker images for Python applications
- Building Jenkins CI/CD pipelines
- Integrating Jenkins with AWS ECR
- Deploying containers to EC2 using SSH
- Working with AWS IAM roles and permissions
- Configuring Prometheus monitoring
- Using Node Exporter for system metrics
- Creating Grafana dashboards
- Troubleshooting container and deployment issues
- Managing infrastructure and application code through Git

---
🔮 Future Improvements

The project can be extended with:

- Automated unit tests in the Jenkins pipeline
- Terraform remote state using Amazon S3
- Terraform modules for reusable infrastructure
- HTTPS using a domain and TLS certificate
- Restricted security-group rules
- Prometheus alerting rules
- Grafana alerts
- Docker image vulnerability scanning
- Blue-green deployment
- Rolling deployments
- Kubernetes / Amazon EKS deployment
- Jenkins webhook-based automatic builds
- Centralized application logging

---
👩‍💻 Author

Suchita Gunasekar

GitHub: [github.com/gsuchita](https://github.com/gsuchita)
