# WeChat - Continuous Deployment Project

A comprehensive multi-cloud microservices application demonstrating continuous integration and deployment (CI/CD) practices with automated builds, testing, and deployment to both Google Cloud Platform (GCP) and Microsoft Azure.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Microservices](#microservices)
- [CI/CD Pipeline](#cicd-pipeline)
- [Technologies](#technologies)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Deployment](#deployment)
- [Configuration](#configuration)
- [Contributing](#contributing)

## Overview

This project implements a scalable, cloud-native chat application with three microservices deployed across multiple cloud platforms. The system demonstrates modern DevOps practices including:

- **Multi-cloud deployment**: GCP (Google Kubernetes Engine) and Azure (Azure Kubernetes Service)
- **Containerization**: Docker-based microservices
- **Orchestration**: Kubernetes with Helm charts
- **CI/CD**: GitHub Actions with automated build, push, and deploy workflows
- **Microservices architecture**: Loosely coupled services with independent scaling
- **Real-time communication**: WebSocket-based chat functionality

## Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          GitHub Repository (Main Branch)                     │
│                                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐      │
│  │   Chat      │  │   Login     │  │  Profile    │  │     Helm     │      │
│  │  Service    │  │  Service    │  │  Service    │  │   Charts     │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └──────────────┘      │
└────────────────────────────────┬────────────────────────────────────────────┘
                                 │
                                 │ Push Event
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         GitHub Actions Workflow                              │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Stage 1: Check Changed Files                                        │   │
│  │  • Detect changes in microservices source code                      │   │
│  │  • Detect changes in Helm templates                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                 │                                             │
│                                 ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Stage 2: Build & Push Docker Images                                │   │
│  │  • Build Docker images for changed services                         │   │
│  │  • Tag with commit SHA                                              │   │
│  │  • Push to Google Artifact Registry (GAR)                           │   │
│  │  • Push to Azure Container Registry (ACR)                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                 │                                             │
│              ┌──────────────────┴──────────────────┐                        │
│              ▼                                      ▼                        │
│  ┌────────────────────────┐            ┌────────────────────────┐          │
│  │  Stage 3a: Deploy GCP  │            │  Stage 3b: Deploy AZ   │          │
│  │  • Update Helm values  │            │  • Update Helm values  │          │
│  │  • Deploy to GKE       │            │  • Deploy to AKS       │          │
│  └────────────────────────┘            └────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────────────┘
                     │                                      │
                     ▼                                      ▼
┌──────────────────────────────────┐    ┌──────────────────────────────────┐
│   Google Cloud Platform (GCP)    │    │   Microsoft Azure                │
│                                   │    │                                   │
│  ┌─────────────────────────────┐ │    │  ┌─────────────────────────────┐ │
│  │  Google Kubernetes Engine   │ │    │  │  Azure Kubernetes Service   │ │
│  │  (GKE)                       │ │    │  │  (AKS)                       │ │
│  │                              │ │    │  │                              │ │
│  │  ┌────────────────────────┐ │ │    │  │  ┌────────────────────────┐ │ │
│  │  │ Chat Service           │ │ │    │  │  │ Profile Service        │ │ │
│  │  │ (WebSocket)            │ │ │    │  │  │ (REST API)             │ │ │
│  │  └────────────────────────┘ │ │    │  │  └────────────────────────┘ │ │
│  │                              │ │    │  │                              │ │
│  │  ┌────────────────────────┐ │ │    │  │  ┌────────────────────────┐ │ │
│  │  │ Login Service          │ │ │    │  │  │ Login Service          │ │ │
│  │  │ (Web UI)               │ │ │    │  │  │ (Web UI)               │ │ │
│  │  └────────────────────────┘ │ │    │  │  └────────────────────────┘ │ │
│  │                              │ │    │  │                              │ │
│  │  ┌────────────────────────┐ │ │    │  │  ┌────────────────────────┐ │ │
│  │  │ Profile Service        │ │ │    │  │  │ Redis Cache            │ │ │
│  │  │ (REST API)             │ │ │    │  │  └────────────────────────┘ │ │
│  │  └────────────────────────┘ │ │    │  │                              │ │
│  │                              │ │    │  │  ┌────────────────────────┐ │ │
│  │  ┌────────────────────────┐ │ │    │  │  │ Ingress Controller     │ │ │
│  │  │ Redis Cache            │ │ │    │  │  └────────────────────────┘ │ │
│  │  └────────────────────────┘ │ │    │  └─────────────────────────────┘ │
│  │                              │ │    │                                   │
│  │  ┌────────────────────────┐ │ │    └───────────────────────────────────┘
│  │  │ Ingress Controller     │ │ │
│  │  └────────────────────────┘ │ │
│  └─────────────────────────────┘ │
│                                   │
│  ┌─────────────────────────────┐ │
│  │  Google Artifact Registry   │ │
│  │  (Container Images)          │ │
│  └─────────────────────────────┘ │
└───────────────────────────────────┘
```

### Microservices Communication Flow

```
┌─────────┐         ┌─────────┐         ┌─────────┐         ┌─────────┐
│  User   │────────▶│  Login  │────────▶│  Chat   │◀───────▶│ Profile │
│ Browser │         │ Service │         │ Service │         │ Service │
└─────────┘         └─────────┘         └─────────┘         └─────────┘
                         │                    │                    │
                         │                    │                    │
                         ▼                    ▼                    ▼
                    ┌────────────────────────────────────────────────┐
                    │              Redis Cache Layer                 │
                    │  • Session Management                          │
                    │  • Message Queue (Pub/Sub)                     │
                    │  • Profile Caching                             │
                    └────────────────────────────────────────────────┘
```

## Microservices

### 1. Chat Service

A scalable WebSocket-based real-time group chat service with profile integration.

**Technology Stack:**
- Java with Spring Boot
- WebSocket for real-time communication
- Redis Pub/Sub for message distribution
- H2/MySQL database support

**Endpoints:**
- `GET /health` - Health check and readiness probe
- `GET /getallprofiles` - Retrieve all user profiles
- `GET /getallmessages` - Retrieve chat message history
- `GET /profile?username=<username>` - Get specific user profile
- `WebSocket /message` - Real-time chat messaging

**Deployment:**
- **GCP**: Deployed to Google Kubernetes Engine (GKE)
- **Azure**: Not deployed (GCP only)

### 2. Login Service

User authentication service providing login functionality and session management.

**Technology Stack:**
- Java with Spring Boot
- Web UI for authentication
- Session management with Redis

**Endpoints:**
- `GET /` - Health check and readiness probe
- `GET /login` - Main login page and authentication

**Deployment:**
- **GCP**: Deployed to Google Kubernetes Engine (GKE)
- **Azure**: Deployed to Azure Kubernetes Service (AKS)

### 3. Profile Service

User profile management service with database backend.

**Technology Stack:**
- Java with Spring Boot
- H2 embedded database (dev) / MySQL (production)
- REST API for profile operations

**Features:**
- User profile CRUD operations
- Database flexibility (H2/MySQL)
- Profile caching with Redis

**Deployment:**
- **GCP**: Deployed to Google Kubernetes Engine (GKE)
- **Azure**: Deployed to Azure Kubernetes Service (AKS)

## CI/CD Pipeline

The project implements a sophisticated CI/CD pipeline using GitHub Actions with the following stages:

### 1. Change Detection

- Monitors changes in microservice source code directories
- Tracks Helm chart modifications
- Triggers selective builds based on changed files

### 2. Build & Push

**For each modified service:**
1. Build Docker image with commit SHA as tag
2. Tag images for both GCP and Azure registries
3. Push to Google Artifact Registry (GAR)
4. Push to Azure Container Registry (ACR)

### 3. Deployment

**GCP Deployment (GKE):**
- Update Helm values with new image tags
- Authenticate with GCP service account
- Deploy to Google Kubernetes Engine using Helm
- Services: chat-service, login-service, profile-service

**Azure Deployment (AKS):**
- Update Helm values with new image tags
- Authenticate with Azure using OIDC
- Deploy to Azure Kubernetes Service using Helm
- Services: login-service, profile-service

### Pipeline Triggers

- **Automatic**: Push to `main` branch
- **Manual**: Workflow dispatch from GitHub Actions UI

## Technologies

### Core Technologies
- **Language**: Java
- **Framework**: Spring Boot
- **Build Tool**: Maven
- **Containerization**: Docker
- **Orchestration**: Kubernetes (GKE, AKS)
- **Package Management**: Helm
- **Cache/Message Queue**: Redis

### Cloud Platforms
- **Google Cloud Platform (GCP)**
  - Google Kubernetes Engine (GKE)
  - Google Artifact Registry (GAR)
  - Service Account authentication
  
- **Microsoft Azure**
  - Azure Kubernetes Service (AKS)
  - Azure Container Registry (ACR)
  - OIDC authentication

### CI/CD & DevOps
- **GitHub Actions**: CI/CD orchestration
- **Docker**: Container images
- **Helm**: Kubernetes deployment management
- **GitHub**: Version control and collaboration

## Prerequisites

### Required Software
- Java 11 or higher
- Maven 3.6+
- Docker 20.10+
- kubectl 1.21+
- Helm 3.0+
- Git

### Required Cloud Accounts
- Google Cloud Platform account with GKE enabled
- Azure account with AKS enabled
- GitHub account

### Required Secrets (GitHub)

Configure the following secrets in your GitHub repository settings:

**GCP Secrets:**
- `GKE_SA_KEY`: Google Cloud service account key (JSON format)

**Azure Secrets:**
- `AZURE_CLIENT_ID`: Azure application client ID
- `AZURE_TENANT_ID`: Azure tenant ID
- `AZURE_SUBSCRIPTION_ID`: Azure subscription ID
- `AZ_REGISTRY_USERNAME`: Azure Container Registry username
- `AZ_REGISTRY_PASSWORD`: Azure Container Registry password

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/vismithaN/continuous-deployment.git
cd continuous-deployment
```

### 2. Configure Environment Variables

Update the environment variables in `.github/workflows/cicd.yml`:

```yaml
env:
  GCP_CLUSTER_NAME: your-gke-cluster-name
  GCP_PROJECT_ID: your-gcp-project-id
  GCP_REGION: your-gcp-region
  AZ_CONTAINER_REGISTRY: your-acr.azurecr.io
  AZ_CLUSTER_NAME: your-aks-cluster-name
  AZ_RESOURCE_GROUP: your-azure-resource-group
```

### 3. Update Helm Values

Edit `helm/values-gcp.yaml` and `helm/values-azure.yaml` with your registry paths and image names.

### 4. Local Development

Build and run services locally:

```bash
# Build a specific service
cd chat-service
./mvnw clean package

# Run with Maven
./mvnw spring-boot:run

# Build Docker image
docker build -f docker/DockerFile -t chat-service:local .
docker run -p 8080:8080 chat-service:local
```

## Deployment

### Automated Deployment

Deployment is automated through GitHub Actions:

1. Make changes to microservice code
2. Commit and push to `main` branch
3. GitHub Actions automatically:
   - Detects changed files
   - Builds Docker images
   - Pushes to registries
   - Deploys to Kubernetes clusters

### Manual Deployment

You can also deploy manually using Helm:

```bash
# Deploy to GCP
helm upgrade --install wecloud helm/ -f helm/values-gcp.yaml

# Deploy to Azure
helm upgrade --install wecloud helm/ -f helm/values-azure.yaml
```

### Verify Deployment

```bash
# Check GKE deployment
kubectl get pods
kubectl get services
kubectl get ingress

# Check AKS deployment
az aks get-credentials --resource-group <resource-group> --name <cluster-name>
kubectl get pods
kubectl get services
```

## Configuration

### Service Configuration

Each microservice has its own configuration in `helm/templates/`:
- `configMap-*.yaml`: Service-specific configuration
- `deployment-*.yaml`: Deployment specifications
- `service-*.yaml`: Kubernetes service definitions

### Helm Values

- `values.yaml`: Default values
- `values-gcp.yaml`: GCP-specific overrides
- `values-azure.yaml`: Azure-specific overrides

### Ingress Configuration

Ingress rules are defined in:
- `Ingress/ingress.yaml`: GCP ingress configuration
- `Ingress/ingress-azure.yaml`: Azure ingress configuration

## Project Structure

```
continuous-deployment/
├── .github/
│   └── workflows/
│       └── cicd.yml                 # CI/CD pipeline definition
├── chat-service/                    # Chat microservice
│   ├── src/                         # Java source code
│   ├── docker/                      # Dockerfile
│   ├── pom.xml                      # Maven configuration
│   └── README.md                    # Service documentation
├── login-service/                   # Login microservice
│   ├── src/
│   ├── docker/
│   ├── pom.xml
│   └── README.md
├── profile-service/                 # Profile microservice
│   ├── src/
│   ├── docker/
│   ├── pom.xml
│   └── README.md
├── helm/                            # Helm charts
│   ├── templates/                   # Kubernetes manifests
│   │   ├── deployment-*.yaml
│   │   ├── service-*.yaml
│   │   ├── configMap-*.yaml
│   │   └── redis.yaml
│   ├── values.yaml                  # Default values
│   ├── values-gcp.yaml              # GCP values
│   ├── values-azure.yaml            # Azure values
│   └── Chart.yaml                   # Helm chart metadata
├── Ingress/                         # Ingress configurations
│   ├── ingress.yaml                 # GCP ingress
│   └── ingress-azure.yaml           # Azure ingress
├── meta.json                        # Repository metadata
├── references                       # Citation references
└── README.md                        # This file
```

## Contributing

### Development Workflow

1. Create a feature branch from `main`
2. Make changes to microservices or infrastructure
3. Test locally using Docker
4. Push changes and create a pull request
5. After review, merge to `main` triggers automated deployment

### Code Standards

- Follow Java coding conventions
- Write meaningful commit messages
- Update documentation for new features
- Test changes before committing

### Adding References

Document all external resources used in the `references` file following the JSON format provided.

## Monitoring and Troubleshooting

### View Logs

```bash
# View pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# View all pods in namespace
kubectl get pods --all-namespaces
```

### Debug Issues

```bash
# Describe pod for events
kubectl describe pod <pod-name>

# Check service endpoints
kubectl get endpoints

# View GitHub Actions logs
# Navigate to: GitHub Repository → Actions → Select workflow run
```

### Common Issues

1. **Image pull errors**: Verify registry credentials and image tags
2. **Pod crashes**: Check logs and resource limits
3. **Service unreachable**: Verify ingress and service configurations
4. **Deployment timeout**: Check cluster resources and quotas

## License

This project is created for educational purposes as part of cloud computing and DevOps coursework.

## Acknowledgments

- Spring Boot framework for microservices development
- Kubernetes and Helm for orchestration
- GitHub Actions for CI/CD automation
- Google Cloud Platform and Microsoft Azure for cloud infrastructure

---

**Note**: This is an educational project demonstrating modern DevOps practices and multi-cloud deployment strategies. Configuration values and credentials should be properly secured in production environments.
