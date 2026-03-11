# Netflix Clone — DevSecOps CI/CD Pipeline

A full DevSecOps pipeline for a Netflix clone application — containerized with Docker, security scanned with Trivy & OWASP, deployed on Kubernetes via Jenkins CI/CD on AWS EC2.

## Architecture

```
GitHub → Jenkins Pipeline
           ├── SonarQube (Code Quality)
           ├── OWASP Dependency Check (CVE Scanning)
           ├── Trivy FS Scan (Filesystem vulnerabilities)
           ├── Docker Build & Push (DockerHub)
           ├── Trivy Image Scan (Container vulnerabilities)
           └── Kubernetes Deploy (AWS EC2)
                ├── deployment.yml
                └── service.yml
```

## Pipeline Stages

| Stage | Tool | Purpose |
|-------|------|---------|
| Clean Workspace | Jenkins | Fresh build environment |
| Git Checkout | GitHub | Pull latest source code |
| SonarQube Analysis | SonarQube | Static code analysis |
| Quality Gate | SonarQube | Block on code quality failure |
| Install Dependencies | Node.js/NPM | Install app dependencies |
| OWASP FS Scan | OWASP DC | Check for known CVEs in dependencies |
| Trivy FS Scan | Trivy | Filesystem vulnerability scan |
| Docker Build & Push | Docker | Build image, push to DockerHub |
| Trivy Image Scan | Trivy | Container image vulnerability scan |
| Deploy to Container | Docker | Run container locally |
| Deploy to Kubernetes | kubectl | Deploy to K8s cluster on AWS |

## Project Structure

```
.
├── Kubernetes/
│   ├── deployment.yml    # K8s Deployment (2 replicas)
│   └── service.yml       # K8s Service
├── pipeline.txt          # Jenkins declarative pipeline
└── README.md
```

## Infrastructure Setup (AWS EC2)

### Instances Required

| Server | Type | Purpose |
|--------|------|---------|
| Jenkins | t2.large, 30GB | CI/CD pipeline |
| SonarQube | t2.medium, 25GB | Code quality |
| Kubernetes Master | t2.medium, 25GB | K8s control plane |
| Kubernetes Worker | t2.medium, 25GB | K8s worker node |

### Jenkins Plugins Required

- Docker & Docker Pipeline
- Kubernetes & Kubernetes CLI
- SonarQube Scanner
- OWASP Dependency Check
- NodeJS
- Email Extension

## Jenkins Pipeline Setup

### 1. Global Tools Configuration
Navigate: `Manage Jenkins → Tools`

- **JDK:** jdk17 (Adoptium)
- **NodeJS:** node16
- **SonarQube Scanner:** sonar-scanner
- **Docker:** latest

### 2. Credentials Setup
Navigate: `Manage Jenkins → Credentials`

| ID | Type | Value |
|----|------|-------|
| `sonar-token` | Secret text | SonarQube token |
| `docker` | Username/Password | DockerHub credentials |
| `k8s` | Secret text | Kubernetes service account token |

### 3. SonarQube Server
Navigate: `Manage Jenkins → System → SonarQube Servers`
- Name: `sonar-server`
- URL: `http://<sonarqube-ip>:9000`

## Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netflix-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: netflix-app
  template:
    spec:
      containers:
      - name: netflix-app
        image: shubhamharanale7/netflix:latest
        ports:
        - containerPort: 80
```

Apply manually:
```bash
kubectl apply -f Kubernetes/deployment.yml
kubectl apply -f Kubernetes/service.yml
kubectl get pods
kubectl get svc
```

## Security Scanning

### Trivy
```bash
# Filesystem scan
trivy fs . > trivyfs.txt

# Image scan
trivy image shubhamharanale7/netflix:latest > trivyimage.txt
```

### OWASP Dependency Check
```
--scan ./ --disableYarnAudit --disableNodeAudit
```

Results published as Jenkins HTML report.

## Email Notifications

Jenkins sends build result emails with Trivy scan reports attached:
- `trivyfs.txt` — filesystem scan
- `trivyimage.txt` — image scan

Configure SMTP in `Manage Jenkins → System → Extended E-mail Notification`.

## Cleanup

```bash
# Stop containers
docker ps
docker stop <container-id>

# Delete K8s resources
kubectl delete -f Kubernetes/deployment.yml
kubectl delete -f Kubernetes/service.yml

# Terminate EC2 instances from AWS Console
```

## Tech Stack

- **Jenkins** — CI/CD orchestration
- **SonarQube** — static code analysis
- **OWASP Dependency Check** — CVE scanning
- **Trivy** — container & filesystem security scanning
- **Docker** — containerization
- **Kubernetes** — container orchestration
- **AWS EC2** — cloud infrastructure
- **Node.js** — application runtime

---

**Built by [Shubham Haranale](https://github.com/Shubhamharanale7)**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Shubhamharanale7)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/shubhamharanale7)

