# Best Buy Cloud-Native Application

This repository contains a cloud-native, microservices-based demo application for Best Buy’s online store, inspired by the Algonquin Pet Store (On Steroids) architecture. The application demonstrates browsing products, placing orders, and managing product inventory. It also integrates with AI services for generating product descriptions and images.

## Updated Architecture Diagram

![image](https://github.com/user-attachments/assets/4ce7bd0c-e707-485c-bea4-3f31589f2c6c)


# Connecting Order-Service to Azure Service Bus

This guide walks through the steps to connect the `order-service` microservice to Azure Service Bus using either **Managed Identity** or **Shared Access Policy** authentication.

---

## ✅ Step 1: Create Azure Resources

```bash
az group create --name BestBuyRg --location canadacentral

az servicebus namespace create \
  --name BestBuyNamespace \
  --resource-group BestBuyRg

az servicebus queue create \
  --name orders \
  --namespace-name BestBuyNamespace \
  --resource-group BestBuyRg

```

## Used Architecture Diagram

![bestar](https://github.com/user-attachments/assets/670e611f-9e66-43a8-a534-d46136212ed9)

**Key Components:**
- **Store-Front:** Customer-facing UI for browsing products and placing orders.
- **Store-Admin:** Employee-facing UI for managing product catalog (CRUD) and viewing orders.
- **Order-Service:** Accepts incoming orders from Store-Front, sends order data to Azure Service Bus queue.
- **Product-Service:** Manages product data, integrates with AI-Service for product descriptions and images.
- **Makeline-Service:** Listens to Azure Service Bus, processes orders, updates MongoDB with order statuses.
- **AI-Service:** Uses GPT-4 and DALL-E (via OpenAI or Azure OpenAI) for generating product descriptions and images.
- **MongoDB:** Persists product and order data.
- **rabbitMq:** RabbitMQ for an order queue.

## Application Functionality

1. **Customers (Store-Front):**
   - View a list of products with AI-generated descriptions and images.
   - Add products to the cart and place orders.

2. **Admins (Store-Admin):**
   - Create, read, update, and delete products.
   - Trigger AI-Service calls to generate new product descriptions and images.
   - View incoming and processed orders.

3. **Order Processing (Order-Service & Makeline-Service):**
   - Order-Service sends new orders to Azure Service Bus.
   - Makeline-Service listens to the queue, processes orders, and updates their status in MongoDB.

4. **AI Integration (AI-Service):**
   - GPT-4: Generates marketing-friendly product descriptions.
   - DALL-E: Generates product images based on the product description keywords.

## Deployment Instructions

### Prerequisites

- **Kubernetes Cluster (AKS recommended)**
  - Ensure `kubectl` is configured to communicate with your cluster.
- **OpenAI / Azure OpenAI**
 Set Up the AI Backing Services
To enable AI-generated product descriptions and image generation features, you will deploy the required **Azure OpenAI Services** for GPT-4 (text generation) and DALL-E 3 (image generation). This step is essential to configure the **AI Service** component in the Best buy Store application.
## Steps to Deploy

 **Clone this repository.**
 
###  Retrieve and Configure API Keys

1. **Get API Keys**:
   - Go to the **Keys and Endpoints** section of your Azure OpenAI resource.
   - Copy the **API Key (API key 1)** and **Endpoint URL**.

2. **Base64 Encode the API Key**:
   - Use the following command to Base64 encode your API key:
     ```bash
     echo -n "<your-api-key>" | base64
     ```
   - Replace `<your-api-key>` with your actual API key.

---

### Update AI Service Deployment Configuration in the `Deployment Files` folder.
1. **Modify Secretes YAML**:
   - Edit the `secrets.yaml` file.
   - Replace `OPENAI_API_KEY` placeholder with the Base64-encoded value of the `API_KEY`. 
2. **Modify Deployment YAML**:
   - Edit the `ai-service.yaml` file.
   - Replace the placeholders with the configurations you retrieved:
     - `AZURE_OPENAI_DEPLOYMENT_NAME`: Enter the deployment name for GPT-4.
     - `AZURE_OPENAI_ENDPOINT`: Enter the endpoint URL for the GPT-4 deployment.
     - `AZURE_OPENAI_DALLE_ENDPOINT`: Enter the endpoint URL for the DALL-E 3 deployment.
     - `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME`: Enter the deployment name for DALL-E 3.

   Example configuration in the YAML file:
   ```yaml
   - name: AZURE_OPENAI_API_VERSION
     value: "2024-07-01-preview"
   - name: AZURE_OPENAI_DEPLOYMENT_NAME
     value: "gpt-4-deployment"
   - name: AZURE_OPENAI_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_DEPLOYMENT_NAME
     value: "dalle-3-deployment"
   ```

2. **Set up ConfigMaps and Secrets:**
   ```bash
   kubectl apply -f Deployment_Files/config-maps.yaml
   kubectl apply -f Deployment_Files/secrets.yaml


### Deploy the application:
Deploy all the yaml files from the Deployment Files folder one by one
```bash
kubectl apply -f Deployment_Files/store-front.yaml
kubectl apply -f Deployment_Files/store-admin.yaml
kubectl apply -f Deployment_Files/product-service.yaml
kubectl apply -f Deployment_Files/order-service.yaml
kubectl apply -f Deployment_Files/ai-service.yaml
kubectl apply -f Deployment_Files/mongodb.yaml
kubectl apply -f Deployment_Files/make-line.yaml
kubectl apply -f Deployment_Files/rabbitmq.yaml
kubectl apply -f Deployment_Files/admin-tasks.yaml
````
## Validate the deployment:
```bash
kubectl get pods
kubectl get services

````


1. **Table for Microservice Repository**:
   - Visit the GitHub repositories for the services listed below and fork them into your own GitHub account:
   
   | Service            | Description                                | Github Repo                                                                    |
   |--------------------|--------------------------------------------|--------------------------------------------------------------------------------|
   | `store-front`      | Web app for customers to place orders      | [store-front](https://github.com/mspanwar21/store-front)                       |
   | `store-admin`      | Web app for store employees                | [store-admin](https://github.com/mspanwar21/store-admin)                       |
   | `order-service`    | Handles order placement                    | [order-service](https://github.com/mspanwar21/order-service)                   |
   | `product-service`  | Handles CRUD operations on products        | [product-service](https://github.com/mspanwar21/product-service)               |
   | `makeline-service` | Processes and completes orders             | [makeline-service](https://github.com/mspanwar21/makeline-service)             |
   | `ai-service`       | AI-based product descriptions and images   | [AI-service](https://github.com/mspanwar21/AI-service)                         |
   | `virtual-customer` | Simulates customer order creation          | [virtual-customer](https://github.com/mspanwar21/virtual-customer)             |
   | `virtual-worker`   | Simulates order completion                 | [virtual-worker](https://github.com/mspanwar21/virtual-worker)                 |
## Implement a CI/CD Pipeline for Each Microservice
1. **Fork the all the above Repositories**
2. 2. **Set Up Secrets**
   - Go to **Settings > Secrets and variables > Actions** in each forked repository.
   - Add the following repository secrets:
     - `DOCKER_USERNAME`: Your Docker Hub username.
     - `DOCKER_PASSWORD`: Your Docker Hub password.
     - `KUBE_CONFIG_DATA`: Base64-encoded content of your Kubernetes configuration file (`kubeconfig`). This is used for authentication with your Kubernetes cluster.
       - Run the following commands to get `KUBE_CONFIG_DATA` after connecting to your AKS cluster:
     ```bash
     cat ~/.kube/config | base64 -w 0 > kube_config_base64.txt
     ``` 
     Use the content of this file as the value for the KUBE_CONFIG_DATA secret in GitHub.
     3. **Set Up Environment Variables (Repository Variables)**
   - Go to **Settings > Secrets and variables > Actions** in each forked repository.
   - Add the following repository variables:
     - `DOCKER_IMAGE_NAME`: The name of the Docker image to be built, tagged, and pushed (For example: `store-front-l9`).
     - `DEPLOYMENT_NAME`: The name of the Kubernetes deployment to update (For example: `store-front`).
     - `CONTAINER_NAME`: The name of the container within the Kubernetes deployment to update (For example: `store-front`).
     
---

## Step 2: Create the Workflow File
1. **Create the Workflow Directory**:
   - In each forked repository, create a directory named `.github/workflows/`.

2. **Add the Workflow File**:
   - Copy the `ci_cd.yaml` file from the `Workflow Files` folder into the `.github/workflows/` directory of each forked repository.
---

# Table of Docker Images

| Service           | Docker Image Link   |
|-------------------|-----------------------------------------------------------------------------|
| Store-Front       | [DOCKER HUB LINK](https://hub.docker.com/r/mohitsp21/store-front)           |
| Store-Admin       | [DOCKER HUB LINK](https://hub.docker.com/r/mohitsp21/store-admin)           |
| Order-Service     | [DOCKER HUB LINK](https://hub.docker.com/r/mohitsp21/order-service)         |
| Product-Service   | [Docker HUB LINK](https://hub.docker.com/r/mohitsp21/product-service)       |
| Makeline-Service  | [Docker Hub LINK](https://hub.docker.com/r/mohitsp21/makeline-service/tags) |
| AI-Service        | [Docker HUB LINK](https://hub.docker.com/r/mohitsp21/ai-service)            |
| Vitrual-customer  | [DOCKER HUB LINK ](https://hub.docker.com/r/mohitsp21/virtual-customer)     |
| Vitrual- worker   | [DOCKER HUB LINK ](https://hub.docker.com/r/mohitsp21/virtual-worker)       |

### Problems

## AI Integration Issue

Despite multiple attempts, I was unable to successfully deploy and integrate the AI services due to the following reasons:

1. Region Limitations:  
   The Azure OpenAI service was not available in the default region of my subscription. I explored and tested all available regions where GPT-4 and DALL·E might be accessible, but none were supported under my account at the time.

2. Quota Limitations:  
   My Azure subscription had **exceeded the usage quota** and **credits were exhausted**, preventing further access or deployment of any AI resources.

3. **Service Deprecation:**  
   I attempted to use **DALL·E 2** as a fallback option, but it had already been **retired**, and the newer **DALL·E 3** required specific configuration and access which was unavailable due to the above limitations.

>  *Although the AI-Service component is structured and ready in the deployment, the actual AI generation functionality could not be demonstrated due to these constraints.*

##  Attempted Integration with Azure Service Bus

One of the challenges faced during the development of this application was attempting to replace RabbitMQ with **Azure Service Bus** to align the architecture more closely with cloud-native messaging services. I followed the standard integration steps:

1. Created a **Service Bus namespace** and a **queue** in the Azure Portal.  
2. Updated the application configuration to include the Service Bus connection string and queue name.  
3. Attempted to use the `Azure.Messaging.ServiceBus` SDK to publish and consume messages within the `Order-Service`.

However, integration issues arose due to the following reasons:
- The **Service Bus client library requires asynchronous programming patterns**, which conflicted with some parts of the existing synchronous RabbitMQ logic.
- **Debugging was further complicated by the lack of local emulation support for Azure Service Bus**, unlike RabbitMQ which can be easily run locally using Docker.

To elaborate on the local debugging challenge:
- I initially attempted to **debug the Azure Service Bus integration locally** by looking for a local emulator, but **Azure does not provide an official emulator for Service Bus**.
- I tried using [Azurite](https://github.com/Azure/Azurite), Azure's official emulator, only to find that it supports **Blob, Queue, and Table storage**, not Service Bus.
- I also explored third-party Docker images and tools that claimed to emulate Service Bus behavior, but they lacked proper support and didn’t integrate well with the Azure SDK.
- I attempted to use the **ServiceBusProcessor** for local testing, but it required a live Azure Service Bus instance, which introduced latency and made offline development impossible.

I reverted to using **RabbitMQ**, which allowed quick local setup and testing via Docker. This ensured I could demonstrate the core functionality of order communication and processing between services effectively.


# Demo video
[Click_Here_to_Watch_Video](https://youtu.be/I_OcTkZDYAE)

