# Launching Mysql and Flask application on EC2 instance from two different public subnet

This documentation outlines the process to set up a VPC with two public subnets, corresponding route tables, and a network gateway. Further we launch the Mysql and Flask application on EC2 instance from two different public subnets.

## Scenario

![alt text](./images/Public-two.jpeg)

You are tasked with deploying a web application that uses Flask as the backend and MySQL as the database. To achieve this, you will create a Virtual Private Cloud (VPC) in AWS to ensure network isolation and security. Within this VPC, you will set up two public subnets: one for the Flask application and another for the MySQL database.

Each subnet will host an EC2 instance:
- The first instance will run the MySQL database, also containerized using Docker.
- The second instance will run the Flask application, containerized using Docker.


This architecture ensures that the application and database are isolated yet can communicate securely within the VPC. After setting up the instances and deploying the Docker containers, you will verify the connectivity between the instances to ensure seamless interaction between the Flask application and the MySQL database.


## Step-by-Step Guide to Create a VPC

### Step 1: Create the VPC

1. **Create a VPC:**
   - In the navigation pane, click on **Your VPCs**.
   - Click **Create VPC**.
   - Provide the following details:
     - **Name tag:** `my-vpc`
     - **IPv4 CIDR block:** `10.0.0.0/16`
     - **Tenancy:** Default
   - Click **Create VPC**.

### Step 2: Create Subnets

1. **Create Public Subnet 1:**
   - In the navigation pane, click on **Subnets**.
   - Click **Create subnet**.
   - Select your VPC from the **VPC ID** dropdown.
   - Provide the following details:
     - **Subnet name:** `public-subnet-1`
     - **Availability Zone:** `us-east-1a`
     - **IPv4 CIDR block:** `10.0.1.0/24`
   - Click **Create subnet**.

2. **Create Public Subnet 2:**
   - Click **Create subnet** again.
   - Select your VPC from the **VPC ID** dropdown.
   - Provide the following details:
     - **Subnet name:** `public-subnet-2`
     - **Availability Zone:** `us-east-1a`
     - **IPv4 CIDR block:** `10.0.2.0/24`
   - Click **Create subnet**.

### Step 3: Create Route Tables

1. **Create Public Route Table 1:**
   - In the navigation pane, click on **Route tables**.
   - Click **Create route table**.
   - Select your VPC from the **VPC ID** dropdown.
   - Provide the following details:
     - **Name tag:** `public-route-table-1`
   - Click **Create route table**.

2. **Create Public Route Table 2:**
   - Click **Create route table** again.
   - Select your VPC from the **VPC ID** dropdown.
   - Provide the following details:
     - **Name tag:** `public-route-table-2`
   - Click **Create route table**.

### Step 4: Create an Internet Gateway

1. **Create the Internet Gateway:**
   - In the navigation pane, click on **Internet Gateways**.
   - Click **Create internet gateway**.
   - Provide the following details:
     - **Name tag:** `my-gateway`
   - Click **Create internet gateway**.

2. **Attach the Internet Gateway to Your VPC:**
   - Select the internet gateway you just created.
   - Click **Actions**, then **Attach to VPC**.
   - Select your VPC from the **VPC ID** dropdown.
   - Click **Attach internet gateway**.

### Step 5: Update Route Tables

1. **Update Public Route Table 1:**
   - Select `public-route-table-1`.
   - Click on the **Routes** tab, then click **Edit routes**.
   - Click **Add route**.
   - Provide the following details:
     - **Destination:** `0.0.0.0/0`
     - **Target:** Select **Internet Gateway** and then select `my-gateway`.
   - Click **Save routes**.

2. **Update Public Route Table 2:**
   - Select `public-route-table-2`.
   - Click on the **Routes** tab, then click **Edit routes**.
   - Click **Add route**.
   - Provide the following details:
     - **Destination:** `0.0.0.0/0`
     - **Target:** Select **Internet Gateway** and then select `my-gateway`.
   - Click **Save routes**.

