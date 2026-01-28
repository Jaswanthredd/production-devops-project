# Production DevOps Project

A production-ready DevOps project featuring a Flask web application deployed on AWS using modern DevOps practices.

## Architecture

- **Application**: Simple Flask web app with health checks
- **Containerization**: Docker
- **Infrastructure**: AWS (VPC, ECS Fargate, RDS PostgreSQL, ALB)
- **CI/CD**: GitHub Actions
- **Monitoring**: CloudWatch logs and alarms
- **Security**: IAM roles, VPC security groups, secrets management

## Project Structure

```
production-devops-project/
├── app/
│   ├── app.py              # Flask application
│   ├── requirements.txt    # Python dependencies
│   └── Dockerfile          # Docker configuration
├── terraform/
│   ├── main.tf             # Infrastructure as Code
│   ├── variables.tf        # Terraform variables
│   ├── outputs.tf          # Terraform outputs
│   └── provider.tf         # AWS provider configuration
├── .github/
│   └── workflows/
│       └── ci-cd.yml       # GitHub Actions pipeline
└── README.md               # This file
```

## Prerequisites

- AWS Account with appropriate permissions
- GitHub repository
- Terraform installed locally (for manual deployment)
- Docker installed locally (for testing)

## Local Development

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd production-devops-project
   ```

2. Build and run locally:
   ```bash
   cd app
   docker build -t flask-devops-app .
   docker run -p 5000:5000 flask-devops-app
   ```

3. Test the application:
   - Open http://localhost:5000 for the main page
   - Open http://localhost:5000/health for health check

## Deployment

### Automated Deployment (Recommended)

1. Set up GitHub Secrets:
   - `AWS_ACCESS_KEY_ID`: Your AWS access key
   - `AWS_SECRET_ACCESS_KEY`: Your AWS secret key
   - `DB_PASSWORD`: Database password for RDS

2. Push to main branch:
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

   The GitHub Actions pipeline will automatically:
   - Run tests
   - Build and push Docker image to ECR
   - Deploy infrastructure with Terraform

### Manual Deployment

1. Configure AWS credentials:
   ```bash
   aws configure
   ```

2. Deploy infrastructure:
   ```bash
   cd terraform
   terraform init
   terraform plan -var="db_password=YOUR_DB_PASSWORD"
   terraform apply -var="db_password=YOUR_DB_PASSWORD"
   ```

3. Build and push Docker image:
   ```bash
   cd app
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
   docker build -t flask-devops-app .
   docker tag flask-devops-app:latest YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app:latest
   docker push YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app:latest
   ```

4. Update ECS service to use new image (or wait for CI/CD).

## Accessing the Application

After deployment, get the ALB DNS name:
```bash
cd terraform
terraform output alb_dns_name
```

Visit `http://<alb_dns_name>` in your browser.

## Monitoring

- **Logs**: Available in CloudWatch Logs group `/ecs/flask-devops-app`
- **Metrics**: ECS CPU utilization alarm configured
- **Health Checks**: ALB health checks on `/health` endpoint

## Security Features

- VPC with public/private subnets
- Security groups restricting traffic
- IAM roles with least privilege
- Secrets managed via GitHub Secrets and Terraform variables

## Scaling

The ECS service is configured with 2 tasks. To scale:
```bash
aws ecs update-service --cluster flask-devops-app-cluster --service flask-devops-app-service --desired-count 4
```

## Cleanup

To destroy all resources:
```bash
cd terraform
terraform destroy -var="db_password=YOUR_DB_PASSWORD"
```

## Best Practices Implemented

- Infrastructure as Code with Terraform
- Containerization with Docker
- Automated testing and deployment
- Health checks and monitoring
- Load balancing and auto-scaling ready
- Environment variable configuration
- Secure secrets management
- Logging and alerting
- Modular code structure