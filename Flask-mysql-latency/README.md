# Flask MySQL Query Latency Measurement Using Prometheus and Grafana

This document will help us set up a Flask application to measure the latency of a MySQL query. 

## Prerequisites
- Python and pip installed on your system.
- Flask and MySQL connector libraries installed.
- MySQL server installed and running.
- Prometheus and Grafana installed and running.

## Start MySQL container in Docker

Running a MySQL Docker container involves using Docker commands to download the MySQL image and start a container based on that image. Below are the steps to achieve this:

### Step 1: Pull the MySQL Docker Image

First, you need to pull the MySQL Docker image from Docker Hub. Open your terminal and run the following command:

```bash
docker pull mysql:latest
```

This command will download the latest version of the MySQL image.

### Step 2: Run the MySQL Docker Container

After pulling the image, you can run the MySQL container. Use the following command:

```bash
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=my_db -p 3307:3306 -d mysql:latest
```

### Step 3: Verify the MySQL Container is Running

To verify that the MySQL container is running, use the following command:

```bash
docker ps
```

You should see an output listing the running containers, including `mysql-container`.

### Step 4: Connect to the MySQL Container

You can connect to the MySQL server running in the container using a MySQL client. For example, you can use the MySQL command-line client:

```bash
docker exec -it mysql-container mysql -u root -proot
```

Enter the root password you set earlier when prompted.

### Step 5: Create "users" Table and add users


Use the following commands to create table:

```sql
USE my_db;

DROP TABLE IF EXISTS users;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Add users in the table: 

```sql
INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');

INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');

INSERT INTO users (username, email) VALUES ('user3', 'user3@example.com');
```

Use the following commands to see the created users:

```sql
SELECT * FROM users;
```

Expected result:

```bash
mysql> DROP TABLE IF EXISTS users;
Query OK, 0 rows affected (0.02 sec)

mysql> SELECT * FROM users;
ERROR 1146 (42S02): Table 'my_db.users' doesn't exist
mysql>
mysql> CREATE TABLE users (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     username VARCHAR(50) NOT NULL,
    ->     email VARCHAR(50) NOT NULL,
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql>
mysql> SELECT * FROM users;
Empty set (0.00 sec)

mysql> INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO users (username, email) VALUES ('user3', 'user3@example.com');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM users;
+----+----------+-------------------+---------------------+
| id | username | email             | created_at          |
+----+----------+-------------------+---------------------+
|  1 | user1    | user1@example.com | 2024-06-11 17:11:00 |
|  2 | user2    | user2@example.com | 2024-06-11 17:11:08 |
|  3 | user3    | user3@example.com | 2024-06-11 17:11:12 |
+----+----------+-------------------+---------------------+
3 rows in set (0.00 sec)

```

## Setup Flask App

### Setup virtual environment:
```bash
python -m venv venv
venv/Scripts/activate
```

### Install required packages

```bash
pip install Flask prometheus_client mysql-connector-python
```

### Create the Flask app (app.py)

```python
from flask import Flask
import mysql.connector
import time
from prometheus_client import start_http_server, Summary
from prometheus_client import Counter, generate_latest

app = Flask(__name__)

# Create a metric to track time spent and requests made
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')
REQUEST_COUNTER = Counter('request_processing_count', 'Number of requests processed')

@app.route('/metrics')
def metrics():
    return generate_latest()

def get_db_connection():
    connection = mysql.connector.connect(
        host='localhost',
        user='root',
        password='root',
        database='my_db'
    )
    return connection

@REQUEST_TIME.time()
@app.route('/')
def query():
    start_time = time.time()
    REQUEST_COUNTER.inc()
    connection = get_db_connection()
    cursor = connection.cursor()
    cursor.execute("SELECT * FROM users")
    result = cursor.fetchall()
    cursor.close()
    connection.close()
    end_time = time.time()
    latency = end_time - start_time
    REQUEST_TIME.observe(latency)
    return f"Query latency: {latency} seconds. Result: {result}"

if __name__ == '__main__':
    # Start up the server to expose the metrics.
    start_http_server(8000)
    # Run the Flask app
    app.run(port=5000)

```

### Run the Flask app

```bash
python app.py
```



## Set Up Prometheus

### Configure Prometheus (prometheus.yml)
Create or modify the Prometheus configuration file to scrape the Flask application's metrics endpoint.

```yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'flask_app'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['localhost:8000']
```

### Run Prometheus

```bash
./prometheus --config.file=prometheus.yml
```

## Set Up Grafana

### Configure Grafana

- Open Grafana in your web browser (default: http://localhost:3000).
- Log in (default credentials: admin/admin).

### Add Prometheus as a data source:

- Go to Configuration -> Data Sources -> Add data source.
- Select Prometheus.
- Set the URL to http://localhost:9090 (default Prometheus address).
- Click Save & Test.

### Create a Dashboard
- Go to Create -> Dashboard.
- Add a new panel.
- In the panel configuration, select Prometheus as the data source.
- Use the metric request_processing_seconds to visualize the latency.

## Varification

### Make a few requests to the Flask app
- Open http://localhost:5000 in browser.


### Check Prometheus
- Open http://localhost:9090 in your browser.
- Query request_processing_seconds to verify that the metrics are being scraped.

### Check Grafana
- Open your Grafana dashboard.
- Ensure that the panel displays the latency of the MySQL queries.

Expected results:

![alt text](<./img1.jpg>)

![alt text](<./img2.jpg>)


By following these steps, you will have a fully functioning monitoring setup for the latency of MySQL queries in a Flask application using Prometheus and Grafana without using Docker containers.