### Step 6: Associate Subnets with Route Tables

1. **Associate Public Subnet 1 with Public Route Table 1:**
   - Select `public-route-table-1`.
   - Click on the **Subnet associations** tab, then click **Edit subnet associations**.
   - Select `public-subnet-1`.
   - Click **Save associations**.

2. **Associate Public Subnet 2 with Public Route Table 2:**
   - Select `public-route-table-2`.
   - Click on the **Subnet associations** tab, then click **Edit subnet associations**.
   - Select `public-subnet-2`.
   - Click **Save associations**.

### Step 7: Launch an EC2 Instance in Public Subnet 1

1. **Launch an Instance:**
   - Click on **Instances** in the navigation pane.
   - Click **Launch Instances**.
   - Choose an Amazon Machine Image (AMI). Search for "Ubuntu" and select the latest Ubuntu Server LTS AMI.
   - Select an instance type, e.g., `t2.micro`.
   - Click **Next: Configure Instance Details**.

2. **Configure Instance Details:**
   - **Network:** Select `my-vpc`.
   - **Subnet:** Select `public-subnet-1`.
   - **Auto-assign Public IP:** Enable.
   - Click **Next: Add Storage**.

3. **Configure Security Group:**
   - Select **Create a new security group**.
   - Security group name: `public-sg-1`.
   - Add rules to allow SSH (port 22) and HTTP (port 80) access.
     - Type: `SSH`, Protocol: `TCP`, Port Range: `22`, Source: `0.0.0.0/0` (Anywhere) or restrict as needed.
     - Type: `All traffic`, Protocol: `All`, Port Range: `All`, Source: `0.0.0.0/0` (Anywhere).

    ![alt text](./images/inbound-rules.PNG)

   - Click **Review and Launch**.

4. **Review and Launch:**
   - Click **Launch**.
   - Select an existing key pair or create a new key pair, then click **Launch Instances**.
   - Click **View Instances** to see the status of your instance.
   - Click "Connect".

### Step 8: Launch an EC2 Instance in Public Subnet 2

1. **Launch an Instance:**
   - Launch another instance like **Step 7** above.

2. **Configure Instance Details:**
   - **Network:** Select `my-vpc`.
   - **Subnet:** Select `public-subnet-2`.
   - **Auto-assign Public IP:** Enable.
   - Click **Next: Add Storage**.

3. **Configure Security Group:**
   - Select **Create a new security group** or use an existing one.
   - Security group name: `public-sg-2`.
   - Add rules to allow SSH (port 22) and HTTP (port 80) access.
     - Type: `SSH`, Protocol: `TCP`, Port Range: `22`, Source: `0.0.0.0/0` (Anywhere) or restrict as needed.
     - Type: `All traffic`, Protocol: `All`, Port Range: `All`, Source: `0.0.0.0/0` (Anywhere).

    ![alt text](./images/inbound-rules-flask.PNG)

   - Click **Review and Launch**.

4. **Review and Launch:**
   - Click **Launch**.
   - Select an existing key pair or create a new key pair, then click **Launch Instances**.
   - Click **View Instances** to see the status of your instance.
   - Click "Connect".

## Resource Map

![alt text](./images/final-4.png)

![alt text](./images/final-5.png)

You have now successfully created a VPC with the following components:
- A VPC named `my-vpc`.
- Two public subnets (`public-subnet-1` and `public-subnet-2`).
- Two public route tables (`public-route-table-1` and `public-route-table-2`).
- An internet gateway named `my-gateway` attached to the VPC.
- Appropriate route configurations and subnet associations for routing internet traffic.
- You have successfully launched two Ubuntu EC2 instances in your VPC:
    - `public-instance-1` in `public-subnet-1` with a public IP address.
    - `public-instance-2` in `public-subnet-2` with a public IP address.


## Mysql Database Launch on EC2 Instance-1

### Step 1: Update the package list and install Docker

Install and set up the docker in ubuntu EC2 instance.


