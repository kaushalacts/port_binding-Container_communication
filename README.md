 # Docker : Port Binding & Container Communication 🐳

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://nginx.org/)
[![PHPMyAdmin](https://img.shields.io/badge/PHPMyAdmin-6C78AF?style=for-the-badge&logo=phpmyadmin&logoColor=white)](https://www.phpmyadmin.net/)

> **Part of #100DaysOfDocker Challenge** - Master Docker fundamentals through hands-on scenarios

## 📖 Table of Contents

- [Overview](#overview)
- [Learning Objectives](#learning-objectives)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Scenarios](#scenarios)
  - [Scenario 1: Nginx Port Binding](#scenario-1-nginx-port-binding)
  - [Scenario 2: Multi-Container Communication](#scenario-2-multi-container-communication)
- [Docker vs Docker Compose](#docker-vs-docker-compose)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This repository demonstrates two fundamental Docker concepts through practical hands-on scenarios:

1. **Port Binding**: Running containers accessible on different host ports
2. **Container Communication**: Setting up multi-container applications with custom networking

We explore both **manual Docker commands** and the **professional Docker Compose approach**, showcasing why Docker Compose is the industry standard for multi-container applications.

## 🎓 Learning Objectives

After completing this challenge, you will understand:

- ✅ How Docker port binding works (`-p` flag)
- ✅ Container-to-container communication
- ✅ Custom Docker networks
- ✅ Docker Compose fundamentals
- ✅ Service dependencies and startup ordering
- ✅ Volume management for data persistence
- ✅ Environment variable configuration
- ✅ Production-ready multi-container setups

## 📋 Prerequisites

- Docker installed ([Installation Guide](https://docs.docker.com/get-docker/))
- Docker Compose installed ([Installation Guide](https://docs.docker.com/compose/install/))
- Basic command line knowledge
- Web browser for testing

**Verify installation:**
```bash
docker --version
docker-compose --version
```

## 🚀 Quick Start

### Clone and Run
```bash
# Clone the repository
git clone https://github.com/kaushalacts/port_binding-Container_communication.git
cd port_binding-Container_communication

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# Access services
# PHPMyAdmin: http://localhost:8081 (root/rootpassword)
```

### Cleanup
```bash
# Stop all services
docker-compose down

# Remove volumes (⚠️ This deletes database data)
docker-compose down -v
```

## 🎪 Scenarios

### Scenario 1: Nginx Port Binding

**Objective**: Run Nginx container accessible on different host ports

#### Manual Docker Approach
```bash
# Run Nginx on port 8080
docker run -d --name nginx-container -p 8080:80 nginx

# Test: http://localhost:8080

# Switch to port 9090
docker stop nginx-container
docker rm nginx-container
docker run -d --name nginx-container -p 9090:80 nginx

# Test: http://localhost:9090
```

#### Docker Compose Approach
```yaml
version: '3.8'
services:
  nginx:
    image: nginx:latest
    container_name: nginx-container
    ports:
      - "8080:80"  # Change to 9090:80 for second part
    restart: unless-stopped 
```

```bash
# Deploy
docker-compose up -d nginx

# Change port in docker-compose.yml, then:
docker-compose up -d nginx
```

### Scenario 2: Multi-Container Communication

**Objective**: Set up MySQL database with PHPMyAdmin web interface

#### Architecture
```
┌─────────────────┐    ┌──────────────────┐
│   PHPMyAdmin    │    │      MySQL       │
│   Port: 8081    │◄──►│   Port: 3306     │
│                 │    │   (internal)     │
└─────────────────┘    └──────────────────┘
         │                       │
         └───────────────────────┘
              Custom Network: mynet
```

#### Manual Docker Commands
```bash
# Create custom network
docker network create mynet

# Run MySQL
docker run -d \
  --name mysql-container \
  --network mynet \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  mysql:8.0

# Run PHPMyAdmin
docker run -d \
  --name phpmyadmin-container \
  --network mynet \
  -p 8081:80 \
  -e PMA_HOST=mysql-container \
  -e PMA_USER=root \
  -e PMA_PASSWORD=rootpassword \
  phpmyadmin/phpmyadmin
```

#### Docker Compose Approach ⭐
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-container
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - mynet
    restart: unless-stopped

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: phpmyadmin-container
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      PMA_USER: root
      PMA_PASSWORD: rootpassword
    depends_on:
      - mysql
    networks:
      - mynet
    restart: unless-stopped

networks:
  mynet:
    driver: bridge

volumes:
  mysql_data:
```

## ⚖️ Docker vs Docker Compose

| Aspect | Manual Docker | Docker Compose |
|--------|---------------|----------------|
| **Commands** | Multiple complex commands | Single `docker-compose up` |
| **Configuration** | Command-line flags | Declarative YAML |
| **Networking** | Manual network creation | Automatic |
| **Dependencies** | Manual startup ordering | Built-in with `depends_on` |
| **Environment Variables** | Repeated `-e` flags | Organized structure |
| **Volume Management** | Manual creation | Automatic |
| **Team Sharing** | Documentation/scripts | Version-controlled YAML |
| **Production Ready** | Complex setup | Built-in best practices |

## ⚙️ Advanced Configuration

### Environment Variables with .env File

Create `.env` file:
```env
MYSQL_ROOT_PASSWORD=supersecret
MYSQL_DATABASE=myapp
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass
NGINX_PORT=8080
PHPMYADMIN_PORT=8081
```

Update `docker-compose.yml`:
```yaml
services:
  nginx:
    ports:
      - "${NGINX_PORT}:80"
  
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
```

### Multi-Environment Support

**Development** (`docker-compose.override.yml`):
```yaml
version: '3.8'
services:
  mysql:
    ports:
      - "3306:3306"  # Expose MySQL for debugging
    volumes:
      - ./init-scripts:/docker-entrypoint-initdb.d
  
  nginx:
    volumes:
      - ./html:/usr/share/nginx/html  # Live reload
```

**Production** (`docker-compose.prod.yml`):
```yaml
version: '3.8'
services:
  mysql:
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '0.5'
    
  nginx:
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 128M
```

Run production:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Health Checks
```yaml
services:
  mysql:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10
      interval: 30s
      start_period: 30s
```

## 🔧 Troubleshooting

### Common Issues

#### Port Already in Use
```bash
# Check what's using the port
sudo lsof -i :8080
# or
sudo netstat -tulpn | grep :8080

# Kill the process
sudo kill -9 <PID>
```

#### Container Communication Issues
```bash
# Test network connectivity
docker-compose exec phpmyadmin ping mysql

# Check network configuration
docker network inspect docker-challenge-day13_mynet

# View container logs
docker-compose logs mysql
docker-compose logs phpmyadmin
```

#### Database Connection Problems
```bash
# Connect to MySQL directly
docker-compose exec mysql mysql -u root -p

# Check MySQL status
docker-compose exec mysql mysqladmin status -u root -p

# Reset MySQL password
docker-compose exec mysql mysql -u root -p -e "ALTER USER 'root'@'%' IDENTIFIED BY 'newpassword';"
```

#### Permission Issues
```bash
# Check volume permissions
ls -la /var/lib/docker/volumes/

# Fix MySQL data directory permissions
sudo chown -R 999:999 ./mysql-data
```

### Debug Commands
```bash
# View detailed container information
docker inspect <container_name>

# Execute commands inside containers
docker-compose exec mysql bash
docker-compose exec phpmyadmin bash

# Monitor resource usage
docker stats

# View Docker Compose configuration
docker-compose config
```

## 🏆 Best Practices

### 🔒 Security
- ✅ Use specific image tags instead of `latest`
- ✅ Store secrets in `.env` files (don't commit to Git)
- ✅ Use strong passwords
- ✅ Limit resource usage with `deploy.resources`
- ✅ Don't expose unnecessary ports
- ✅ Use non-root users in containers

### ⚡ Performance
- ✅ Use multi-stage builds for custom images
- ✅ Optimize MySQL configuration for your use case
- ✅ Use volumes for persistent data
- ✅ Implement health checks
- ✅ Configure restart policies

### 🛠️ Development
- ✅ Use `.dockerignore` files
- ✅ Implement proper logging
- ✅ Use meaningful container and service names
- ✅ Document environment variables
- ✅ Version your Docker Compose files

### 📁 File Structure
```
docker-challenge-day13/
├── docker-compose.yml          # Main configuration
├── docker-compose.override.yml # Development overrides
├── docker-compose.prod.yml     # Production configuration
├── .env.example               # Environment template
├── .env                       # Local environment (gitignored)
├── .dockerignore              # Docker ignore rules
├── init-scripts/              # Database initialization
│   └── init.sql
├── nginx/                     # Nginx configuration
│   └── nginx.conf
└── README.md                  # This file
```

## 📊 Architecture Diagram



*Complete system architecture showing port binding, container communication, and Docker Compose orchestration*

## 🧪 Testing

### Automated Tests
```bash
# Test script
#!/bin/bash
set -e

echo "🧪 Testing Docker Challenge Day 13..."

# Start services
docker-compose up -d

# Wait for services to be ready
sleep 30

# Test Nginx

# Test PHPMyAdmin
echo "Testing PHPMyAdmin..."
curl -f http://localhost:8081 || exit 1

# Test database connection
echo "Testing database connection..."
docker-compose exec -T mysql mysql -u root -prootpassword -e "SHOW DATABASES;" || exit 1

echo "✅ All tests passed!"

# Cleanup
docker-compose down
```

### Manual Verification
1. **PHPMyAdmin Test**: Navigate to `http://localhost:8081`
2. **Database Login**: Use `root` / `rootpassword`
3. **Check testdb**: Verify `testdb` database exists

## 📈 Monitoring

### Service Health
```bash
# Check all services
docker-compose ps

# View logs
docker-compose logs -f

# Monitor resource usage
docker stats $(docker-compose ps -q)
```

### Log Management
```yaml
services:
  mysql:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

 
## 📚 Resources

### Official Documentation
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- [Nginx Docker Hub](https://hub.docker.com/_/nginx)
- [PHPMyAdmin Docker Hub](https://hub.docker.com/r/phpmyadmin/phpmyadmin)

### Learning Resources
- [Docker Tutorial](https://docker-curriculum.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### How to Contribute
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Setup
```bash
# Clone your fork
git clone https://github.com/kaushalacts/port_binding-Container_communication.git
cd port_binding-Container_communication

# Create feature branch
git checkout -b feature/your-feature

# Make changes and test
docker-compose up -d
# ... test your changes ...
docker-compose down

# Commit and push
git add .
git commit -m "Your changes"
git push origin feature/your-feature
```

 

 
## 🔗 Connect

 
- 💼 LinkedIn: [LinkedIn](https://linkedin.com/in/kaushalacts)
 
---

**⭐ If this repository helped you learn Docker, please give it a star!**

*Part of the #100DaysOfDocker challenge - Building Docker expertise one day at a time!*
