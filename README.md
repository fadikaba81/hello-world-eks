# 🚀 Go Web App on AWS EKS (Deployed via GitHub Actions)

This project demonstrates how to build, containerize, and deploy a simple **Go web application** into **Amazon EKS** using **GitHub Actions CI/CD pipeline**.

The pipeline automatically:
1. Builds the Go application
2. Creates a Docker image
3. Pushes the image to **Amazon ECR**
4. Deploys the updated image to an **EKS cluster**

---

## 🏗️ Project Architecture

your-app/
├── .github/
│ └── workflows/
│ └── deploy.yaml # GitHub Actions deployment pipeline
├── Dockerfile # Multi-stage docker build
├── main.go # Go web server
└── k8s/
├── deployment.yaml # Kubernetes Deployment
└── service.yaml # Kubernetes Service (LoadBalancer)

---

## 🌐 Application (Go Web Server)

`main.go` runs a lightweight web server that listens on `:8080`:

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from Go running in EKS 🚀")
})
```
🐳 Docker Build
The app is built using a multi-stage Dockerfile:
```
dockerfile
Copy code
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/server .
EXPOSE 8080
CMD ["./server"]
Build and run locally:

sh
docker build -t go-web .
docker run -p 8080:8080 go-web
Visit: http://localhost:8080
```

☸️ Kubernetes Deployment

Located in k8s/deployment.yaml and k8s/service.yaml:

Deployment defines pod and container

Service exposes the app via a LoadBalancer

Apply manually (optional):

sh
Copy code
kubectl apply -f k8s/
🔁 GitHub Actions (CI/CD Pipeline)
Workflow: .github/workflows/deploy.yaml

What it does:

Step Action
✅ Checkout code	Pulls repo from GitHub
✅ Login to AWS	Uses aws-actions/configure-aws-credentials
✅ Build Docker image	Uses Docker CLI
✅ Push to ECR	Stores container image
✅ Update EKS deployment	kubectl set image ...

Trigger: every push to branch main.

🔐 GitHub Secrets Required
```
Secret Name	Description
AWS_ACCESS_KEY_ID_IAM User Access Key
AWS_SECRET_ACCESS_KEY_IAM User Secret
AWS_ACCOUNT_ID_AWS Account ID
EKS_CLUSTER_NAME_Name of your EKS cluster

Create them in:

GitHub → Repository → Settings → Secrets → Actions

✅ Required IAM Permissions
Assign these permissions to the IAM user used by GitHub Actions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": "eks:*", "Resource": "*" },
    { "Effect": "Allow", "Action": "ecr:*", "Resource": "*" },
    { "Effect": "Allow", "Action": "sts:*", "Resource": "*" }
  ]
}
```
For least-privilege security, this can be restricted later.

🚀 Deployment Flow (How it works)
Push code to main branch:

```sql
git add .
git commit -m "update app"
git push origin main
GitHub Actions will:

Build new Docker image

Push to ECR with SHA tag

Update image in EKS

Trigger rolling update

Run rollout status (optional):
```

```sh
kubectl rollout status deployment/go-web
🧠 Troubleshooting
Issue Fix
- Pods not updating Ensure kubectl set image matches the deployment/container name
- LoadBalancer not getting an external IP Verify AWS subnet tagging in EKS
Unauthorized ECR push Check IAM policy & GitHub secrets
```

📌 Future Enhancements (Ideas)
Add Helm charts instead of raw manifests

Add OpenTelemetry tracing

Add health/liveness probes

Add monitoring dashboards (CloudWatch / Prometheus)

✨ Author
Fadi Kaba
Cloud & SRE Engineer
GitHub: https://github.com/fadikaba81