### Step 2: Create a database initialization script (init_db.sql)

   ```sql
   CREATE DATABASE IF NOT EXISTS test_db;
   USE test_db;

   CREATE TABLE IF NOT EXISTS users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(50) NOT NULL,
       email VARCHAR(50) NOT NULL
   );
   ```

### Step 3: Create a docker-compose.yaml file

   ```yaml
   version: '3.8'

   services:
     db:
       image: mysql:5.7
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: root
         MYSQL_USER: newuser
         MYSQL_PASSWORD: newpass
         MYSQL_DATABASE: test_db
       volumes:
         - ./init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
       ports:
         - "3306:3306"
   ```

### Step 4: Install Docker Compose

1. **Download the Docker Compose binary**:

   ```sh
   sudo yum update -y
   sudo yum install curl gnupg -y
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. **Apply executable permissions to the binary**:

   ```sh
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Verify the installation**:

   ```sh
   docker-compose --version
   ```


### Step 5: Build and Run Docker Containers

1. **Build and run the containers**:

   ```sh
   sudo docker-compose up --build -d
   ```
   - `-d`: Runs the containers in detached mode, meaning they run in the background.

### Step 7: Verify that the database and tables are set up correctly

1. **Install MySQL client**:
   ```sh
   sudo apt-get update
   sudo apt-get install mysql-client -y
   ```

2. **Get the MySQL Server container ID**:
   ```sh
   sudo docker ps
   ```
   - Look for the container ID of the MySQL service in the output.

3. **Get the IP address of the MySQL container**:
   ```sh
   sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id>
   ```
   - Replace `<container_id>` with the actual container ID from the previous step.
   - Note the IP address shown in the output.

4. **Log in to MySQL**:
   ```sh
   mysql -h <container_ip> -u newuser -pnewpass test_db
   ```
   - Replace `<container_ip>` with the IP address obtained in the previous step.
   - When prompted, enter the password `newpass`.

5. **Verify the database and tables**:
   ```sql
   SHOW DATABASES;
   USE test_db;
   SHOW TABLES;
   ```

6. **Perform basic database operations**:
   ```sql
   INSERT INTO users (name, email) VALUES ('Jane Doe', 'jane@example.com');
   SELECT * FROM users;
   ```

## Flask Application Launch on EC2 Instance-2

### 1. Set Up the Flask Application

Create a new directory for your project and navigate into it:

```sh
mkdir flask_mysql_docker
cd flask_mysql_docker
```

Create a file named app.py and add the following code:

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+mysqlconnector://newuser:newpass@10.0.2.141/test_db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    email = db.Column(db.String(50), nullable=False)

    def serialize(self):
        return {
            'id': self.id,
            'name': self.name,
            'email': self.email
        }

@app.route('/')
def index():
    return jsonify(message="Connected to MySQL database")

