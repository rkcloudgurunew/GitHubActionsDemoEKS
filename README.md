# Spring Boot EKS Deployment with GitHub Actions

A simple Spring Boot application with automated deployment to AWS EKS using GitHub Actions.

## 📋 Prerequisites

Before deploying, ensure you have:

### 1. AWS Resources
- **EKS Cluster**: A running EKS cluster
- **ECR Repository**: Create an ECR repository for your Docker images
- **IAM User**: With permissions for ECR and EKS access
- **kubectl**: Configured to access your EKS cluster

### 2. GitHub Secrets
Add the following secrets to your GitHub repository:
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key

### 3. Local Development Tools
- Java 17+
- Maven 3.6+
- Docker
- kubectl
- AWS CLI

## 🚀 Quick Start

### Step 1: Create AWS Resources

#### 1.1 Create EKS Cluster
```bash
# Install eksctl if not already installed
brew install eksctl  # macOS
# or download from https://github.com/weaveworks/eksctl

# Create EKS cluster
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

#### 1.2 Create ECR Repository
```bash
# Create ECR repository
aws ecr create-repository \
  --repository-name spring-eks-demo \
  --region us-east-1

# Note the repository URI from the output
```

#### 1.3 Update Configuration
Update the following files with your AWS details:

**`.github/workflows/deploy-to-eks.yml`**:
```yaml
env:
  AWS_REGION: us-east-1              # Your AWS region
  ECR_REPOSITORY: spring-eks-demo    # Your ECR repository name
  EKS_CLUSTER_NAME: my-eks-cluster   # Your EKS cluster name
```

**`k8s/deployment.yaml`**:
Replace the image placeholder with your ECR URL format (this will be automated by the workflow).

### Step 2: Set Up GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add:
   - Name: `AWS_ACCESS_KEY_ID`, Value: Your AWS access key
   - Name: `AWS_SECRET_ACCESS_KEY`, Value: Your AWS secret key

### Step 3: Deploy

#### Option 1: Deploy via GitHub Actions (Recommended)
```bash
# Commit and push your code
git add .
git commit -m "Initial commit"
git push origin main
```

The GitHub Actions workflow will automatically:
1. Build the Spring Boot application
2. Create a Docker image
3. Push to Amazon ECR
4. Deploy to EKS cluster

#### Option 2: Manual Local Deployment
```bash
# Build the application
mvn clean package

# Build Docker image
docker build -t spring-eks-demo:latest .

# Tag and push to ECR (replace with your ECR URL)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
docker tag spring-eks-demo:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/spring-eks-demo:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/spring-eks-demo:latest

# Update kubeconfig
aws eks update-kubeconfig --name my-eks-cluster --region us-east-1

# Update deployment.yaml with your ECR image URL
# Then deploy to EKS
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

## 🔍 Verify Deployment

```bash
# Check pods
kubectl get pods

# Check service
kubectl get service spring-eks-demo-service

# Get LoadBalancer URL
kubectl get service spring-eks-demo-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Test the application
curl http://<LOAD_BALANCER_URL>/
curl http://<LOAD_BALANCER_URL>/health
curl http://<LOAD_BALANCER_URL>/info
```

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-to-eks.yml      # GitHub Actions workflow
├── k8s/
│   ├── deployment.yaml            # Kubernetes deployment
│   └── service.yaml               # Kubernetes service
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/demo/
│       │       └── DemoApplication.java
│       └── resources/
│           └── application.properties
├── Dockerfile                      # Multi-stage Docker build
├── pom.xml                        # Maven configuration
└── README.md
```

## 🔧 Customization

### Change Application Port
Edit [src/main/resources/application.properties](src/main/resources/application.properties):
```properties
server.port=8080
```

### Adjust Resources
Edit [k8s/deployment.yaml](k8s/deployment.yaml):
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "1Gi"
    cpu: "500m"
```

### Scale Replicas
```bash
kubectl scale deployment spring-eks-demo --replicas=3
```

## 📊 Monitoring

```bash
# View logs
kubectl logs -l app=spring-eks-demo

# Follow logs
kubectl logs -f deployment/spring-eks-demo

# Check resource usage
kubectl top pods
kubectl top nodes
```

## 🛠️ Troubleshooting

### Pod not starting
```bash
# Describe pod to see events
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>
```

### ImagePullBackOff error
- Verify ECR repository exists
- Check IAM permissions for EKS nodes to pull from ECR
- Ensure the image was pushed successfully

### Service not accessible
```bash
# Check service
kubectl get svc spring-eks-demo-service

# Verify security groups allow traffic on port 80
# Check EKS cluster security group settings
```

## 🧹 Cleanup

```bash
# Delete Kubernetes resources
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml

# Delete EKS cluster
eksctl delete cluster --name my-eks-cluster --region us-east-1

# Delete ECR repository
aws ecr delete-repository --repository-name spring-eks-demo --region us-east-1 --force
```

## 📚 Additional Resources

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📝 License

This project is open source and available under the MIT License.
# Deploying to EKS
