# 🚀 MCP Server Deployment on Huawei ECS
## Comprehensive Guide for Container Deployment and Management

## 📋 Prerequisites

- Huawei Cloud ECS instance with Linux OS (Ubuntu 20.04 recommended)
- Docker Engine and Docker Compose installed
- Access to Huawei SWR (Software Repository) registry
- Root or sudo access to the server
- Network connectivity and security group configuration

## 🎯 Deployment Overview

### Step 1: Server Access

Establish SSH connection to your ECS instance:

```bash
ssh root@[your-ecs-ip]
# Alternative: Use key-based authentication for enhanced security
```

### Step 2: Directory Structure Setup

Create the required directory hierarchy for the MCP server:

```bash
# Create base directory
mkdir -p /opt/mcp-server
cd /opt/mcp-server

# Create subdirectories for different components
mkdir -p data charts logs temp config

# Set appropriate permissions
chmod -R 755 data charts logs temp config
```

### Step 3: 📝 Docker Compose Configuration

Create `/opt/mcp-server/docker-compose.yml` with the following configuration:

```yaml
# Docker Compose configuration for MCP Server
services:
  pandas-mcp-server:
    image: swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest
    container_name: pandas-mcp-server
    ports:
      - "8000:8000"
    environment:
      - MCP_SERVER_HOST=0.0.0.0
      - MCP_SERVER_PORT=8000
      - MCP_SERVER_TRANSPORT=sse
      - MCP_LOG_LEVEL=INFO  # Options: DEBUG, INFO, WARNING, ERROR
      - MCP_MAX_DATAFRAME_SIZE_MB=100
      - MCP_MAX_FILE_SIZE_MB=100
    volumes:
      - ./data:/app/data        # Data persistence
      - ./charts:/app/charts    # Chart outputs
      - ./logs:/app/logs        # Application logs
      - ./temp:/tmp            # Temporary files
      - ./config:/app/config   # Configuration files
    restart: unless-stopped
    networks:
      - mcp-network
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"

networks:
  mcp-network:
    driver: bridge
```

### Step 4: 🔐 SWR Authentication

Configure Docker to authenticate with Huawei SWR:

```bash
# Standard authentication command
docker login -u ap-southeast-3@[YOUR_ACCOUNT_ID] swr.ap-southeast-3.myhuaweicloud.com
```

#### Authentication Troubleshooting

**Method A: Console-Generated Credentials**
```bash
# Obtain login command from Huawei Cloud Console:
# Console → Container Registry → Access Keys → Get Login Command
docker login -u ap-southeast-3@[ACCESS_KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

**Method B: Token Refresh**
```bash
# Tokens expire every 12 hours
# Regenerate through Huawei Cloud Console when needed
```

**Method C: Region Verification**
```bash
# Ensure correct region endpoint:
# ap-southeast-3: Singapore
# ap-southeast-1: Hong Kong
# Verify in SWR console URL
```

**Method D: Credential Reset**
```bash
# Clear existing Docker credentials if experiencing issues
rm ~/.docker/config.json
# Re-authenticate with fresh credentials
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

### Step 5: 🐳 Image Deployment

Pull the container image from SWR:

```bash
# Pull the latest image
docker pull swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Verify image download
docker images | grep pandas-mcp-server
```

### Step 6: 🔧 Permission Configuration

Configure proper file permissions for container volumes:

```bash
# Create log file
touch /opt/mcp-server/logs/pandas_mcp.log
chmod 666 /opt/mcp-server/logs/pandas_mcp.log

# Standard permission setup (recommended)
chown -R 1000:1000 /opt/mcp-server/logs
chown -R 1000:1000 /opt/mcp-server/data
chown -R 1000:1000 /opt/mcp-server/charts
chown -R 1000:1000 /opt/mcp-server/temp
chown -R 1000:1000 /opt/mcp-server/config

# Alternative permissions (if standard setup fails)
# Note: Use more permissive settings only when necessary
chmod -R 775 /opt/mcp-server/logs
chmod -R 775 /opt/mcp-server/data
chmod -R 775 /opt/mcp-server/charts
```

### Step 7: 🚀 Service Launch

Start the MCP server using Docker Compose:

```bash
# Launch container in detached mode
docker-compose up -d

# Verify container status
docker-compose ps

# Monitor startup logs
docker-compose logs -f --tail=100
```

### Step 8: ✅ Service Verification

Confirm the service is operational:

```bash
# Test SSE endpoint
curl http://localhost:8000/sse

# Check log output
ls -la /opt/mcp-server/logs/
tail -f /opt/mcp-server/logs/pandas_mcp.log
```

## 📁 Directory Structure

Expected directory layout after successful deployment:

```
/opt/mcp-server/
├── docker-compose.yml    # Docker Compose configuration
├── data/                 # Persistent data storage
├── charts/              # Generated chart outputs
├── logs/                # Application logs
│   └── pandas_mcp.log  # Main application log
├── temp/                # Temporary file storage
└── config/              # Configuration files
```

## 🛠️ Operational Commands

### Monitoring and Diagnostics

