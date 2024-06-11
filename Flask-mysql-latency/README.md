# Flask MySQL Query Latency Measurement

This guide will help you set up a Flask application to measure the latency of a MySQL query. 

## Prerequisites

- Python installed on your system.
- MySQL database installed and running.
- Flask and MySQL connector libraries installed.

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



## Installation

1. **Clone the repository (if applicable) or create a new project directory:**

   ```bash
   mkdir flask-mysql-latency
   cd flask-mysql-latency
   ```

2. **Create a virtual environment (optional but recommended):**

   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows use `venv\Scripts\activate`
   ```

3. **Install the required Python packages:**

   ```bash
   pip install flask mysql-connector-python
   ```






## Configuration

1. **Create a file named `config.py` and add your MySQL database configuration:**

   ```python
   # config.py
   DB_CONFIG = {
       'user': 'your_username',
       'password': 'your_password',
       'host': 'localhost',
       'database': 'your_database'
   }
   ```
   
   In my case I am using root user and password:

    ```python
   # config.py
   DB_CONFIG = {
       'user': 'root',
       'password': 'root',
       'host': 'localhost',
       'database': 'your_database'
   }
   ```


2. **Replace `your_username`, `your_password`, `localhost`, and `your_database` with your actual MySQL credentials.**

## Implementation

1. **Create a file named `app.py` and add the following code:**

   ```python
   # app.py
   from flask import Flask, jsonify
   import mysql.connector
   import time
   from config import DB_CONFIG

   app = Flask(__name__)

   @app.route('/')
   def query_latency():
       try:
           # Connect to the MySQL database
           conn = mysql.connector.connect(**DB_CONFIG)
           cursor = conn.cursor()

           # Start timing
           start_time = time.time()

           # Execute the query
           cursor.execute("SELECT * FROM users")

           # Fetch the results (optional, depending on what you want to measure)
           results = cursor.fetchall()

           # End timing
           end_time = time.time()

           # Calculate latency
           latency = end_time - start_time

           # Close the cursor and connection
           cursor.close()
           conn.close()

           return jsonify({
               'latency': latency,
               'results': results  # You can remove this if you don't want to return the results
           })

       except mysql.connector.Error as err:
           return jsonify({'error': str(err)}), 500

   if __name__ == '__main__':
       app.run(debug=True)
   ```


## Running the Application

1. **Run the Flask application:**

   ```bash
   python app.py
   ```

2. **You should see output indicating that the Flask server is running:**

   ```
   * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
   ```

## Testing the Endpoint

1. **Open a web browser or use a tool like `curl` or Postman to test the endpoint:**

   ```bash
   http://127.0.0.1:5000/
   ```

2. **You should receive a JSON response with the query latency:**

   ```json
   {
       "latency": 0.123456,
       "results": [...]
   }
   ```

## Notes

- **Database Configuration**: Ensure your MySQL database is running and accessible.
- **Timing the Query**: The `time.time()` function is used to measure the start and end times of the query execution.
- **Error Handling**: If there's an error with the database connection or query execution, the application will return a JSON response with the error message and a 500 status code.
- **Virtual Environment**: Using a virtual environment is recommended to manage dependencies and avoid conflicts with other projects.

---

This documentation provides a step-by-step guide to set up and run a Flask application that measures the latency of a MySQL query. Follow the instructions carefully to ensure a successful setup.