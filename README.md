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

Step-by-Step Implementation

1. Containerizing the .NET Microservices
2. 
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

