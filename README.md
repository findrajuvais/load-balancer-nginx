# Load balancer using Nginx:
A load balancer is a device or software application that distributes incoming network traffic across multiple servers. This ensures that no single server is overwhelmed, improving the reliability, scalability, and availability of applications or services.

In the context of Docker and Nginx, you can set up a load balancer that distributes HTTP(S) or WebSocket traffic across multiple backend services or servers.

### Steps to Install Nginx Using Docker without ssl:
#### For ubuntu:
```
sudo apt update
sudo apt install docker.io
```

#### For RHEL/CentOS:
```
sudo yum install -y docker
```

##### Start and enable Docker if it's not running already:
```
sudo systemctl start docker
sudo systemctl enable docker
```

### Create nginx.conf file:
```
# Main configuration block
worker_processes 1;

# Events block: Defines how NGINX handles connections
events {
    worker_connections 1024;  # Max number of connections per worker
    multi_accept on;  # Allow workers to accept multiple connections at once
}

http {
    upstream api_gateways {
        # Define the 2 API Gateway instances (your real IPs)
        server ipv4:port weight=1; #IPv4 addres
        server [ipv6]:port weight=1; #IPv6 addres

        server ipv4:port weight=1; #IPv4 addres
        server [ipv6]:port weight=1; #IPv6 addres
    }

    # weight means distribute loads
    server {
       listen 80;                      # Listen for IPv4 on port 80
       listen [::]:80;                  # Listen for IPv6 on port 80

        location / {
            proxy_pass http://api_gateways;  # Direct traffic to the API Gateway upstream
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Create Dockerfile:
```
FROM nginx:latest

# Copy the custom NGINX configuration into the container
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80 for HTTP traffic (or 443 for HTTPS if needed)
EXPOSE 80
```

### Run command to build docker for load-balancer:
```
docker build -t nginx-lb-image .
```

### Run command to run docker images:
```
docker run -d --name nginx-lb -p 80:80 nginx-lb-image
```

### If you are making your own DNS like. `api_gateways` so you have to add this /etc/hosts
```
# vi /etc/hosts

# add below command to hosts file

xx.x.x.1 api_gateways
xxxx:xxx:xxx:xx::xxxx api_gateways
```

### Check ipv4 and ipv6 is working or not
```
ping -4 api_gateways
ping -6 api_gateways
```

### If you want to use from virtual server so you have to add from VM to resolve DNS or you have to add your Godaddy, :
```
# vi /etc/hosts

# add below command to hosts file

xx.x.x.1 api_gateways
xxxx:xxx:xxx:xx::xxxx api_gateways
```

#### Next:
- [Load Balancer with SSL](load-balancer-with-ssl.md)
