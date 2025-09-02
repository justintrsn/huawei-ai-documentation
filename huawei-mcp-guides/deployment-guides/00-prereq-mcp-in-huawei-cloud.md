# 🚀 MCP and Huawei ECS Deployment Guide
## Technical Documentation for Model Context Protocol and Elastic Cloud Server Integration

## 📚 Table of Contents

- [Overview](#overview)
- [Model Context Protocol (MCP)](#model-context-protocol-mcp)
- [Huawei Elastic Cloud Server (ECS)](#huawei-elastic-cloud-server-ecs)
- [Integration Architecture](#integration-architecture)
- [Deployment Resources](#deployment-resources)
- [Best Practices](#best-practices)
- [Troubleshooting Guide](#troubleshooting-guide)

---

## 🎯 Overview

This documentation provides comprehensive guidance for implementing and deploying Model Context Protocol (MCP) servers on Huawei Cloud's Elastic Cloud Server (ECS) infrastructure. The guide covers both the protocol implementation and the infrastructure deployment requirements.

### 🛠️ Technology Stack

- **Model Context Protocol (MCP)**: Anthropic's standardized protocol for AI model tool integration
- **Huawei ECS**: Virtual machine instances for hosting containerized applications
- **Docker**: Container runtime for application deployment
- **SWR**: Huawei's Software Repository for container image management

---

## 🤖 Model Context Protocol (MCP)

### Introduction

The Model Context Protocol (MCP) is a standardized communication protocol developed by Anthropic that enables AI models to interact with external tools and services. It provides a consistent interface for tool discovery, execution, and result handling.

### ⚡ Core Features

MCP provides the following capabilities:
- ✅ Standardized function exposure to AI models
- ✅ Built-in schema validation for input/output
- ✅ Support for Server-Sent Events (SSE) for real-time communication
- ✅ Automatic tool discovery mechanisms
- ✅ Comprehensive error handling framework

### 💻 Implementation Example

```python
from fastmcp import FastMCP

# Initialize MCP server
mcp = FastMCP("my-server")

@mcp.tool()
async def process_data(input: str) -> str:
    """Process input data and return results"""
    # Implementation logic
    return processed_result
```

### 🏗️ Protocol Architecture

The MCP lifecycle consists of three primary phases:

**📍 Phase 1: Initialization**
```python
from fastmcp import FastMCP
mcp = FastMCP("server-name")
```

**📍 Phase 2: Error Handling Implementation**
```python
try:
    result = await tool.execute()
except MCPError as e:
    logger.error(f"MCP execution error: {e}")
    # Implement retry logic or fallback
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    # Handle general exceptions
```

**📍 Phase 3: Production Deployment**
- Implement comprehensive logging
- Add monitoring and alerting
- Document all custom tools and their schemas
- Maintain version compatibility

### 🎯 Key Capabilities

- **Claude Integration**: Native support for Anthropic's Claude models
- **Schema Validation**: Automatic input/output validation based on defined schemas
- **Streaming Support**: Real-time data streaming capabilities
- **Tool Discovery**: Automatic tool registration and discovery mechanisms

---

## ☁️ Huawei Elastic Cloud Server (ECS)

### Service Overview

Huawei's Elastic Cloud Server (ECS) provides virtual machine instances for hosting applications. It's important to note that Huawei ECS is a virtual server solution, not a container orchestration service like AWS ECS.

### ⚠️ Service Clarification

**Important Distinction:**
- **Huawei ECS**: Elastic Cloud Server - Virtual machine instances
- **Huawei CCE**: Cloud Container Engine - Kubernetes-based container orchestration
- **AWS ECS**: Elastic Container Service - Container orchestration platform

📝 **Note**: For containerized workloads requiring orchestration, consider using Huawei CCE instead of ECS.

### 🖥️ ECS Instance Configuration

Standard configuration for MCP server deployment:

```yaml
Instance Specifications:
  Type: s6.large.2
  Operating System: Ubuntu 20.04 LTS
  
Network Configuration:
  VPC: vpc-default
  Subnet: subnet-az1
  Security Group: Custom configuration required
  Elastic IP: Required for external access
  
Storage Configuration:
  System Disk: 40GB minimum
  Data Disk: 100GB recommended
```

### 🐳 Docker Deployment Process

Manual Docker installation and configuration on ECS:

```bash
# Connect to ECS instance
ssh root@[elastic-ip-address]

# Install Docker
curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker

# Configure SWR authentication
docker login swr.[region].myhuaweicloud.com

# Deploy container
docker pull swr.[region].myhuaweicloud.com/[organization]/[image]:[tag]
docker run -d -p 8000:8000 --restart=unless-stopped [image-id]
```

### 📊 Huawei Cloud Services Reference

Essential services for MCP deployment:

- **ECS**: Elastic Cloud Server - Virtual machine instances
- **CCE**: Cloud Container Engine - Kubernetes orchestration
- **SWR**: Software Repository - Container registry
- **OBS**: Object Storage Service - Object storage solution
- **VPC**: Virtual Private Cloud - Network isolation
- **EIP**: Elastic IP - Public IP addresses
- **IAM**: Identity and Access Management - Access control
- **Cloud Eye**: Monitoring and metrics service

---

## 🏛️ Integration Architecture

### System Architecture

The integration between MCP and ECS follows this architecture:

```
┌────────────────┐
│   AI Model     │
└────────┬───────┘
         │ MCP Protocol
         ▼
┌────────────────┐
│   ECS Instance │
│  ┌──────────┐  │
│  │MCP Server│  │
│  └──────────┘  │
└────────────────┘
         │
         ▼
    [Data Sources]
```

### 🚀 Deployment Configuration

**🔧 Development Environment:**
```bash
python server.py
# Verify server startup and tool registration
```

**🏭 Production Deployment on Huawei ECS:**

1. **Instance Preparation**
   - SSH access configuration
   - Docker installation
   - Security group configuration

2. **SWR Configuration**
   ```bash
   docker login swr.ap-southeast-3.myhuaweicloud.com
   # Username format: [region]@[account-id]
   # Password: IAM user password
   ```

3. **Container Deployment**
   ```bash
   docker pull swr.[region].myhuaweicloud.com/[org]/[image]:[version]
   docker run -d -p 8000:8000 --restart=unless-stopped [image]
   ```

---

## 📦 Deployment Resources

### 📂 Repository References

#### 🐼 Pandas MCP Server
Repository: [https://github.com/justintrsn/pandas-mcp-server](https://github.com/justintrsn/pandas-mcp-server)

**Features:**
- Pandas data processing operations
- Session management
- Error handling and recovery
- Chart generation capabilities

#### 🎨 Streamlit MCP Client
Repository: [https://github.com/justintrsn/streamlit-pandas-mcp-client](https://github.com/justintrsn/streamlit-pandas-mcp-client)

**Features:**
- Web-based user interface
- Real-time data updates
- File upload functionality
- Data visualization

### 📖 Deployment Guides

1. **🐳 Docker Image Management with Huawei SWR**
   - Authentication configuration
   - Regional endpoint setup
   - Image push and pull procedures
   - Registry management

2. **☁️ ECS Deployment Procedures**
   - Instance provisioning
   - Container deployment
   - Service configuration
   - Monitoring setup

### 🌐 Live Application

Production deployment: [https://huawei-pandas-mcp-client.streamlit.app/](https://huawei-pandas-mcp-client.streamlit.app/)

---

## ✨ Best Practices

### 💡 MCP Development Guidelines

**✅ Recommended Practices:**
- Implement comprehensive unit and integration testing
- Add timeout handling for all external operations
- Maintain detailed logging for debugging
- Version all tool schemas and maintain backward compatibility
- Implement retry logic with exponential backoff

**⚠️ Key Considerations:**
- Validate all inputs from AI models
- Implement connection resilience mechanisms
- Include comprehensive error handling
- Schedule deployments during maintenance windows

### ☁️ Huawei Cloud Deployment Guidelines

**✅ Recommended Practices:**
- Use CCE for production container orchestration requirements
- Consider Function Graph for serverless workloads
- Maintain documentation of SWR endpoints for each region
- Document security group configurations
- Implement monitoring using Cloud Eye
- Use bastion hosts for secure SSH access

**⚠️ Key Considerations:**
- Understand the distinction between Huawei ECS and container orchestration services
- Include region prefixes in all SWR URLs
- Implement proper IAM password rotation policies
- Use specific version tags instead of "latest" in production
- Configure comprehensive monitoring and alerting
- Properly configure security groups for required access

---

## 🔧 Troubleshooting Guide

### 🚨 Common Issues and Solutions

**❌ MCP Server Startup Failures**
- Verify all required environment variables are set
- Check Docker container logs for initialization errors
- Confirm port availability and binding

**❌ ECS Instance Instability**
- Review Cloud Eye metrics for resource utilization
- Check disk space availability
- Monitor memory usage patterns
- Verify container health checks

**❌ Network Connectivity Issues**
- Verify security group rules for both VPC and ECS levels
- Confirm EIP attachment and routing
- Check application port bindings

**❌ Local vs. Production Discrepancies**
- Verify environment variable configuration
- Check network policies and firewall rules
- Confirm SWR image versions match development

**❌ SWR Push Failures**
- Verify endpoint format: `swr.[region].myhuaweicloud.com`
- Confirm IAM credentials and permissions
- Check image size and layer limits

**❌ HTTP 502 Errors**
- Verify container status via SSH
- Check Docker logs for application errors
- Confirm health check configurations

**❌ SSH Connection Issues**
- Verify EIP attachment to instance
- Confirm security group allows port 22
- Validate SSH key pair configuration

**🔍 General Debugging Approach**
1. Check Cloud Eye logs and metrics
2. Review container logs via Docker
3. Verify network configuration
4. Confirm security group rules
5. Validate IAM permissions

---

## 📝 Conclusion

This guide provides the technical foundation for deploying MCP servers on Huawei Cloud ECS infrastructure. Successful implementation requires careful attention to both protocol requirements and infrastructure configuration. For production deployments, consider using Huawei CCE for improved container orchestration capabilities or Function Graph for serverless architectures.

---

## 📚 Additional Resources

- [🐳 Docker Image Management with Huawei SWR](/huawei-mcp-guides/deployment-guides/01-swr-docker-push.md)
- [🚀 MCP Server Deployment on ECS](/huawei-mcp-guides/deployment-guides/02-ecs-deployment.md)
- [🌐 Streamlit Web Application](https://huawei-pandas-mcp-client.streamlit.app/)

---

*📅 Documentation Version: 1.0*  
*🔄 Last Updated: 09 September, 2025*