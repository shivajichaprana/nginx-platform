# Nginx Platform - ECS Fargate Deployment

Simple Nginx application deployed to AWS ECS Fargate using GitHub Actions with OIDC authentication.

## Architecture

- **Application**: Nginx serving static HTML
- **Container Registry**: AWS ECR
- **Compute**: AWS ECS Fargate
- **Load Balancer**: Application Load Balancer
- **CI/CD**: GitHub Actions with OIDC (keyless authentication)

## Repository Structure

```
application/
├── .github/workflows/
│   ├── CI.yaml          # Build and push Docker image
│   └── CD.yaml          # Deploy to ECS
├── src/
│   ├── Dockerfile       # Container image definition
│   └── index.html       # Static HTML content
└── deploy/
    └── task-def.json    # ECS task definition template
```

## CI/CD Pipeline

### CI Workflow (Triggered on push to main)
1. Checkout code
2. Authenticate to AWS via OIDC
3. Login to Amazon ECR
4. Build Docker image
5. Push image with commit SHA tag and latest tag

### CD Workflow (Triggered after CI completes)
1. Checkout code
2. Authenticate to AWS via OIDC
3. Render ECS task definition with new image
4. Deploy to ECS service
5. Wait for service stability

## Configuration

### AWS Resources Required
- ECR Repository: `nginx-demo`
- ECS Cluster: `nginx-fargate-demo-cluster`
- ECS Service: `nginx-fargate-demo-service`
- IAM Role: `github-actions-nginx-platform` (OIDC)
- Region: `us-east-1`

### GitHub Settings
- **Repository**: `shivajichaprana/nginx-platform`
- **Branch Protection**: Only `main` branch can deploy
- **Permissions**: Workflows have `id-token: write` for OIDC

## Security

- ✅ No AWS credentials stored in GitHub
- ✅ OIDC authentication with temporary tokens
- ✅ Scoped IAM permissions
- ✅ Only main branch can deploy
- ✅ Container health checks enabled

## Local Development

### Option 1: Using Docker Compose (Recommended)
```bash
docker-compose up
```

### Option 2: Using Docker directly
```bash
cd src
docker build -t nginx-platform:local .
docker run -p 8080:80 nginx-platform:local
```

Visit: http://localhost:8080

## Deployment

Push to `main` branch triggers automatic deployment:

```bash
git add .
git commit -m "Update application"
git push origin main
```

## Monitoring

- **Application URL**: Check ALB DNS from infrastructure outputs
- **ECS Logs**: CloudWatch Logs group `/ecs/nginx-fargate-demo`
- **GitHub Actions**: Check workflow runs for deployment status

## Image Tags

- `latest`: Most recent successful build from main branch (used for deployments)
- `<commit-sha>`: Specific version for rollback (e.g., `abc1234`)

## Health Check

Container health check runs every 30 seconds:
```bash
wget --no-verbose --tries=1 --spider http://localhost/ || exit 1
```

## Troubleshooting

### Deployment fails
- Check GitHub Actions logs
- Verify OIDC role permissions
- Check ECS service events in AWS Console

### Container unhealthy
- Check CloudWatch logs: `/ecs/nginx-fargate-demo`
- Verify health check endpoint responds

### Cannot access application
- Check ALB target health
- Verify security group rules
- Check ECS service is running
