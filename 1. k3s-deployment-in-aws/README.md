# Deploying K3s on AWS EC2

In this lab, we will deploy a k3s kubernetes cluster on aws EC2.


<!-- This documentation provides a comprehensive guide on how to deploy a K3s Kubernetes cluster on AWS EC2, including detailed explanations of the key concepts and terms involved. -->

<!-- ## Table of Contents

1. [Introduction to AWS EC2](#introduction-to-aws-ec2)
2. [Setting Up the EC2 Instance](#setting-up-the-ec2-instance)
3. [Allocating and Associating an Elastic IP](#allocating-and-associating-an-elastic-ip)
4. [Accessing the EC2 Instance via SSH](#accessing-the-ec2-instance-via-ssh)
5. [Installing K3s on AWS EC2](#installing-k3s-on-aws-ec2)
6. [Dockerizing Your Application](#dockerizing-your-application)
7. [Pushing the Docker Image to DockerHub](#pushing-the-docker-image-to-dockerhub)
8. [Creating a K3s Kubernetes Cluster](#creating-a-k3s-kubernetes-cluster)
9. [Deploying Your Application](#deploying-your-application) -->

## Introduction to AWS EC2

Amazon Elastic Compute Cloud (EC2) is a service that provides scalable computing capacity in the cloud. It allows users to rent virtual computers on which to run their own applications. EC2 instances are like virtual servers that can run different operating systems, depending on your preference.

## Setting Up the VPC
<details>
  <summary>Click to expand</summary>

# AWS VPC and EC2 Setup Documentation

### Step 1: Create Your VPC
1. **Open the AWS Management Console** and search for "VPC" in the search bar.
2. On the left-hand side, click on "Your VPCs".
3. Click on "Create VPC" at the top right corner.

   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/create-vpc.jpeg)

4. Name your VPC using a tag.
5. Set the IPv4 CIDR block to `10.0.0.0/16`.

   Congratulations on creating your first VPC!

### Step 2: Create a Public Subnet
1. After creating your VPC, click on "Subnets" on the left-hand side.
2. Click on "Create Subnet".

   <!-- ![Create Subnet](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/Create-subnet.jpeg)

3. Designate the VPC you just created.
4. Assign a CIDR block within your VPC’s range (e.g., `10.0.1.0/24`).
5. Click on the created subnet and then "Edit subnet settings".
6. Enable "Auto-assign public IPv4 address" and save.

   <!-- ![Enable Auto-assign IPv4](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/edit-subnet-settings.jpeg)

### Step 3: Create and Attach an Internet Gateway
1. Click on "Internet Gateways" on the left-hand side.
2. Click "Create internet gateway".

   <!-- ![Create Internet Gateway](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/create-internet-gateway.jpeg)

3. Once created, click "Actions" and then "Attach to VPC".
4. Select your VPC and attach the Internet Gateway.

   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/Attach-to-VPC.jpeg)

### Step 4: Create Route Tables
1. Click on "Route Tables" on the left-hand side.
2. Click "Create route table".

   <!-- ![Create Route Table](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/Create-route-table.jpeg)

3. Associate the new route table with your VPC.
4. Add a route to allow internet traffic by specifying the destination `0.0.0.0/0` and target as your Internet Gateway.

   <!-- ![Add Route](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/Edit-routes.jpeg)

5. Click on the "Subnet Associations" tab, then "Edit Subnet Associations".
6. Select your public subnet and save.

   <!-- ![Associate Subnet](image) -->
   ![alt text](https://github.com/Konami33/MySQl-Database-Connection/raw/flask-mysql-aws/EC2-Setup/images/edit-subnet-associations.jpeg)

### Resource Map:

![](./image/14.png)

Now we have our vpc setup.
</details>

## Setting Up the EC2 Instance

### Choosing an OS Distribution

For this guide, we will use an `Ubuntu` machine. Ubuntu is a popular Linux distribution known for its ease of use and extensive community support.

### Creating an EC2 Instance

1. **Launch an EC2 Instance**: Go to the AWS Management Console, navigate to the EC2 dashboard, and launch an instance.
2. **Select an AMI (Amazon Machine Image)**: Choose an Ubuntu AMI.
3. **Instance Type**: Select a free tier-eligible instance type such as `t2.micro` or `t3.micro`.
4. **Configure Instance**:
    - Configure security groups to allow HTTP (port 80) and HTTPS (port 443) traffic.
    - Create and assign an SSH key pair to securely access your instance.

### Launch the Instance

Review your settings and launch the instance. Once the instance is running, note the public IP address.

## Allocating and Associating an Elastic IP

`Elastic IPs` are static IP addresses designed for dynamic cloud computing. They allow your instance to maintain the `same IP address` even after it is stopped and started.

1. **Allocate an Elastic IP**:
    - Navigate to the Elastic IPs section in the EC2 dashboard.
    - Click "Allocate new address" and follow the prompts to create an Elastic IP.

2. **Associate the Elastic IP**:
    - Select the newly created Elastic IP.
    - Click "Actions" > "Associate Elastic IP address".
    - Choose your instance and private IP address to associate.

![](./image/1.png)

## Accessing the EC2 Instance via SSH

To set up K3s on your EC2 instance, you need to access it via SSH. There are two main ways to do this:

### Using SSH Key

1. **SSH Command**: Use the following command to connect to your instance:

    ```sh
    ssh -i path_to_your_key.pem ubuntu@your_instance_ip
    ```

### Using AWS EC2 Instance Connect

**EC2 Instance Connect**:
- Select your instance in the AWS console.
- Click "Connect" and choose "EC2 Instance Connect".
- Open the terminal in your browser to access the instance.

For simplification we will use the AWS EC2 Instance Connect. 

## Installing K3s on AWS EC2

K3s is a lightweight Kubernetes distribution designed for resource-constrained environments.

### Install K3s

1. **Run the Installation Command**:

    ```sh
    curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
    ```
![](./image/2.png)

2. **Verify the Installation**:

    ```sh
    kubectl get nodes
    # Or
    kubectl cluster-info
    ```
![](./image/3.png)

So we have installed k3s on AWS EC2.

## Exaple task: Deploy a nginx web server on k3s kubenetes destribution.

<!-- ## Dockerizing Your Application

Docker is a platform for developing, shipping, and running applications inside containers.

### Install Docker

1. **Update and Install Docker**:

    ```sh
    sudo apt update -y && sudo apt install docker.io -y
    ```

2. **Add User to Docker Group**:

    ```sh
    sudo usermod -aG docker $USER
    newgrp docker
    ```

### Create Application Directory and Files

1. **Create Directory and Index File**:

    ```sh
    mkdir k3s-app
    cd k3s-app
    echo "<h1>Welcome to K3s Kubernetes Cluster on AWS EC2</h1>" > index.html
    ```

### Create Dockerfile

1. **Dockerfile**:

    ```Dockerfile
    FROM nginx:alpine
    COPY . /usr/share/nginx/html
    ```

### Build and Run Docker Image

1. **Build Docker Image**:

    ```sh
    docker build -t your_dockerhub_username/k3s-app:latest .
    ```

2. **Run Docker Image**:

    ```sh
    docker run -d -p 3000:80 your_dockerhub_username/k3s-app:latest
    ```

3. **Update Security Group**: Ensure port 3000 is open in your security group settings.

4. **Access Application**: Open the application in your browser using `http://your_elastic_ip:3000/`. -->

<!-- ## Pushing the Docker Image to DockerHub

DockerHub is a cloud-based registry service that allows you to link to code repositories, build your images, and test them.

1. **Login to DockerHub**:

    ```sh
    docker login
    ```

2. **Push the Image**:

    ```sh
    docker push your_dockerhub_username/k3s-app:latest
    ``` -->

## Creating a K3s Kubernetes Cluster

To run an application on K3s, you need to create Kubernetes deployment and service files.

### Create Kubernetes Deployment File

1. **Create `k3s-app.yml`**:

    ```sh
    vi k3s-app.yml
    ```

2. Add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k3s-app
  labels:
    app: k3s-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: k3s-app
  template:
    metadata:
      labels:
        app: k3s-app
    spec:
      containers:
        - name: k3s-app
          image: nginx:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: k3s-app-service
  name: k3s-app-service
spec:
  ports:
    - name: "3000-80"
      port: 3000
      protocol: TCP
      targetPort: 80
  selector:
    app: k3s-app
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k3s-app-ingress
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: k3s-app-service
                port:
                  number: 3000
```

### Explanation

1. **Deployment**:
   - The `Deployment` named `k3s-app` is updated to use the `nginx:latest` image.
   - It will create a single replica (`replicas: 1`) of the Nginx container.

2. **Service**:
   - The `Service` named `k3s-app-service` is set to expose port 3000 and maps it to port 80 of the Nginx container.
   - It uses a `ClusterIP` type service to expose the application internally within the cluster.

3. **Ingress**:
   - The `Ingress` named `k3s-app-ingress` is configured to route HTTP traffic to the `k3s-app-service` on port 3000.
   - The annotation `ingress.kubernetes.io/ssl-redirect: "false"` indicates that SSL redirection is disabled.

### Applying the Manifest

Save this configuration to a file named `nginx-deployment.yaml` and apply it using the following command:

```sh
kubectl apply -f nginx-deployment.yaml
```

### Verify the Deployment

To check if the deployment, service, and ingress are created successfully, you can use the following commands:

```sh
kubectl get deployments
kubectl get services
kubectl get ingress
```

![](./image/9.png)


## Deploying Your Application

To test the deployment, open your EC2 Elastic IP address in your browser. For example:

```sh
http://your_elastic_ip/
```

In our case `elastic_ip` is `3.231.129.241`

![](./image/13.png)

You should see your application running, indicating that the deployment was successful.

![](./image/11.png)

---

By following this documentation, you have successfully deployed a K3s Kubernetes cluster on AWS EC2 and run your application on it. This guide covers the setup of the EC2 instance, installation of K3s, Dockerizing and deploying an application, and configuring Kubernetes for production use.