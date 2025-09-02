# 🐳 Docker Image Deployment to Huawei SWR
## Technical Guide for Container Registry Management

## 📋 Prerequisites

- Docker installed and configured locally
- Huawei Cloud account with SWR (Software Repository) access
- Access credentials (Access Key ID and Secret Access Key)
- Basic familiarity with Docker commands

## 🎯 Deployment Process Overview

### Step 1: Configure SWR Namespace

Navigate to Huawei Cloud Console → Container Registry (SWR) → Organization Management to set up your namespace.

```bash
# SWR repository URL format:
# swr.[REGION].myhuaweicloud.com/[NAMESPACE]/[IMAGE_NAME]

# Example:
# swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server
```

📝 **Note**: Document your namespace and region for consistent usage across deployments.

### Step 2: Build Docker Image

```bash
# Build your Docker image
docker build -t pandas-mcp-server:latest .

# Verify successful build
docker images | grep pandas-mcp-server
```

### Step 3: 🔐 SWR Authentication

Huawei SWR requires specific authentication format for Docker login.

#### Method A: Console Authentication
1. Navigate to SWR Console → Access Credentials
2. Click "Obtain Login Command"
3. Copy the generated command
4. Execute in terminal

#### Method B: CLI Authentication
```bash
# Authentication format
docker login -u [REGION]@[ACCESS_KEY] -p [SECRET_KEY] swr.[REGION].myhuaweicloud.com

# Example:
docker login -u ap-southeast-3@ABCD1234 -p [YOUR_SECRET_TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

⚠️ **Important**: Login tokens expire every 12 hours and must be refreshed regularly.

#### 🔧 Troubleshooting Authentication Issues

**Common Authentication Error:**
```bash
# Error: "unauthorized: authentication required"
# Solution: Token has expired, refresh credentials

# Clear existing authentication
docker logout swr.ap-southeast-3.myhuaweicloud.com

# Re-authenticate with fresh credentials
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

#### 🛠️ Advanced Credential Management

When standard authentication methods fail, comprehensive credential reset may be required:

```bash
# Check current credential helper configuration
cat ~/.docker/config.json | grep credsStore

# Clear credential helper cache based on your system:

# macOS:
docker-credential-osxkeychain erase < /dev/null

# Linux (with pass):
docker-credential-pass erase < /dev/null

# Windows (PowerShell as Administrator):
docker-credential-wincred erase

# Backup and reset Docker configuration
mv ~/.docker/config.json ~/.docker/config.json.backup

# Restart Docker service
# macOS:
killall Docker && open -a Docker

# Linux:
sudo systemctl restart docker

# Windows:
# Restart Docker Desktop from system tray

# Re-authenticate
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

**Manual Credential Configuration:**
```bash
# Edit ~/.docker/config.json directly
{
  "auths": {
    "swr.ap-southeast-3.myhuaweicloud.com": {
      "auth": "BASE64_ENCODED_CREDENTIALS"
    }
  }
}

# Generate base64 encoded credentials
echo -n "ap-southeast-3@YOUR_KEY:YOUR_TOKEN" | base64
```

### Step 4: 🏷️ Tag Images for SWR

Properly tag your local images with the SWR repository path:

```bash
# Tag image with SWR repository path
docker tag [LOCAL_IMAGE]:[TAG] [SWR_FULL_PATH]:[TAG]

# Example:
docker tag pandas-mcp-server:latest swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Additional version tagging (recommended)
docker tag pandas-mcp-server:latest swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:v1.0.0

# Verify tags
docker images | grep swr
```

### Step 5: 🚀 Push to SWR

Execute the push operation to upload your image:

```bash
# Push image to SWR
docker push swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Monitor upload progress
# Note: Large images may require multiple retry attempts
```

**Common Push Issues and Solutions:**

| Error | Solution |
|-------|----------|
| `denied: requested access to the resource is denied` | Re-authenticate with valid credentials |
| `manifest size exceeds the limit` | Reduce image size through optimization |
| `timeout` errors | Retry operation; check network connectivity |

### Step 6: ✅ Verification

Verify successful upload through SWR Console:
- Navigate to SWR Console → Image Repository → Your Repository
- Confirm image presence and tags

## 📝 Complete Deployment Script

Automated deployment script for streamlined operations:

```bash
#!/bin/bash
# deploy-to-swr.sh - Automated SWR deployment script

# Configuration
NAMESPACE="wooyankit"
IMAGE_NAME="pandas-mcp-server"
VERSION="v1.0.0"
REGION="ap-southeast-3"
SWR_URL="swr.${REGION}.myhuaweicloud.com"

