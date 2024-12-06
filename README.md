# EKS-and-Monitoring-with-OpenTelemetry

- Github Link: https://github.com/open-telemetry/opentelemetry-demo
- Documentation link: https://opentelemetry.io/docs/demo/

## Phase 1: Environment and Initial Application Setup

### Docker Deployment

- Set up a dedicated EC2 instance (minimum m5.large instance with 16GB storage) to
install and configure Docker.
- Use the docker-compose.yml file available in the GitHub repository to bring up the
application.
- Validate that all services defined in the docker-compose.yml are up and running.
Ensure this by:
  - Running Docker commands like docker ps and docker-compose logs.
  - Accessing application endpoints (if applicable) and confirming service
functionality.
- Once the deployment is validated, delete the EC2 instance to clean up resources.

### Deliverables
1. Screenshot of the EC2 instance details (instance type, storage, etc.).
   ![image](https://github.com/user-attachments/assets/fabeb0ab-9960-4da2-9042-93d4632d7b03)

2. Screenshots of the services running (docker ps).
   ```
   # Update the system
   sudo apt update -y

   # Install dependencies
   sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

   # Add Docker’s official GPG key
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  
   # Set up the stable repository
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
   # Update the package index again
   sudo apt update -y
  
   # Install Docker
   sudo apt install docker-ce docker-ce-cli containerd.io -y
  
   # Verify Docker installation
   sudo docker --version
   
   # Download Docker Compose
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

   # Set executable permissions
   sudo chmod +x /usr/local/bin/docker-compose

   # Verify installation
   docker-compose --version

   # Install Git if not already installed
   sudo apt install git -y

   # Clone the repository
   git clone https://github.com/open-telemetry/opentelemetry-demo.git

   # Navigate to the project directory
   cd opentelemetry-demo

   # Bring up the services defined in the docker-compose.yml file
   sudo docker-compose up --force-recreate --remove-orphans --detach


   ```
   
3. Screenshots of Docker logs showing the application's startup sequence.
   ![image](https://github.com/user-attachments/assets/6ecf4596-4b3f-48c8-8936-006e5327d9ac)

   ![image](https://github.com/user-attachments/assets/429d098f-8489-4bae-a08a-993c3b6b33a5)
   ![image](https://github.com/user-attachments/assets/8412a767-3aed-4e3f-a300-6c6ef7dc7b3e)
   ![image](https://github.com/user-attachments/assets/f05bcd82-dc9a-4ec0-85ae-7c4a87c7e8d8)


4. Screenshot of accessible application endpoints.
   - ### Web store:
     ![image](https://github.com/user-attachments/assets/60bcd2c4-4064-43bb-98cb-16d70fdccf79)

   - ### Grafana:
     ![image](https://github.com/user-attachments/assets/9a27ed2e-fff2-42be-9e0c-54492a28a636)
  
   - ### Load Generator UI:
   - ![image](https://github.com/user-attachments/assets/c2d8e46c-ad45-4cd4-b7f5-6b070e2672b4)
  
   - ### Jaeger UI :
   - ![image](https://github.com/user-attachments/assets/bac3f795-b029-42d9-b05e-226cd08d2783)

   - ### Flagd configurator UI:
     ![image](https://github.com/user-attachments/assets/417597d1-879e-49d4-9f09-1fb376b8bef1)




### Kubernetes Setup
- Set up an EKS Cluster in AWS with:
  - At least 2 worker nodes.
  - Instance type: m5.large for worker nodes.
    
- Deploy the application to the EKS cluster using the provided opentelemetry-demo.yaml manifest file.
  
- Use a dedicated EC2 instance as the EKS client to:
  - Install and configure kubectl, AWS CLI, and eksctl for cluster management.
  - Run all Kubernetes commands from the EC2 instance (not from a local machine).
    
- Validate the deployment:
  - Ensure all pods and services are running as expected in the otel-demo namespace.
  - Access application endpoints through port-forwarding or service URLs.
  - Collect the cluster details, including node and pod status.
    
- Do not delete the EKS cluster unless explicitly stated.

### Deliverables
- #### Prerequisites:
  1. AWS Account with necessary permissions (IAM role with EKS, EC2, and other related service permissions).
  2. EC2 Instance (m5.large) to act as the EKS client.
  3. `AWS CLI`, `kubectl`, `eksctl` installed on the EC2 instance.
 
- Install AWS CLI: Run the following commands to install AWS CLI v2:
    ```
    bash
    Copy code
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    aws --version  # Check if AWS CLI is installed
    aws configure
    ```
- Install kubectl:
  ```
  curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl
  sudo mv kubectl /usr/local/bin/
  kubectl version --client  # Verify kubectl installation
  ```

- Install eksctl:
  ```
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/v0.138.0/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  eksctl version  # Verify eksctl installation
  ```

## 1. Setting up EKS Cluster:
```
eksctl create cluster \
  --name opentelemetry-demo-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type m5.large \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed
```

kubectl get nodes
kubectl get svc

## 2. Deploy the OpenTelemetry Demo Application to EKS (refer documentation):
```
kubectl apply --namespace otel-demo -f https://raw.githubusercontent.com/open-telemetry/opentelemetry-demo/main/kubernetes/opentelemetry-demo.yaml

kubectl get pods -n otel-demo
kubectl get svc -n otel-demo

```

## 3. Port Forwarding 
```
kubectl port-forward svc/opentelemetry-demo-frontendproxy 8080:8080 -n otel-demo --address 0.0.0.0

kubectl port-forward svc/opentelemetry-demo-grafana 8080:80 -n otel-demo --address 0.0.0.0
```
    
1. Screenshot of the EKS cluster configuration details (number of nodes, instance type,
etc.).
![image](https://github.com/user-attachments/assets/bd1ec0d4-5c20-45fa-b7f0-45db77667999)
![image](https://github.com/user-attachments/assets/cd7ac5d3-fe63-483c-9e22-0abcddb446cd)

2. Screenshot of the EC2 instance used as the EKS client (instance type, storage, etc.).
 ![image](https://github.com/user-attachments/assets/c3aa3590-665e-42c8-97a2-88054dcc673c)

3. Screenshot of kubectl get all -n otel-demo showing the status of pods, services, and deployments.
   ![image](https://github.com/user-attachments/assets/d532560c-0496-4593-9026-19be721338ed)

4. Screenshot of logs from key application pods to confirm successful startup.
   ```
   kubectl logs opentelemetry-demo-frontendproxy -n otel-demo --tail=50
   ```
   ![image](https://github.com/user-attachments/assets/02b78de9-7031-4ea0-ae37-76452851742e)

5. Exported Kubernetes manifest (opentelemetry-demo.yaml), if modified.
   ```
   kubectl get deployment -n otel-demo opentelemetry-demo-frontendproxy -o yaml > opentelemetry-demo.yaml
   ```
   ![image](https://github.com/user-attachments/assets/a521e6fa-152a-427a-983c-a365d7714238)

6. Screenshot of accessible application endpoints. follow the project documentation link.

![image](https://github.com/user-attachments/assets/42a70e56-545f-4b8a-b48e-15f3c8b190d0)
![image](https://github.com/user-attachments/assets/401f5b15-e036-4b83-8f68-699407caa78d)

![image](https://github.com/user-attachments/assets/1218c524-501b-44a8-83f3-c7bbf6855b98)

![image](https://github.com/user-attachments/assets/ee78daf4-6da2-449e-b4c6-cef5f6808c25)

   