```bash
# View real-time logs
docker-compose logs -f pandas-mcp-server

# Check container status
docker-compose ps

# Monitor resource usage
docker stats pandas-mcp-server

# Application log monitoring
tail -f /opt/mcp-server/logs/pandas_mcp.log
```

### Container Management

```bash
# Stop services
docker-compose down

# Restart services
docker-compose restart

# Update container image
docker-compose pull
docker-compose up -d

# Access container shell
docker-compose exec pandas-mcp-server /bin/bash
```

## 🚨 Troubleshooting Guide

### Permission Denied Errors

Resolution steps for file permission issues:

```bash
# Stop the container
docker-compose down

# Reset permissions
chmod -R 775 /opt/mcp-server/logs
touch /opt/mcp-server/logs/pandas_mcp.log
chmod 666 /opt/mcp-server/logs/pandas_mcp.log

# Restart container
docker-compose up -d
```

### Docker Compose Version Issues

Update Docker Compose to the latest version:

```bash
# Download latest Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -sf /usr/local/bin/docker-compose /usr/bin/docker-compose

# Verify installation
docker-compose --version
```

### Container Startup Failures

Diagnostic steps for container issues:

```bash
# Review recent logs
docker-compose logs --tail=200

# Inspect container configuration
docker inspect pandas-mcp-server

# Check disk space
df -h

# Verify memory availability
free -h
```

## 🔐 Security Configuration

### Firewall Rules

Configure firewall to allow MCP server access:

```bash
# Allow traffic on port 8000
sudo ufw allow 8000/tcp
sudo ufw reload
```

⚠️ **Important**: Also configure Huawei Cloud Security Groups through the console to allow inbound traffic on port 8000.

### Environment Variables

Secure configuration using environment files:

```bash
# Create environment file
cat > /opt/mcp-server/.env << 'EOF'
# Environment configuration
MCP_LOG_LEVEL=INFO
MCP_MAX_DATAFRAME_SIZE_MB=100
MCP_MAX_FILE_SIZE_MB=100
# Add additional sensitive configurations as needed
EOF

# Secure the file
chmod 600 /opt/mcp-server/.env
```

## 📊 Service Access

### Access Points

Once deployed, the service is accessible via:

- **Internal Access**: `http://localhost:8000` (from within the server)
- **External Access**: `http://[YOUR_ECS_PUBLIC_IP]:8000` (requires security group configuration)

### Available Endpoints

| Endpoint | Description |
|----------|-------------|
| `/sse` | MCP Server-Sent Events endpoint |
| `/` | Base endpoint for service verification |

## ✅ Deployment Verification Checklist

Ensure successful deployment with these verification steps:

- [ ] Docker image successfully pulled from SWR
- [ ] File permissions properly configured
- [ ] Container running without errors
- [ ] No permission denied errors in logs
- [ ] Log files being written and updated
- [ ] Port 8000 responding to requests
- [ ] Security groups configured correctly
- [ ] Environment variables loaded properly

## 📝 Operational Notes

### Key System Behaviors

- **Session Management**: Sessions expire after 30 minutes of inactivity
- **Log Rotation**: Automatic rotation with 5 files, 10MB each
- **Auto-Restart**: Container automatically restarts on failure (unless-stopped policy)
- **Data Persistence**: All data persists in mounted volumes

### Maintenance Recommendations

1. **Regular Monitoring**: Check logs daily for errors or warnings
2. **Update Schedule**: Plan regular updates during maintenance windows
3. **Backup Strategy**: Implement regular backups of `/opt/mcp-server/data`
4. **Resource Monitoring**: Track CPU and memory usage trends
5. **Security Updates**: Keep Docker and system packages updated

## 🔍 Debugging Workflow

### Systematic Troubleshooting Approach

1. **Check Service Status**
   ```bash
   docker-compose ps
   ```

2. **Review Logs**
   ```bash
   docker-compose logs --tail=50 pandas-mcp-server
   ```

3. **Verify Permissions**
   ```bash
   ls -la /opt/mcp-server/
   ```

4. **Test Connectivity**
   ```bash
   curl -I http://localhost:8000/sse
   ```

5. **Resource Check**
   ```bash
   df -h && free -h
   ```

6. **Network Verification**
   ```bash
   netstat -tlnp | grep 8000
   ```

## 🎯 Best Practices

### Deployment Recommendations

- **Use Version Tags**: Specify exact versions instead of `latest` for production
- **Monitor Resources**: Set up CloudEye monitoring for proactive alerts
- **Document Changes**: Maintain a deployment log for troubleshooting
- **Test Updates**: Always test updates in staging before production
- **Secure Access**: Use HTTPS proxy for external access in production

### Performance Optimization

- Configure appropriate resource limits in Docker Compose
- Implement caching strategies for frequently accessed data
- Use volume mounts for better I/O performance
- Consider horizontal scaling for high-load scenarios

---

## 📚 Additional Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Huawei ECS Best Practices](https://support.huaweicloud.com/intl/en-us/ecs/)
- [MCP Protocol Specification](https://github.com/anthropics/mcp)

---

*📅 Document Version: 1.0*  
*🔄 Last Updated: 09 September, 2025*