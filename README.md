# MERN Stack Project Deployment Guide



## Table of Contents
- [Local Development](#local-development)
- [Docker Setup](#docker-setup)
- [Deployment Process](#deployment-process)
- [Server Configuration](#server-configuration)
- [SSL Configuration](#ssl-configuration)
- [Administration & Maintenance](#administration--maintenance)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Acknowledgments](#acknowledgments)

## Local Development

Clone the repository and install dependencies:

```bash
git clone https://github.com/username/project_name.git
cd project_name
npm install
```

To run the application locally:

```bash
npm run dev
```

## Docker Setup

### Dockerfile for Frontend

The project uses a multi-stage Docker build to minimize the final image size:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:23 AS build
# Set the working directory
WORKDIR /app
# Copy the package.json and package-lock.json
COPY package*.json ./
# Install dependencies
RUN npm install
# Copy the rest of the application code
COPY . .
# Build the app
RUN npm run build
# Check if build directory exists and list contents
RUN ls -la

# Use a minimal Node.js runtime to serve the app
FROM node:23-alpine
# Set the working directory
WORKDIR /app
# Copy the build output from the previous stage
COPY --from=build /app/dist ./build
# Install a simple web server to serve the static files
RUN npm install -g serve
# Expose port 5002 to the outside world
EXPOSE 5002
# Command to run the app
CMD ["serve", "-s", "build", "-l", "5002"]
```

### Build and Run Locally

```bash
# Build the Docker image
docker build -t username/project_name:V0.0.1 .

# Run the container
docker run -d -p 5002:5002 username/project_name:V0.0.1
```

### Dockerfile for Backend

The project uses a multi-stage Docker build to minimize the final image size:

```dockerfile

# Use an official Node.js runtime as a parent image
FROM node:23 AS build

# Set the working directory
WORKDIR /app

# Copy the package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the app
RUN npm run build

# Use a minimal Node.js runtime to serve the app
FROM node:23-alpine

# Set the working directory
WORKDIR /app

# Copy the build output from the previous stage
COPY --from=build /app/build ./build

# Install a simple web server to serve the static files
RUN npm install -g serve

# Expose port 5000 to the outside world
EXPOSE 5005

# Command to run the app
CMD ["serve", "-s", "build", "-l", "5005"]



# TEMP

# Use an official Node.js runtime as a parent image
FROM node:20 AS build

# Set the working directory
WORKDIR /app

# Copy the package.json and package-lock.json
COPY package*.json ./

#Copy the rest of the application code
COPY . .

#Build the app (if you have a build process)
RUN npm run build

# Use a minimal Node.js runtime to serve the app
FROM node:20-alpine

#Set the working directory
WORKDIR /app

# Copy node_modules from your local machine to the Docker image
COPY node_modules /app/node_modules

# # Copy the build output from the previous stage
# COPY --from=build /app/build /app/build

# Install a simple web server to serve the static files
RUN npm install -g serve

# Expose port 5005 to the outside world
EXPOSE 5005

# Command to run the app
CMD ["serve", "-s", "build", "-l", "5005"]


## TEST

# Use an official Node.js runtime as a parent image
FROM node:20-alpine

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json (if needed for installation or build process)
COPY package*.json ./

# Copy node_modules from the local machine to the Docker image
COPY node_modules /app/node_modules

# Copy the rest of the application code and build output
COPY . .

# Install a simple web server to serve static files
RUN npm install -g serve

# Expose port 5005 to the outside world
EXPOSE 5005

# Command to run the app
CMD ["serve", "-s", "build", "-l", "5005"]

```

### Push to Docker Hub

```bash
# Login to Docker Hub (if not already logged in)
docker login

# Push the image
docker push username/project_name:V0.0.1
```

## Deployment Process

### Server Prerequisites

Ensure the following are installed on your server:
- Docker
- Nginx
- Certbot
- FirewallD (or another firewall manager)

### SSH to Your Server

```bash
ssh root@your_server_ip
```

### Pull Docker Image

```bash
docker pull username/project_name:V0.0.1
```

### Run the Container

```bash
docker run -d -p 5002:5002 username/project_name:V0.0.1
```

## Server Configuration

### Configure Firewall

Allow the required ports through the firewall:

```bash
sudo firewall-cmd --zone=public --add-port=5002/tcp --permanent
sudo firewall-cmd --reload
```

Verify the configuration:

```bash
sudo firewall-cmd --list-all
```

### Configure Nginx

Create a new site configuration file:

#### For distributions using sites-available/sites-enabled:

```bash
sudo nano /etc/nginx/sites-available/project
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name project.com;
    location / {
        proxy_pass http://127.0.0.1:5002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled/
```

#### For distributions using conf.d:

```bash
sudo nano /etc/nginx/conf.d/project.com.conf
```

Add the same configuration as above.

### Test and Reload Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## SSL Configuration

### Configure HTTPS with Certbot

```bash
sudo certbot --nginx -d project.com
```

Certbot will:
1. Obtain the SSL certificate
2. Configure Nginx to use HTTPS
3. Set up automatic renewal for the certificate

## Administration & Maintenance

### Docker Management

Check running containers:
```bash
sudo docker ps
```

Stop a container:
```bash
sudo docker stop <container_id>
```

Remove a container:
```bash
sudo docker rm <container_id>
```

### Logs and Debugging

Nginx logs:
```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

Application logs:
```bash
sudo docker logs <container_id>
```

### Renewing Certificates Manually

Certbot automatically renews certificates. To manually renew:
```bash
sudo certbot renew
```

Reload Nginx after renewal:
```bash
sudo systemctl reload nginx
```

### Update the Application

1. Build and push a new Docker image
2. SSH into your server
3. Pull the new image and restart the container:

```bash
docker pull username/project_name:latest
docker stop <container_id>
docker run -d -p 5002:5002 username/project_name:latest
```

## Troubleshooting

### Container Issues

Check if the container is running:
```bash
docker ps
```

View container logs:
```bash
docker logs <container_id>
```

### Nginx Issues

Check Nginx error logs:
```bash
sudo tail -f /var/log/nginx/error.log
```

### Firewall Issues

Verify firewall settings:
```bash
sudo firewall-cmd --list-all
```

### SSL Issues

Check Certbot certificates:
```bash
sudo certbot certificates
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Acknowledgments

* **Docker Hub**: For container registry services - [Docker Hub](https://hub.docker.com/r/username/project_name)
* **Let's Encrypt & Certbot**: For free SSL certificates - [Let's Encrypt](https://letsencrypt.org/)
* **Nginx**: For web server and reverse proxy - [Nginx Documentation](https://nginx.org/en/docs/)
* All the amazing contributors and supporters of this open source project

---

⭐ If you found this project helpful, please consider giving it a star!
