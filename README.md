# RHB Lab - Spring Petclinic CI/CD

## Pipeline Flow
1. Push code to GitHub
2. Jenkins detects change via webhook
3. Jenkins builds Docker image
4. Jenkins pushes to Docker Hub
5. Jenkins deploys to EKS

## Services
- api-gateway: port 8080
- customers-service: port 8081
- vets-service: port 8083
- visits-service: port 8082
- discovery-server: port 8761
- config-server: port 8888
