# ShopNow Backend: Java Spring Boot Microservices with AWS Infrastructure & DevSecOps Pipeline

Enterprise-grade Spring Boot microservices backend deployed on AWS with production-ready CI/CD automation, Docker containerization, and comprehensive security scanning across development and production environments.

<details>
<summary><strong>Table of Contents</strong></summary>

- [ShopNow Backend: Java Spring Boot Microservices with AWS Infrastructure \& DevSecOps Pipeline](#shopnow-backend-java-spring-boot-microservices-with-aws-infrastructure--devsecops-pipeline)
  - [1. System Architecture](#1-system-architecture)
  - [2. Microservices Overview](#2-microservices-overview)
    - [API Gateway](#api-gateway)
    - [Discovery Server (Eureka)](#discovery-server-eureka)
    - [Config Server](#config-server)
    - [Product Service](#product-service)
    - [User Service](#user-service)
    - [Shopping Cart Service](#shopping-cart-service)
  - [3. Multi-Environment Strategy](#3-multi-environment-strategy)
    - [Development Environment](#development-environment)
    - [Production Environment (EKS/K8s - via shopnow-infa)](#production-environment-eksk8s---via-shopnow-infa)
    - [Environment-Specific Configuration](#environment-specific-configuration)
  - [4. Network \& Security](#4-network--security)
    - [Development Environment (EC2)](#development-environment-ec2)
    - [Production Environment (EKS)](#production-environment-eks)
  - [5. Repository Structure \& Build Management](#5-repository-structure--build-management)
    - [Docker Image Management](#docker-image-management)
  - [6. Docker Microservices](#6-docker-microservices)
    - [Local Development with Docker Compose](#local-development-with-docker-compose)
  - [7. Tech Stack](#7-tech-stack)
    - [Backend Framework](#backend-framework)
    - [Service Architecture](#service-architecture)
    - [Data Access](#data-access)
    - [Containerization \& DevOps](#containerization--devops)
    - [Security \& Scanning](#security--scanning)
    - [API Documentation](#api-documentation)
  - [8. API Documentation](#8-api-documentation)
    - [Available Endpoints](#available-endpoints)
    - [Swagger UI](#swagger-ui)
    - [Postman Collection](#postman-collection)
  - [9. Monitoring \& Operations](#9-monitoring--operations)
    - [Local Logging](#local-logging)
    - [Production Monitoring (EKS)](#production-monitoring-eks)
    - [Service Dependencies](#service-dependencies)
  - [10. Contact Information](#10-contact-information)

</details>

---

## 1. System Architecture

The backend is built with a distributed microservices architecture on AWS:

* **API Gateway:** Spring Cloud Gateway routes all requests to appropriate microservices, handles OAuth2/OIDC authentication via Keycloak.
* **Service Discovery:** Eureka server enables automatic service registration and discovery for inter-service communication.
* **Config Server:** Centralized configuration management for all microservices (dev/prod environment variables).
* **Microservices:** Independent Spring Boot services (Product, User, Shopping Cart) with separate databases.
* **Authentication:** Keycloak handles OAuth2/OpenID Connect with role-based access control (RBAC).
* **Database:** PostgreSQL for relational data, MySQL for Keycloak state.
* **Container Registry:** AWS ECR stores multi-service Docker images.
* **Logging & Monitoring:** AWS CloudWatch aggregates logs from all containerized services.

--- 

## 2. Microservices Overview

### API Gateway
- **Purpose:** Single entry point for all frontend requests
- **Responsibilities:** Request routing, OAuth2 token validation, rate limiting
- **Port:** 5860
- **Stack:** Spring Cloud Gateway, Spring Security, Keycloak Integration
- **Dockerfile:** [api-gateway/Dockerfile](./api-gateway/Dockerfile)

### Discovery Server (Eureka)
- **Purpose:** Service registry for dynamic service discovery
- **Responsibilities:** Registers all microservices, health checks, load balancing
- **Port:** 8761
- **Stack:** Spring Cloud Netflix Eureka
- **Dockerfile:** [discovery-server/Dockerfile](./discovery-server/Dockerfile)

### Config Server
- **Purpose:** Centralized configuration management
- **Responsibilities:** Provides environment-specific configs to all services
- **Port:** 5859
- **Stack:** Spring Cloud Config Server
- **Dockerfile:** [config-server/Dockerfile](./config-server/Dockerfile)

### Product Service
- **Purpose:** Product catalog management
- **Responsibilities:** CRUD operations for products, inventory management
- **Port:** 5861
- **Database:** PostgreSQL
- **Stack:** Spring Boot Data JPA, OpenFeign for inter-service calls
- **Dockerfile:** [product-service/Dockerfile](./product-service/Dockerfile)

### User Service
- **Purpose:** User account management
- **Responsibilities:** User registration, profile management, authentication integration
- **Port:** 5865
- **Database:** PostgreSQL
- **Stack:** Spring Boot Data JPA, Spring Security
- **Dockerfile:** [user-service/Dockerfile](./user-service/Dockerfile)

### Shopping Cart Service
- **Purpose:** Shopping cart operations
- **Responsibilities:** Add/remove items, cart persistence, order preparation
- **Port:** 5863
- **Database:** PostgreSQL
- **Stack:** Spring Boot Data JPA, Feign clients to Product/User services
- **Dockerfile:** [shopping-cart-service/Dockerfile](./shopping-cart-service/Dockerfile)

---

## 3. Multi-Environment Strategy

### Development Environment

- **Trigger:** Manual docker-compose deployment
- **Configuration:** All services in single docker-compose stack
- **Database:** PostgreSQL container (single instance)
- **Authentication:** Keycloak container with MySQL backend
- **CloudWatch Logs:** `/ec2-docker/api`, `/ec2-docker/products`, `/ec2-docker/cart`, `/ec2-docker/user`
- **Port Range:** 5859-5865 on localhost
- **Features:** All services running, fast iteration, less strict security

### Production Environment (EKS/K8s - via shopnow-infa)

- **Deployment:** Kubernetes manifests on AWS EKS
- **Database:** AWS RDS PostgreSQL (managed, high-availability)
- **Authentication:** Keycloak deployed on EKS with RDS backend
- **CloudWatch Logs:** `/prod/api-gateway`, `/prod/product-service`, `/prod/user-service`, `/prod/cart-service`
- **Replica Count:** 2-3 pods per service for HA
- **Resources:** CPU/Memory limits enforced
- **Features:** Full security scanning, auto-scaling, rolling updates, zero-downtime deployments

### Environment-Specific Configuration

| Aspect | Development | Production |
|--------|-------------|-----------|
| **Deployment** | Docker Compose | Kubernetes (EKS) |
| **Database** | PostgreSQL Container | AWS RDS PostgreSQL |
| **Keycloak** | Container (MySQL) | EKS Pod (RDS MySQL) |
| **Service Discovery** | Eureka Container | Kubernetes DNS |
| **Logging** | CloudWatch (optional) | CloudWatch (required) |
| **Replicas** | 1 per service | 2-3 per service |
| **Resource Limits** | None | CPU/Memory enforced |
| **Auto-scaling** | Manual | Horizontal Pod Autoscaler |
| **Deployment Time** | 2-5 minutes | 10-15 minutes |
| **Rollback** | Manual | Kubernetes instant rollback |

---

## 4. Network & Security

Infrastructure provisioned via Terraform [shopnow-infa](https://github.com/Bel7phegor/shopnow-infa):

### Development Environment (EC2)

* **VPC CIDR:** 10.0.0.0/16
* **Public Subnets:** Bastion EC2 + NAT Gateway
* **Private Subnets:** Backend runner EC2 (Docker containers)
* **Single NAT Gateway:** Cost-optimized for dev

**Security Groups:**
- Bastion SG: SSH (22) from VPC only
- Backend Runner SG: ECR pull, GitHub API, frontend ALB ingress

### Production Environment (EKS)

* **VPC CIDR:** 10.0.0.0/16
* **Multi-AZ Public Subnets:** NAT Gateways (one per AZ)
* **Multi-AZ Private Subnets:** EKS worker nodes, RDS
* **Network Load Balancer (NLB):** Routes to API Gateway service
* **Service-to-Service:** Kubernetes NetworkPolicy for pod-to-pod isolation

**Security Groups (EKS):**
- EKS Control Plane SG: Ingress from nodes (443) & bastion
- EKS Worker Nodes SG: Node-to-node, bastion access, NLB ingress (80, 443, 30000-32767)
- RDS SG: PostgreSQL (5432) from EKS nodes only

**Database Security:**
- PostgreSQL: Private subnet, RDS security group isolation
- Keycloak MySQL: Private subnet, RDS managed
- Encryption: RDS encryption enabled
- Backups: Automated daily snapshots (30-day retention)

**SSL/TLS Encryption:**
- AWS Certificate Manager (ACM) manages SSL certificates
- TLS 1.2+ enforced
- NLB listener: 80 (HTTP redirect) → 443 (HTTPS)

---

## 5. Repository Structure & Build Management

### Docker Image Management

**Multi-stage Build Pattern (each service):**

```dockerfile
# Stage 1: Build
FROM openjdk:17.0.1-jdk-slim AS builder
  ├─ ./mvnw clean package
  └─ Creates target/*.war

# Stage 2: Runtime
FROM openjdk:17.0.1-jdk-slim
  ├─ Copy WAR from builder
  ├─ Run java -jar
  └─ Output: ~400MB optimized image
```

**Image Tagging Strategy:**
- Dev: `shopnow-backend-api-gateway:dev_${SHA}`, `:latest`
- Prod: `shopnow-backend-api-gateway:${VERSION}_${SHA}`, `:latest`
- Registry: AWS ECR (private repository)

---

## 6. Docker Microservices

### Local Development with Docker Compose

**Services Running:**
- api-gateway (5860)
- product-service (5861)
- shopping-cart-service (5863)
- user-service (5865)
- discovery-server (8761)
- config-server (5859)
- PostgreSQL (6543 → 5432)
- Keycloak (8080)
- Keycloak MySQL (internal)

**Environment Variables (docker-compose):**
```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/postgres
SPRING_DATASOURCE_USERNAME: postgres
SPRING_DATASOURCE_PASSWORD: admin
KEYCLOAK_ADMIN: admin
KEYCLOAK_ADMIN_PASSWORD: admin
```

**CloudWatch Logging (Local):**
- Log Group: `/ec2-docker/api`, `/ec2-docker/products`, etc.
- Log Driver: `awslogs` (requires AWS credentials)

---

## 7. Tech Stack

### Backend Framework

- **Spring Boot 3.1.7:** REST API framework
- **Spring Cloud:** Microservices orchestration (Gateway, Eureka, Config, OpenFeign)
- **Spring Data JPA:** ORM for database operations
- **Spring Security:** Authentication & authorization
- **Spring Boot Actuator:** Health checks & metrics

### Service Architecture

- **API Gateway:** Spring Cloud Gateway with rate limiting
- **Service Discovery:** Netflix Eureka (automatic registration)
- **Config Management:** Spring Cloud Config Server
- **Inter-Service Communication:** OpenFeign (declarative HTTP client)
- **OAuth2/OIDC:** Keycloak integration for secure authentication

### Data Access

- **ORM:** Spring Data JPA with Hibernate
- **Database:** PostgreSQL (primary application data)
- **Auth Store:** MySQL 5.7 (Keycloak state)
- **Migrations:** Flyway/Liquibase ready

### Containerization & DevOps

- **Docker:** Multi-stage builds for all services
- **Docker Compose:** Local development orchestration
- **Container Registry:** AWS ECR (private)
- **Orchestration:** Kubernetes (EKS) for production
- **Terraform:** Infrastructure as Code via shopnow-infa

### Security & Scanning

- **OAuth2/OIDC:** Keycloak identity provider
- **API Security:** Spring Security with JWT tokens
- **Transport Security:** TLS 1.2+ with AWS ACM certificates
- **Secret Management:** Environment variables + AWS Secrets Manager
- **Vulnerability Scanning:** Can integrate Snyk/Trivy for CI/CD

### API Documentation

- **OpenAPI/Swagger:** SpringDoc OpenAPI starter (Springdoc-openapi)
- **Endpoint:** `/swagger-ui.html` (auto-generated API docs)
- **Postman Collection:** [Spring Boot Microservice.postman_collection.json](./Spring%20Boot%20Microservice.postman_collection.json)

---

## 8. API Documentation

### Available Endpoints

**API Gateway (5860):**
- `GET /api/products` - List all products
- `GET /api/products/{id}` - Get product details
- `POST /api/products` - Create product (admin only)
- `PUT /api/products/{id}` - Update product
- `DELETE /api/products/{id}` - Delete product

**User Service (5865):**
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - User login
- `GET /api/users/{id}` - Get user profile
- `PUT /api/users/{id}` - Update user

**Shopping Cart (5863):**
- `GET /api/cart` - View cart
- `POST /api/cart/add` - Add item to cart
- `DELETE /api/cart/remove/{itemId}` - Remove item
- `POST /api/cart/checkout` - Proceed to checkout

### Swagger UI

- **URL:** http://localhost:5860/swagger-ui.html
- **Auto-generated documentation** for all microservices

### Postman Collection

Import [Spring Boot Microservice.postman_collection.json](./Spring%20Boot%20Microservice.postman_collection.json) into Postman:

```bash
# Environment variables to set:
- base_url: http://localhost:5860
- keycloak_url: http://localhost:8080
- username: admin
- password: admin
```

---

## 9. Monitoring & Operations

### Local Logging

**CloudWatch Logs (docker-compose):**
- Log Group: `/ec2-docker/api`, `/ec2-docker/products`, etc.
- Requires AWS credentials in ~/.aws/credentials

**Docker Logs:**
```bash
docker-compose logs -f api-gateway
docker logs shopnow-backend-api-gateway-1 --tail 100
```

### Production Monitoring (EKS)

**CloudWatch Logs:**
- Log Group: `/prod/api-gateway`, `/prod/product-service`, etc.
- Auto-collected from container stdout/stderr

**Metrics:**
- Pod CPU/Memory via Kubernetes metrics-server
- Custom metrics via Spring Boot Actuator
- ALB/NLB target health

### Service Dependencies

```
API Gateway → Keycloak (OAuth2)
           → Product Service → PostgreSQL
           → User Service → PostgreSQL
           → Cart Service → PostgreSQL & Product/User

All Services → Discovery Server (Eureka)
            → Config Server
            → PostgreSQL (shared database)
```

---

## 10. Contact Information

**Author:** Bel7phegor (Nguyễn An Phúc)

- **Email:** [nguyenanphuc12032002@gmail.com](mailto:nguyenanphuc12032002@gmail.com)
- **LinkedIn:** [linkedin.com/in/nguyen-an-phuc](https://www.linkedin.com/in/nguyen-an-phuc/)
- **GitHub:** [@Bel7phegor](https://github.com/Bel7phegor)
- **Portfolio:** [anphuc.site](https://anphuc.site)

**Related Projects:**
- Frontend: [shopnow-frontend](https://github.com/Bel7phegor/shopnow-frontend) (React)
- Infrastructure: [shopnow-infa](https://github.com/Bel7phegor/shopnow-infa) (Terraform/AWS)
---

**Objective:** Build and maintain highly available, secure, and scalable microservices with automated deployment pipelines across development and production cloud environments.