# Build image
echo "🔨 Building Docker image..."
docker build -t ${IMAGE_NAME}:${VERSION} .

# Authenticate with SWR
echo "🔐 Authenticating with SWR..."
# Insert authentication command from Huawei Console

# Tag images
echo "🏷️ Tagging images for SWR..."
docker tag ${IMAGE_NAME}:${VERSION} ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}
docker tag ${IMAGE_NAME}:${VERSION} ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest

# Push to repository
echo "🚀 Pushing to SWR repository..."
docker push ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}
docker push ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest

echo "✅ Deployment complete!"
echo "   Images available at:"
echo "   - ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}"
echo "   - ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest"
```

## ⚡ Optimization Strategies

### 1. Multi-Stage Builds
Reduce image size through multi-stage build processes:

```dockerfile
# Build stage
FROM python:3.10 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.10-slim
WORKDIR /app
COPY --from=builder /app .
COPY . .
```

### 2. Layer Caching Optimization
Structure Dockerfile for optimal caching:

```dockerfile
# Dependencies (changes less frequently)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Application code (changes frequently)
COPY . .
```

### 3. Use .dockerignore
Exclude unnecessary files from build context:

```bash
# .dockerignore
.git
*.pyc
__pycache__
.env
logs/
data/
*.md
.DS_Store
```

## 🌍 Regional Endpoints

SWR endpoints vary by region:

| Region | Endpoint |
|--------|----------|
| Singapore | `swr.ap-southeast-3.myhuaweicloud.com` |
| Hong Kong | `swr.ap-southeast-1.myhuaweicloud.com` |
| Shanghai | `swr.cn-east-3.myhuaweicloud.com` |

📍 **Tip**: Verify your region's endpoint in the Huawei Cloud Console.

## 📊 Image Management Best Practices

### Version Control Strategy

Implement semantic versioning for better image management:

```bash
# Semantic versioning examples
docker tag myapp:latest swr.../myapp:v1.0.0  # Major release
docker tag myapp:latest swr.../myapp:v1.0.1  # Patch/Bug fix
docker tag myapp:latest swr.../myapp:v1.1.0  # Minor/Feature release

# Maintain standard tags
docker tag myapp:latest swr.../myapp:latest   # Latest stable
docker tag myapp:latest swr.../myapp:dev      # Development branch
docker tag myapp:latest swr.../myapp:staging  # Staging environment
```

### Local Image Cleanup

Maintain local Docker environment:

```bash
# Remove unused images
docker image prune -a

# Remove specific SWR images
docker rmi swr.ap-southeast-3.myhuaweicloud.com/namespace/image:tag

# Complete system cleanup (use with caution)
docker system prune -a --volumes
```

## 🚨 Troubleshooting Guide

### Common Issues and Resolutions

**Issue: "Login succeeded" but push fails**
- **Cause**: Token expiration during push operation
- **Solution**: Re-authenticate and retry push immediately

**Issue: Push succeeds but image not visible in console**
- **Cause**: Region mismatch or console delay
- **Solution**: Verify correct region; allow time for console update

**Issue: "Layer already exists" with slow push**
- **Cause**: SWR layer verification process
- **Solution**: Allow process to complete; layers are being verified

**Issue: "Connection reset by peer"**
- **Cause**: Network interruption or server-side issue
- **Solution**: Retry after brief delay; check network connectivity

## ✅ Deployment Checklist

Ensure successful deployment with this verification checklist:

- [ ] Docker image builds successfully locally
- [ ] Image size optimized (target < 500MB, max 1GB)
- [ ] SWR authentication successful
- [ ] Image properly tagged with SWR repository URL
- [ ] Push operation completed without errors
- [ ] Image visible in SWR console
- [ ] Version tags applied appropriately
- [ ] Documentation updated with new image details

## 🎯 Key Recommendations

### Best Practices Summary

1. **Always use version tags** in addition to `latest`
2. **Refresh authentication tokens** before large push operations
3. **Optimize image size** through multi-stage builds
4. **Document your namespace and region** for team reference
5. **Implement automated deployment scripts** for consistency
6. **Monitor push operations** for timeout or network issues
7. **Maintain cleanup routines** for local Docker environment

### Security Considerations

- Store credentials securely using secret management tools
- Rotate access keys regularly
- Use IAM policies to limit SWR access appropriately
- Enable image scanning in SWR for vulnerability detection
- Implement image signing for production deployments

---

## 📚 Additional Resources

- [Huawei SWR Official Documentation](https://support.huaweicloud.com/intl/en-us/swr/index.html)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Container Security Guidelines](https://docs.docker.com/engine/security/)

---

*📅 Document Version: 1.0*  
*🔄 Last Updated: 09 September, 2025*