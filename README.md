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

**Overall Picture**

See the overall picture. There are 3 microservices develop and deploy together.


<img width="962" height="630" alt="Overall_Architecture" src="https://github.com/user-attachments/assets/ff99b636-9ce5-4020-82de-f41818574fb7" />

Here is a high-level architectural diagram of the final deployment:

<img width="1014" height="1274" alt="High Level diagram" src="https://github.com/user-attachments/assets/d9755fb8-03f2-48b8-82cb-4817ced75d90" />

**Step-by-Step Implementation**

**1. Containerizing the .NET Microservices**
   
The first step was to create a Dockerfile for each microservice. This ensures a consistent, isolated environment for our applications.

Example Dockerfile (for a typical .NET API):

# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

WORKDIR /src

COPY ["MyMicroservice/MyMicroservice.csproj", "MyMicroservice/"]

RUN dotnet restore "MyMicroservice/MyMicroservice.csproj"

COPY . .

WORKDIR "/src/MyMicroservice"

RUN dotnet build "MyMicroservice.csproj" -c Release -o /app/build

# Publish stage
      FROM build AS publish

      RUN dotnet publish "MyMicroservice.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Final stage
     FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final

     WORKDIR /app

     COPY --from=publish /app/publish .

     ENTRYPOINT ["dotnet", "MyMicroservice.dll"]

**2. Setting up the Azure Infrastructure**
   
I used the Azure CLI to provision the necessary resources.

Key Commands:

# Create a Resource Group

az group create \
   
     --name myaks-rg --location eastus

# Create an Azure Container Registry (ACR)

az acr create \
   
     --resource-group myaks-rg --name myacrMohit --sku Basic

# Create an AKS Cluster, integrating it with ACR for seamless image pulls

az aks create \

    --resource-group myaks-rg \
    
    --name myAksCluster \
    
    --node-count 2 \
    
    --generate-ssh-keys \
    
    --attach-acr myacrMohit

**3. Defining Kubernetes Manifests for Different Environments**
   
A key part of this project was structuring the Kubernetes manifests to support both local development and cloud deployment.

**Local Development (Docker Desktop)**

For local testing, I used a set of manifests in the /k8s directory. These were simple and pointed to images built locally.

Example: /k8s/product-deployment.yaml (Local)

**Production Deployment (AKS + ACR)**

For the automated Azure DevOps pipeline, I created a separate set of manifests in the /aks folder. 
The critical difference is that these manifests reference the container images stored in my Azure Container Registry (ACR).

Example: /aks/product-deployment.yaml (AKS/ACR)

**Key Changes for AKS/ACR Manifests:**

•	Image Reference: Changed from a local image name (product-service:latest) to the full ACR URI (myacrmohit.azurecr.io/product-service:$(Build.BuildId)).

•	Dynamic Tagging: Used a pipeline variable $(Build.BuildId) for immutable, traceable image tags instead of latest.

•	Secrets Management: Referenced Kubernetes Secrets (populated from Azure Key Vault) for sensitive data like database connection strings, instead of hardcoded or local values.

**4. Automating with Azure DevOps Pipelines**

The core of this project is the CI/CD pipeline defined in azure-pipelines.yml. This pipeline is specifically configured to use the manifests from the /aks directory.

**Continuous Integration (CI) Stage:**

1.	Trigger: On every push to the main branch.

2.	Build: Restores dependencies and builds the .NET solution.

3.	Test: Runs any unit tests.

4.	Push: Builds the Docker images, tags them with the $(Build.BuildId), and pushes them to Azure Container Registry (ACR).

**Continuous Deployment (CD) Stage:**

1.	Trigger: Automatically after a successful CI build.

2.	Deploy to AKS:

o	The pipeline task is configured to use the manifest files from the /aks directory.

o	It substitutes the $(Build.BuildId) token in the YAML files with the actual build number.

o	Runs kubectl apply -f /aks to deploy the latest images from ACR to the AKS cluster.

Check the Snippet from azure-pipelines.yml

**How to Run This Project**

You can find the complete source code, Dockerfiles, and Kubernetes manifests in this repository.

Prerequisites: Ensure you have the Azure CLI, kubectl, and Docker installed.

Clone the repository.

**Conclusion**

This project successfully demonstrates a modern, automated workflow for deploying .NET microservices. 
By leveraging the power of Azure Kubernetes Service (AKS) and the automation capabilities of Azure DevOps, we can achieve a robust, scalable, and maintainable deployment process.

I welcome any feedback, suggestions, or questions! Feel free to connect with me or open an issue in this repository.
