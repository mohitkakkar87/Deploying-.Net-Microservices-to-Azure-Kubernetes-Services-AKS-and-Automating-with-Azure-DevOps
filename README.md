# Deploying-.Net-Microservices-to-Azure-Kubernetes-Services-AKS-and-Automating-with-Azure-DevOps
This repository documents my hands-on journey of deploying a .NET microservices-based application to Azure Kubernetes Service (AKS) and fully automating the build and deployment pipeline using Azure DevOps.
This project serves as a practical guide for developers and DevOps enthusiasts looking to understand the end-to-end process of containerizing, orchestrating, and continuously deploying .NET applications.

**Project Overview**
The application is a simplified e-commerce system built with a microservices architecture, inspired by the AspnetRun design. It consists of several independent services:
•	Product Service: Handles product information.
•	Order Service: Manages order processing.
•	Basket Service: Manages user shopping carts.
•	API Gateway (Ocelot): A single entry point for routing client requests to the appropriate microservice.

**Architecture & Technology Stack**
The entire setup is designed to be cloud-native and resilient.
•	Application Framework: .NET 8
•	Containerization: Docker
•	Orchestration: Azure Kubernetes Service (AKS)
•	Service Mesh & Ingress: NGINX Ingress Controller
•	Database: Azure SQL Database (external to the cluster)
•	CI/CD: Azure Pipelines (Azure DevOps)
•	Configuration & Secrets: Kubernetes ConfigMaps & Secrets, Azure Key Vault Provider