@app.route('/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify([user.serialize() for user in users])

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = User.query.get(user_id)
    if user:
        return jsonify(user.serialize())
    else:
        return jsonify({"error": "User not found"}), 404

@app.route('/users', methods=['POST'])
def add_user():
    new_user_data = request.get_json()
    new_user = User(name=new_user_data['name'], email=new_user_data['email'])
    db.session.add(new_user)
    db.session.commit()
    return jsonify({"id": new_user.id}), 201

@app.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return jsonify({"error": "User not found"}), 404
    update_data = request.get_json()
    user.name = update_data['name']
    user.email = update_data['email']
    db.session.commit()
    return jsonify({"message": "User updated successfully"})

@app.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return jsonify({"error": "User not found"}), 404
    db.session.delete(user)
    db.session.commit()
    return jsonify({"message": "User deleted successfully"})

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```


### 2. Create the Dockerfile

Create a file named Dockerfile and add the following content:

```Dockerfile
FROM python:3.8-slim-buster

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir Flask Flask-SQLAlchemy mysql-connector-python

EXPOSE 5000

CMD ["python", "app.py"]
```


### 3. Create the Docker Compose File

Create a file named docker-compose.yml and add the following content:

```yaml
version: '3.8'

services:
  web:
    build: .
    command: python app.py
    volumes:
      - .:/app
    ports:
      - "5000:5000"
    environment:
      FLASK_APP: app.py
      FLASK_RUN_HOST: 0.0.0.0
      DATABASE_URL: mysql+mysqlconnector://newuser:newpass@10.0.2.141/test_db
```

### 4. Build and Run the Docker Container

Ensure Docker and Docker Compose are installed on your EC2 instance. Then run the following commands:

```sh
docker-compose build
docker-compose up
```

This will build the Docker image and run the Flask application on port 5000.

## Testing the Connection

### 1. Obtain the IP Addresses

1. *Public IPs*: You can find the public IPs of both instances from the AWS Management Console under the EC2 Dashboard.
2. *Private IPs*: Similarly, the private IPs can be found in the same section.

### 2. Ping the Instances

#### From the MySQL Instance

1. *Ping the Flask Instance (Public IP)*:
    ```sh
    ping <Flask_Instance_Public_IP>
    ```

![alt text](./images/public-ping.PNG)
    

2. *Ping the Flask Instance (Private IP)*:
    ```sh
    ping <Flask_Instance_Private_IP>
    ```

![alt text](./images/private-ping.PNG)
    

#### From the Flask Instance

1. *Ping the MySQL Instance (Public IP)*:
    ```sh
    ping <MySQL_Instance_Public_IP>
    ```

2. *Ping the MySQL Instance (Private IP)*:
    ```sh
    ping <MySQL_Instance_Private_IP>
    ```

![alt text](./images/final3.png)

## Testing the API

1. **Check if the Flask API is running**:

   ```sh
   curl http://localhost:5000
   ```

2. **Add a user**:

   ```sh
   curl -X POST -H "Content-Type: application/json" -d '{"name": "Fazlul Karim", "email": "fazlulkarim@gmail.com"}' http://localhost:5000/users
   ```

3. **Get all users**:

   ```sh
   curl http://localhost:5000/users
   ```

4. **Get user by ID**:

   ```sh
   curl http://localhost:5000/users/1
   ```

5. **Update a user**:

   ```sh
   curl -X PUT -H "Content-Type: application/json" -d '{"name": "Fazlul", "email": "fazlulkarim362@gmail.com"}' http://localhost:5000/users/1
   ```

6. **Delete a user**:

   ```sh
   curl -X DELETE http://localhost:5000/users/1
   ```

   Expected Output:
    ![alt text](./images/final-1.png)

7. **Check that Mysql Database Table is updating**:

    Connect to your MySQL instance and check that the `users` table is being updated correctly.

    ![alt text](./images/users-table.PNG)

## Test Flask API from Web Browser

1. Open your web browser and enter `http://<your-ec2-public-ip>:5000`.
2. Verify the connection message confirming MySQL database access.
3. Test endpoints by accessing `http://<your-ec2-public-ip>:5000/users`.

Expected Output:

![alt text](./images/final.png)

## Issues

Launching a MySQL database in a public subnet on an EC2 instance involves several considerations and potential issues related to security, accessibility, and performance. Here are some of the key issues you might encounter and how to address them:

### 1. **Security Concerns**
- **Exposure to the Internet**: A public subnet means the EC2 instance will have a public IP address, making it accessible from the internet. This increases the risk of attacks.
  - **Solution**: Use security groups to restrict access to the MySQL port (default is 3306) to only trusted IP addresses. Consider using a VPN or SSH tunneling for secure access.

### 2. **Network Configuration**
- **Security Groups**: Ensure that the security group associated with the EC2 instance allows inbound traffic on port 3306 from trusted IP addresses.
  - **Solution**: Modify the security group rules to allow inbound traffic on port 3306 only from specific IP addresses or IP ranges.

- **Network ACLs**: Ensure that network ACLs associated with the public subnet allow traffic on port 3306.
  - **Solution**: Adjust network ACLs to permit inbound and outbound traffic on port 3306.

