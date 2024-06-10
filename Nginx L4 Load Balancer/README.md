# Deploy three EC2 instances, two hosting Flask applications, and one serving as an NGINX L4 load balancer


To implement an L4 load balancer using NGINX for our Flask API servers on AWS EC2 instances, we can follow these steps:

![alt text](<./image/WhatsApp Image 2024-06-11 at 02.01.56_b0610b73.jpg>)

At first, we need to create a `VPC` in AWS, configure subnet, route tables and gateway. 

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.17.00_d0cd50d3.jpg>)

Then we need to create 3 instances in `EC2`.

## Create EC2 Instances

### Create the Flask API Server EC2 Instances:

- Launch two EC2 instances (let's call them `flask-server-1` and `flask-server-2`).

- Choose an appropriate AMI (e.g., Ubuntu).

- Configure the instances with necessary security group rules to allow HTTP/HTTPS traffic (typically port 80/443).

- Assign a key pair for SSH access.

### Create the NGINX EC2 Instance:

- Launch another EC2 instance for the NGINX load balancer (let's call it `nginx-lb`).

- Configure the instance with a security group to allow incoming traffic on the load balancer port (typically port 80/443) and outgoing traffic to the Flask servers.

- Assign a key pair for SSH access.

Here are the instances:

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.16.59_2112dd93.jpg>)


## Set Up Flask API Servers 1

### Connect to the Flask Server Instances:

- Navigate to your instance in the AWS console.
- Click on `Connect`, select "EC2 Instance Connect", and open the terminal in your browser

### Install Flask and Create a Simple API:

- Run te following commands to execute command as `root` user:
    ```bash
    sudo su -
    ```

- Update the package lists and install `Python` and `flask`.
    ```sh
    apt update -y
    apt install python3 -y
    apt install python3-flask -y
    ```

- Create a simple Flask API.
    ```sh
    mkdir ~/flask_app
    cd ~/flask_app
    nano app.py
    ```

- Edit `app.py`:

     ```python
     from flask import Flask
     app = Flask(__name__)

     @app.route('/')
     def hello_world():
         return 'Hello, from flask app 1'

     if __name__ == '__main__':
         app.run(host='0.0.0.0')
     ```

- Run the Flask API.
     ```sh
     python3 app.py
     ```


Now we can access the `flask-server-1` from browser using public ip (in our case: `34.201.165.37`) and port `5000`. 

Note that, you may need to setup inbound rules to access the app from the browser.

Expected output:

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.08.25_c4d389e4.jpg>)



## Set Up Flask API Servers 2

Follow the same steps as above for flask app server 2.

- Here is the `app.py`:

    ```python
    from flask import Flask
    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        return 'Hello, from flask app 2'

    if __name__ == '__main__':
        app.run(host='0.0.0.0')
    ```

- Run the Flask API.
    ```sh
    python3 app.py
    ```

Now we can access the `flask-server-2` from browser using public ip (in our case: `18.235.255.23`) and port `5000`. 

Note that, you may need to setup inbound rules to access the app from the browser.

Expected output:

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.09.00_2b984360.jpg>)



## Step 3: Set Up NGINX as L4 Load Balancer

### Connect to the Flask Server Instances:

- Navigate to your `nginx-lb` instance in the AWS console.
- Click on `Connect`, select "EC2 Instance Connect", and open the terminal in your browser


### Install NGINX:

Nginx is very easy to install if we install it from a package manager like apt on Ubuntu or yum in CentOS. It is good for a general proposed load balancer, reverse proxy, and web server. But sometimes we need additional modules to add more function to Nginx that is not included in default installation from the package manager. If that is the case, you need to install the Nginx from source. In this tutorial, we will guide you step by step on how to install Nginx from source on ubuntu.

#### Sudo Privileges
Before starting, make sure that we have no permission issue on the installation & configuration.

```bash
sudo su
```

#### Install Dependencies
Run this command to install Nginx dependencies

```bash
apt update -y && apt-get install git build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev libgd-dev libxml2 libxml2-dev uuid-dev
```

#### Download Nginx Source Code
Before you download the Nginx source code, you can visit http://nginx.org/en/download.html to see the Nginx version available now. After that you can download them by running this command:

```bash
wget http://nginx.org/download/nginx-<version>.tar.gz
```
Now, the latest stable version is 1.20.2, so for me, I will download the nginx-1.20.2 version

```bash
wget http://nginx.org/download/nginx-1.20.2.tar.gz
```

Extract the downloaded file

```bash
tar -zxvf nginx-1.20.2.tar.gz
```

#### Build & Install Nginx
After extract the file, go to the nginx directory

```bash
cd nginx-1.20.2
```
Now is the time to configure Nginx that suits your need, this is where you put in the module you want to include in Nginx using the ./configure command. The full documentation is in here: Building Nginx from Sources. For now, I will give you the minimum configure option so you can build a good load balancer, reverse proxy, or webserver. Run this command to configure Nginx:

```bash
./configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/run/nginx.pid \
    --sbin-path=/usr/sbin/nginx \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_stub_status_module \
    --with-http_realip_module \
    --with-file-aio \
    --with-threads \
    --with-stream \
    --with-stream_ssl_preread_module
```

After that, run this command to build & install the Nginx

```bash
make && make install
```

To verify the installation, you can check the Nginx version

```bash
nginx -V
```



### Configure NGINX as an L4 Load Balancer
Open the NGINX configuration file.
```sh
sudo nano /etc/nginx/nginx.conf
```

Modify the configuration to set up the load balancer.
```nginx
events {
    worker_connections 1024;
}

stream {
    upstream flask_servers {
        server <flask-server-1-private-ip>:5000;
        server <flask-server-2-private-ip>:5000;
    }

    server {
        listen 80;
        proxy_pass flask_servers;
    }
}
```




### Create Systemd File
To make Nginx easier to manage, we can build a systemd file. First, create a new file in the systemd folder:

```bash
nano /lib/systemd/system/nginx.service
```
And then copy & paste this config to the file
```bash
[Unit]
Description=Nginx Custom From Source
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Reload the systemd:

```bash
systemctl daemon-reload
```

Enable Nginx service so it will auto start when the server boot

```bash
systemctl enable nginx
```

Now you can control Nginx using systemd, just like this:

```
service nginx start
service nginx stop
service nginx reload
service nginx restart
```






## Test the Load Balancer

### Access the Load Balancer:
- Open a web browser and navigate to the public IP address of the `nginx-lb` instance (in our case: `52.3.245.51`).
- You should see the response from one of your Flask API servers.

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.16.59_f11751aa.jpg>)

### Verify Round Robin:
- Refresh the page multiple times and you should see the responses alternating between the two Flask servers.

![alt text](<./image/WhatsApp Image 2024-06-11 at 01.16.59_f11751aa.jpg>)
![alt text](<./image/WhatsApp Image 2024-06-11 at 01.16.58_ba06a21c.jpg>)



By following these steps, you will set up an L4 load balancer with NGINX that distributes traffic to your Flask API servers using the round-robin method.

