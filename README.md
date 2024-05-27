# React-Webapp-on-Docker-and-Kubernetes
React-Webapp-on-Docker-and-Kubernetes


Setting Up Docker and Kubernetes for Nebula Cafe

Prerequisites
A computer running macOS, Linux, or Windows
Basic command-line knowledge

# Step 1: Install Docker
Download Docker Desktop:

Visit the Docker Desktop download page.
Download and install Docker Desktop for your operating system.
Start Docker Desktop:

Open Docker Desktop and ensure it is running.
Verify Docker Installation:
Open Terminal (or Command Prompt) and run:

bash
Copy code
docker --version

# Step 2: Create a Dockerfile for Nebula Cafe
Create a Dockerfile:
In the root directory of your nebula-cafe project, create a file named Dockerfile and add the following content:

dockerfile
Copy code
# Use an official node image as the base image
FROM node:16-alpine

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the working directory
COPY . .

# Build the React application
RUN npm run build

# Install a lightweight web server to serve the static files
RUN npm install -g serve

# Expose the port the app runs on
EXPOSE 3000

# Start the web server to serve the built files
CMD ["serve", "-s", "build", "-l", "3000"]

Build the Docker Image:

bash
Copy code
docker build -t nebula-cafe .
Run the Docker Container:

bash
Copy code
docker run -p 3000:3000 nebula-cafe

# Step 3: Push the Docker Image to Docker Hub
Log In to Docker Hub:

bash
Copy code
docker login
Enter your Docker Hub credentials.

Tag and Push the Docker Image:

bash
Copy code
docker tag nebula-cafe <your-dockerhub-username>/nebula-cafe:latest
docker push <your-dockerhub-username>/nebula-cafe:latest

# Step 4: Install Kubernetes and Minikube
Install kubectl:

bash
Copy code
brew install kubectl
Install Minikube:

bash
Copy code
brew install minikube
Start Minikube:

bash
Copy code
minikube start --nodes 4 --driver=docker

# Step 5: Deploy Nebula Cafe on Kubernetes
Create Kubernetes Deployment YAML File:
Create a file named deployment.yaml with the following content:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nebula-cafe-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nebula-cafe
  template:
    metadata:
      labels:
        app: nebula-cafe
    spec:
      containers:
      - name: nebula-cafe
        image: <your-dockerhub-username>/nebula-cafe:latest
        ports:
        - containerPort: 3000
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: NotIn
                values:
                - minikube-m03
Create Kubernetes Service YAML File:
Create a file named service.yaml with the following content:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: nebula-cafe-service
spec:
  selector:
    app: nebula-cafe
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
Apply the Deployment and Service Configuration:

bash
Copy code
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Step 6: Access the Kubernetes Dashboard
Deploy the Kubernetes Dashboard:

bash
Copy code
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml
Create a Service Account and ClusterRoleBinding:

bash
Copy code
kubectl create serviceaccount dashboard-admin-sa -n kubernetes-dashboard
kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin-sa
Get the Service Account Token:

bash
Copy code
kubectl create token dashboard-admin-sa -n kubernetes-dashboard
Start the Kubernetes Proxy:

bash
Copy code
kubectl proxy
Access the Dashboard:
Open your browser and navigate to:

bash
Copy code
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
Log In to the Dashboard:
Use the token generated in step 3 to log in.

# Step 7: Managing Pods
Add a New Pod:

Update the deployment.yaml file to increase the number of replicas.
Apply the changes:
bash
Copy code
kubectl apply -f deployment.yaml
Remove a Pod:

Update the deployment.yaml file to decrease the number of replicas.
Apply the changes:
bash
Copy code
kubectl apply -f deployment.yaml
Verify the Changes:

bash
Copy code
kubectl get pods -o wide
kubectl get nodes
kubectl get deployment nebula-cafe-deployment

By following these steps, you can set up Docker and Kubernetes for the nebula-cafe application, deploy it, and manage it using Kubernetes. This guide should help anyone, even beginners, to achieve the same setup. If you have any questions or encounter issues, feel free to reach out for further assistance.
