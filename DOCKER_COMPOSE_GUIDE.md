# Docker Compose Setup Guide

This guide explains how to set up and run the MyResumo application stack using Docker Compose with integrated GitHub Copilot API service.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [GitHub Secrets Configuration](#github-secrets-configuration)
- [Environment Configuration](#environment-configuration)
- [Service Details](#service-details)
- [Local Development](#local-development)
- [Production Deployment](#production-deployment)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

## 🔍 Overview

This Docker Compose configuration orchestrates a complete AI-powered resume customization platform consisting of:

- **copilot-api**: GitHub Copilot API integration service (Port 4141)
- **myresumo**: AI-powered resume customization platform (Port 8080)
- **mongodb**: Document database for data persistence (Port 27017)
- **redis**: Cache service for improved performance (Port 6379)

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐
│   Users/Web     │    │   Developers    │
│                 │    │                 │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          │ Port 8080            │ Port 4141
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│    MyResumo     │───▶│   copilot-api   │
│   (Frontend)    │    │  (AI Service)   │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│    MongoDB      │    │  External APIs  │
│   (Database)    │    │ (GitHub/OpenAI) │
└─────────┬───────┘    └─────────────────┘
          │
          ▼
┌─────────────────┐
│     Redis       │
│    (Cache)      │
└─────────────────┘
```

## 📋 Prerequisites

### Required Software
- Docker Engine 20.10+
- Docker Compose 2.0+
- Git (for cloning the repository)

### Required Accounts & Tokens
- GitHub account with Personal Access Token
- OpenAI API key (optional, for fallback)
- MongoDB Atlas account (for production) - optional

### System Requirements
- RAM: 4GB minimum, 8GB recommended
- Storage: 10GB free space
- Network: Internet connection for AI services

## 🚀 Quick Start

### 1. Clone and Setup

```bash
# Clone the repository
git clone https://github.com/your-username/MyResumo.git
cd MyResumo

# Copy environment template
cp .env.example .env
```

### 2. Configure Environment Variables

Edit the `.env` file with your actual values:

```bash
# Required: GitHub token for copilot-api
GH_TOKEN=your_github_token_here

# Optional: Keep these defaults for local development
API_KEY=sk-placeholder
API_BASE=http://copilot-api:4141
MODEL_NAME=gpt-3.5-turbo
MONGODB_URI=mongodb://admin:devpassword@mongodb:27017/myresumo_dev
```

### 3. Start Services

```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f
```

### 4. Access Applications

- **MyResumo UI**: http://localhost:8080
- **Copilot API**: http://localhost:4141
- **MongoDB**: localhost:27017
- **Redis**: localhost:6379

## 🔐 GitHub Secrets Configuration

### For Public Forks

Since this is a public repository, sensitive tokens must be stored securely using GitHub repository secrets.

#### Setting Up Repository Secrets

1. **Navigate to Repository Settings**
   ```
   Your Repository → Settings → Secrets and variables → Actions
   ```

2. **Add Required Secrets**
   
   Click "New repository secret" and add:
   
   | Secret Name | Description | Example |
   |-------------|-------------|---------|
   | `GH_TOKEN` | GitHub Personal Access Token | `ghp_xxxxxxxxxxxx` |
   | `OPENAI_API_KEY` | OpenAI API key (optional) | `sk-xxxxxxxxxxxxx` |
   | `MONGO_ROOT_PASSWORD` | Production MongoDB password | `secure_password` |
   | `REDIS_PASSWORD` | Production Redis password | `secure_redis_pwd` |

3. **GitHub Token Permissions**
   
   Your GitHub token needs these scopes:
   - `repo` (if private repository)
   - `read:user`
   - `codespace` (for Copilot access)

#### Creating a GitHub Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Click "Generate new token (classic)"
3. Select required scopes: `repo`, `read:user`, `codespace`
4. Copy the token and add it to repository secrets as `GH_TOKEN`

### GitHub Actions Integration

The included workflow (`.github/workflows/docker-compose-deploy.yml`) automatically:

- Validates Docker Compose configuration
- Creates environment file from secrets
- Builds and tests all services
- Deploys to production (if configured)

#### Workflow Triggers

```yaml
on:
  push:
    branches: [main, develop]  # Auto-deploy on push
  pull_request:
    branches: [main]           # Test on PRs
  workflow_dispatch:           # Manual trigger
```

## ⚙️ Environment Configuration

### Development Configuration (`.env`)

```bash
# GitHub Integration
GH_TOKEN=your_github_token_here

# MyResumo Development Settings
API_KEY=sk-placeholder
API_BASE=http://copilot-api:4141
MODEL_NAME=gpt-3.5-turbo
DEBUG=true
LOG_LEVEL=debug

# Local Database
MONGODB_URI=mongodb://admin:devpassword@mongodb:27017/myresumo_dev
MONGO_ROOT_USERNAME=admin
MONGO_ROOT_PASSWORD=devpassword

# Local Cache
REDIS_PASSWORD=devredispassword
```

### Production Configuration

```bash
# Production GitHub Integration
GH_TOKEN=${GITHUB_SECRET}

# Production MyResumo Settings
API_KEY=${OPENAI_API_KEY}
API_BASE=http://copilot-api:4141
MODEL_NAME=gpt-4
DEBUG=false
LOG_LEVEL=warning

# Production Database
MONGODB_URI=mongodb://username:password@production-host:27017/myresumo
MONGO_ROOT_PASSWORD=${SECURE_PASSWORD}

# Production Cache
REDIS_PASSWORD=${SECURE_REDIS_PASSWORD}
```

## 🔧 Service Details

### copilot-api Service

**Purpose**: Provides OpenAI-compatible API using GitHub Copilot  
**Port**: 4141  
**Health Check**: Available at `/health` (if implemented)

```yaml
copilot-api:
  build: .
  ports:
    - "4141:4141"
  environment:
    - GH_TOKEN=${GH_TOKEN}
  restart: unless-stopped
```

**Key Features**:
- Acts as OpenAI API proxy
- Uses GitHub Copilot for AI functionality
- Handles authentication automatically
- Provides cost optimization

### myresumo Service

**Purpose**: AI-powered resume customization platform  
**Port**: 8080  
**Base Image**: `ghcr.io/analyticace/myresumo:latest`

```yaml
myresumo:
  image: ghcr.io/analyticace/myresumo:latest
  ports:
    - "8080:8080"
  environment:
    - API_BASE=http://copilot-api:4141
  depends_on:
    - copilot-api
    - mongodb
```

**Key Features**:
- Resume optimization and customization
- ATS compatibility scoring
- Multiple export formats
- Version management

### MongoDB Service

**Purpose**: Primary data persistence  
**Port**: 27017  
**Image**: `mongo:7-jammy`

```yaml
mongodb:
  image: mongo:7-jammy
  volumes:
    - mongodb_data:/data/db
  environment:
    - MONGO_INITDB_ROOT_USERNAME=admin
    - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
```

**Stored Data**:
- User resumes and profiles
- Job descriptions and analysis
- ATS scoring results
- Application settings

### Redis Service

**Purpose**: Caching and session management  
**Port**: 6379  
**Image**: `redis:7-alpine`

```yaml
redis:
  image: redis:7-alpine
  command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
  volumes:
    - redis_data:/data
```

**Use Cases**:
- API response caching
- Rate limiting data
- Session storage
- Temporary computation results

## 🛠️ Local Development

### Development Setup

```bash
# Start with development overrides
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d

# Or simply (override is automatic)
docker-compose up -d
```

### Development Features

The `docker-compose.override.yml` provides:

- **Hot Reloading**: Code changes are reflected immediately
- **Debug Mode**: Enhanced logging and error reporting
- **Volume Mounts**: Source code mounted for development
- **Faster Health Checks**: Reduced intervals for development

### Development Commands

```bash
# View logs for specific service
docker-compose logs -f myresumo

# Execute commands in running container
docker-compose exec myresumo bash

# Restart specific service
docker-compose restart copilot-api

# View service configuration
docker-compose config
```

### Database Management

```bash
# Access MongoDB shell
docker-compose exec mongodb mongosh -u admin -p devpassword

# Access Redis CLI
docker-compose exec redis redis-cli -a devredispassword

# Backup MongoDB data
docker-compose exec mongodb mongodump --out /data/backup

# View database logs
docker-compose logs mongodb
```

## 🚀 Production Deployment

### Production Checklist

- [ ] GitHub secrets configured
- [ ] Environment variables validated
- [ ] SSL certificates ready (if using HTTPS)
- [ ] Database backups configured
- [ ] Monitoring and logging set up
- [ ] Resource limits defined
- [ ] Security hardening applied

### Production Deployment Steps

1. **Prepare Production Environment**
   ```bash
   # Create production environment file
   cp .env.example .env.production
   
   # Edit with production values
   nano .env.production
   ```

2. **Deploy with Production Configuration**
   ```bash
   # Deploy using production environment
   docker-compose --env-file .env.production up -d
   
   # Or use production compose file
   docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
   ```

3. **Verify Deployment**
   ```bash
   # Check all services are running
   docker-compose ps
   
   # Test service connectivity
   curl -f http://localhost:8080/health
   curl -f http://localhost:4141/health
   
   # Check logs for errors
   docker-compose logs --tail=100
   ```

### Production Security

- **Environment Variables**: Use secrets management
- **Network Security**: Implement firewall rules
- **SSL/TLS**: Configure HTTPS with valid certificates
- **Regular Updates**: Keep images and dependencies updated
- **Monitoring**: Set up health checks and alerting

## 🔧 Troubleshooting

### Common Issues

#### 1. GitHub Token Issues

**Symptom**: copilot-api fails to start or returns authentication errors

**Solutions**:
```bash
# Check if token is set
echo $GH_TOKEN

# Verify token permissions at github.com/settings/tokens
# Ensure scopes include: repo, read:user, codespace

# Test token manually
curl -H "Authorization: token $GH_TOKEN" https://api.github.com/user
```

#### 2. Port Conflicts

**Symptom**: "Port already in use" errors

**Solutions**:
```bash
# Check what's using the ports
netstat -tulpn | grep :8080
netstat -tulpn | grep :4141

# Stop conflicting services or change ports in docker-compose.yml
# For example:
ports:
  - "8081:8080"  # Change host port
```

#### 3. Service Won't Start

**Symptom**: Service exits immediately or fails health checks

**Solutions**:
```bash
# Check service logs
docker-compose logs servicename

# Check service configuration
docker-compose config

# Validate environment variables
docker-compose exec servicename env | grep API

# Restart with fresh volumes
docker-compose down -v
docker-compose up -d
```

#### 4. Database Connection Issues

**Symptom**: MyResumo can't connect to MongoDB

**Solutions**:
```bash
# Check if MongoDB is running
docker-compose ps mongodb

# Test MongoDB connection
docker-compose exec mongodb mongosh --eval "db.adminCommand('ping')"

# Check network connectivity
docker-compose exec myresumo ping mongodb

# Verify MongoDB URI format
echo $MONGODB_URI
```

#### 5. API Communication Issues

**Symptom**: MyResumo can't reach copilot-api

**Solutions**:
```bash
# Check if copilot-api is accessible
docker-compose exec myresumo curl http://copilot-api:4141

# Verify internal network
docker network ls
docker network inspect myresumo_app-network

# Check API_BASE configuration
docker-compose exec myresumo env | grep API_BASE
```

### Debugging Commands

```bash
# Get detailed service information
docker-compose ps --services
docker-compose ps --all

# Check resource usage
docker stats

# Inspect specific container
docker inspect myresumo_myresumo_1

# Check network configuration
docker network ls
docker network inspect myresumo_app-network

# View real-time logs
docker-compose logs -f --tail=100

# Test service health
docker-compose exec myresumo curl -f http://localhost:8080/health
```

### Log Analysis

```bash
# View all logs
docker-compose logs > all-logs.txt

# Filter logs by service
docker-compose logs myresumo > myresumo-logs.txt

# Search for errors
docker-compose logs | grep -i error

# Follow logs in real-time
docker-compose logs -f --tail=50 myresumo
```

## ❓ FAQ

### Q: Can I use this without GitHub Copilot?

**A**: Yes! You can configure MyResumo to use OpenAI directly:

```bash
# In your .env file
API_KEY=your_openai_api_key
API_BASE=https://api.openai.com/v1
MODEL_NAME=gpt-3.5-turbo
```

### Q: How do I backup my data?

**A**: Use these commands to backup your data:

```bash
# Backup MongoDB
docker-compose exec mongodb mongodump --out /tmp/backup
docker cp myresumo_mongodb_1:/tmp/backup ./mongodb-backup

# Backup Redis
docker-compose exec redis redis-cli --rdb /tmp/dump.rdb
docker cp myresumo_redis_1:/tmp/dump.rdb ./redis-backup.rdb
```

### Q: How do I scale the services?

**A**: You can scale services using:

```bash
# Scale MyResumo to 3 instances
docker-compose up -d --scale myresumo=3

# Use a load balancer (nginx, traefik) for production scaling
```

### Q: Can I run this on different ports?

**A**: Yes, modify the ports in `docker-compose.yml`:

```yaml
services:
  myresumo:
    ports:
      - "8081:8080"  # External:Internal
  copilot-api:
    ports:
      - "4142:4141"
```

### Q: How do I update to the latest version?

**A**: Update the services with:

```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d

# Clean up old images
docker image prune
```

### Q: How do I monitor resource usage?

**A**: Use these monitoring commands:

```bash
# Real-time stats
docker stats

# Service resource usage
docker-compose top

# System resource usage
docker system df
```

## 📚 Additional Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [MyResumo Project Repository](https://github.com/AnalyticAce/MyResumo)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [MongoDB Docker Documentation](https://hub.docker.com/_/mongo)
- [Redis Docker Documentation](https://hub.docker.com/_/redis)

## 🆘 Getting Help

If you encounter issues:

1. Check this troubleshooting guide
2. Review the [GitHub Issues](https://github.com/AnalyticAce/MyResumo/issues)
3. Create a new issue with:
   - Docker Compose version
   - Error logs
   - Environment configuration (without secrets)
   - Steps to reproduce

---

**Happy Resume Building! 🎯**