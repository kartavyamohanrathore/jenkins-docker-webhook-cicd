# CI/CD Pipeline: GitHub → Jenkins → Docker → AWS EKS

An automated CI/CD pipeline that builds a Docker image on every code push and deploys it to a Kubernetes cluster on Amazon EKS with a zero-downtime rolling update.

![CI/CD Pipeline Flow](docs/cicd_pipeline_flow.png)

## Tech Stack

- **Source control:** GitHub
- **CI/CD:** Jenkins (declarative pipeline)
- **Containerization:** Docker, DockerHub
- **Cloud:** AWS
- **Orchestration:** Kubernetes on Amazon EKS, managed with kubectl

## Pipeline Flow

### Phase 1: Continuous Integration

1. Developer pushes code to the GitHub repository
2. Jenkins pipeline starts
3. Jenkins clones the source code
4. Docker image is built
5. Image is tagged with the build number and `latest`
6. Image is pushed to DockerHub

### Phase 2: Continuous Deployment

7. Jenkins authenticates with AWS
8. kubectl is configured for the EKS cluster
9. Kubernetes YAML files are applied
10. Deployment image is updated to the new tag
11. Rolling update starts and pods are replaced gradually
12. Pipeline waits for the rollout to complete
13. Deployment is successful

## Repository Structure

```
.
├── Jenkinsfile          # Pipeline definition
├── Dockerfile           # Image build instructions
├── k8s/                 # Kubernetes manifests (deployment, service)
├── docs/
│   └── cicd_pipeline_flow.png
└── README.md
```

## Prerequisites

- Jenkins with Docker and kubectl installed
- DockerHub account and credentials stored in Jenkins
- AWS credentials with access to the EKS cluster, stored in Jenkins
- A running Amazon EKS cluster

## How It Works

On every push, Jenkins builds the image, tags it with the build number for traceable versions plus `latest`, and pushes it to DockerHub. It then authenticates with AWS, points kubectl at the EKS cluster, applies the manifests and updates the Deployment image. Kubernetes performs a rolling update, so the application stays available while new pods replace old ones, and the pipeline waits until the rollout finishes before reporting success.

