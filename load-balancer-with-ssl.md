# Load balancer using Nginx:
A load balancer is a device or software application that distributes incoming network traffic across multiple servers. This ensures that no single server is overwhelmed, improving the reliability, scalability, and availability of applications or services.

In the context of Docker and Nginx, you can set up a load balancer that distributes HTTP(S) or WebSocket traffic across multiple backend services or servers.

### Steps to Install Nginx Using Docker without SSL:

### Create nginx.conf file:
```nginx
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

    server {
        listen 80;                        # Listen for IPv4 on port 80
        listen [::]:80;                    # Listen for IPv6 on port 80
        server_name example.com;           # Your domain name

        # Redirect HTTP traffic to HTTPS
        return 301 https://$host$request_uri;
    }

    # weight means distribute loads
    listen 443 ssl;                    # Listen for HTTPS (SSL) traffic
        listen [::]:443 ssl;               # Listen for HTTPS on IPv6

        server_name example.com;           # Your domain name

        # SSL configuration
        ssl_certificate /etc/nginx/ssl/cert.pem;         # Path to your SSL certificate
        ssl_certificate_key /etc/nginx/ssl/key.pem;      # Path to your SSL private key
        ssl_protocols TLSv1.2 TLSv1.3;                    # Secure protocols
        ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';  # Strong ciphers

        # Optional: Redirect to a more secure cipher suite
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass http://api_gateways;             # Direct traffic to the API Gateway upstream
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```

### Create Dockerfile:
```dockerfile
# Use official Nginx image as the base image
FROM nginx:latest

# Set the maintainer label
# LABEL maintainer="yourname@example.com"

# Copy the Nginx configuration file to the container
COPY nginx.conf /etc/nginx/nginx.conf

# Copy SSL certificate and key to the container (replace with your actual paths)
COPY ./ssl/cert.pem /etc/nginx/ssl/cert.pem
COPY ./ssl/key.pem /etc/nginx/ssl/key.pem

# Expose ports for HTTP and HTTPS
EXPOSE 80
EXPOSE 443

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Run command to build docker for load-balancer:
```bash
docker build -t nginx-lb-image .
```

### Run command to run docker images:
```bash
docker run -d --name nginx-lb -p 80:80 nginx-lb-image
```

### If you are making your own DNS like. `api_gateways` so you have to add this /etc/hosts
```bash
# vi /etc/hosts

# add below command to hosts file

xx.x.x.1 api_gateways
xxxx:xxx:xxx:xx::xxxx api_gateways
```

### Check ipv4 and ipv6 is working or not
```bash
ping -4 api_gateways
ping -6 api_gateways
```

### If you want to use from virtual server so you have to add from VM to resolve DNS or you have to add your Godaddy, :
```bash
# vi /etc/hosts

# add below command to hosts file

xx.x.x.1 api_gateways
xxxx:xxx:xxx:xx::xxxx api_gateways
```

#### Next:
- [Load Balancer without SSL](README.md)
