# 🚢 Pushing Docker Images to Huawei SWR - A Survivor's Guide

*Because apparently, Docker Hub was too mainstream for us.*

## 📋 Prerequisites

- Docker installed locally (shocking, I know)
- A Huawei Cloud account with SWR access
- Your sanity (optional, you'll lose it anyway)
- Coffee ☕ (mandatory)

## 🎯 The Journey to SWR

### Step 1: Set Up Your SWR Namespace

First, navigate to Huawei Cloud Console → Container Registry (SWR) → Organization Management

```bash
# Your organization/namespace will look like:
# swr.ap-southeast-3.myhuaweicloud.com/YOUR_NAMESPACE/YOUR_IMAGE_NAME

# Example:
# swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server
```

*Pro tip: Write this down. You'll type it wrong at least 5 times.*

### Step 2: Build Your Docker Image (The Easy Part)

```bash
# Build your masterpiece
docker build -t pandas-mcp-server:latest .

# Verify it actually built
docker images | grep pandas-mcp-server
# No results? Check your Dockerfile exists. Yes, really.
```

### Step 3: The Authentication Tango 💃

Here's where Huawei gets creative with security:

#### Option A: The Console Method (For People Who Like Clicking)
1. Go to SWR Console → Access Credentials
2. Click "Obtain Login Command"
3. Copy that impossibly long command
4. Paste it in terminal
5. Hope it works

#### Option B: The CLI Method (For Terminal Warriors)
```bash
# The format Huawei wants (yes, it's weird)
docker login -u [REGION]@[ACCESS_KEY] -p [SECRET_KEY] swr.[REGION].myhuaweicloud.com

# Real example:
docker login -u ap-southeast-3@ABCD1234 -p [YOUR_SECRET_TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

*Note: Your login expires every 12 hours. Because permanent tokens are apparently too convenient.*

#### When Login Fails (And It Will)

```bash
# Error: "unauthorized: authentication required"?
# Translation: Your token expired. Again.

# Step 1: Get a fresh token from console
# Step 2: Clear Docker's confusion
docker logout swr.ap-southeast-3.myhuaweicloud.com

# Step 3: Try again with fresh credentials
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

#### The Nuclear Option 💣 (When Nothing Else Works)

Sometimes Docker's credential helper gets so confused it needs a complete reset. Time for the nuclear option:

```bash
# ⚠️ WARNING: This will nuke ALL your Docker logins (Docker Hub, SWR, everything)

# Step 1: Find out what cursed credential helper you're using
cat ~/.docker/config.json | grep credsStore
# See "desktop", "wincred", "osxkeychain", or "pass"? That's your problem.

# Step 2: The Complete Credential Genocide
# macOS users:
docker-credential-osxkeychain erase < /dev/null

# Linux users with pass:
docker-credential-pass erase < /dev/null

# Windows users (in PowerShell as admin):
docker-credential-wincred erase

# Step 3: Nuke the config file itself
rm ~/.docker/config.json
# Or if you're feeling less destructive:
mv ~/.docker/config.json ~/.docker/config.json.backup

# Step 4: Restart Docker (because why not)
# macOS:
killall Docker && open -a Docker

# Linux:
sudo systemctl restart docker

# Windows:
# Restart Docker Desktop from the system tray like a civilized person

# Step 5: Start fresh
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com

# Still failing? Time for the ULTIMATE nuclear option:
# Warning: This removes EVERYTHING Docker-related
docker system prune -a --volumes --force
rm -rf ~/.docker
# Reinstall Docker
# Question your career choices
```

**Credential Helper Still Being Annoying?**

Disable it entirely and go manual:

```bash
# Edit ~/.docker/config.json
{
  "auths": {
    "swr.ap-southeast-3.myhuaweicloud.com": {
      "auth": "BASE64_ENCODED_CREDENTIALS"
    }
  }
  // DELETE any "credsStore" or "credHelpers" lines
}

# Generate the auth manually (because we're savages now)
echo -n "ap-southeast-3@YOUR_KEY:YOUR_TOKEN" | base64
# Paste that base64 string in the "auth" field above
```

### Step 4: Tag Your Image (The Naming Ceremony)

This is where you tell your local image about its new cloud home:

```bash
# The sacred tagging ritual
docker tag [LOCAL_IMAGE]:[TAG] [SWR_FULL_PATH]:[TAG]

# Example that actually works:
docker tag pandas-mcp-server:latest swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Also tag as version number (because latest isn't always best)
docker tag pandas-mcp-server:latest swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:v1.0.0

# Verify your tags (you probably typo'd something)
docker images | grep swr
```

### Step 5: The Push (Where Dreams Go to Die... Slowly)

```bash
# Push to SWR (grab a snack, this takes a while)
docker push swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Watch the layers upload one... by... one...
# "Retrying in 5 seconds" is normal. Apparently.
```

**Common Push Issues:**

```bash
# "denied: requested access to the resource is denied"
# Translation: You forgot to login, or it expired (again)
docker login -u ap-southeast-3@[KEY] -p [TOKEN] swr.ap-southeast-3.myhuaweicloud.com

# "manifest invalid: manifest size exceeds the limit"
# Translation: Your image is thicc. Slim it down.
docker images | grep your-image  # Check the size

# "timeout" errors?
# Just retry. It's Huawei's servers, not you (probably)
docker push swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest
```

### Step 6: Verify Your Image Made It

Go to SWR Console → Image Repository → Your Repository

*Or if you trust the CLI more:*

```bash
# You can't actually list remote images via Docker CLI with SWR
# Just check the console like a normal person
# I know, it's 2024 and we still can't have nice things
```

## 📝 The Complete Workflow Script

Here's everything in one place (for copy-paste heroes):

```bash
#!/bin/bash
# deploy-to-swr.sh - Your one-stop shop for SWR deployment

# Configuration (CHANGE THESE!)
NAMESPACE="wooyankit"
IMAGE_NAME="pandas-mcp-server"
VERSION="v1.0.0"
REGION="ap-southeast-3"
SWR_URL="swr.${REGION}.myhuaweicloud.com"

# Build the image
echo "🔨 Building image..."
docker build -t ${IMAGE_NAME}:${VERSION} .

# Login to SWR (you'll need to add your credentials)
echo "🔐 Logging in to SWR..."
# Get this command from Huawei Console → SWR → Access Credentials
# docker login -u ${REGION}@YOUR_KEY -p YOUR_TOKEN ${SWR_URL}

# Tag for SWR
echo "🏷️ Tagging image..."
docker tag ${IMAGE_NAME}:${VERSION} ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}
docker tag ${IMAGE_NAME}:${VERSION} ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest

# Push to SWR
echo "🚀 Pushing to SWR (this might take a while)..."
docker push ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}
docker push ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest

echo "✅ Done! Image available at:"
echo "   ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:${VERSION}"
echo "   ${SWR_URL}/${NAMESPACE}/${IMAGE_NAME}:latest"
```

## 🏃‍♂️ Speed Run Tips

1. **Multi-stage builds**: Your image is probably bloated
   ```dockerfile
   # Use multi-stage to keep it slim
   FROM python:3.10 AS builder
   # ... build stuff ...
   
   FROM python:3.10-slim
   # ... only copy what you need ...
   ```

2. **Layer caching**: Order your Dockerfile wisely
   ```dockerfile
   # Put things that change less frequently first
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   # THEN copy your frequently-changing code
   COPY . .
   ```

3. **Use .dockerignore**: Don't push your entire git history
   ```bash
   # .dockerignore
   .git
   *.pyc
   __pycache__
   .env
   logs/
   data/
   ```

## 🎭 Regional Considerations

Different regions, different URLs (because consistency is overrated):

```bash
# Singapore
swr.ap-southeast-3.myhuaweicloud.com

# Hong Kong  
swr.ap-southeast-1.myhuaweicloud.com

# Shanghai
swr.cn-east-3.myhuaweicloud.com

# Just check your console URL to be sure
```

## 📊 Image Management Pro Tips

### Versioning Strategy (Because "latest" Isn't a Strategy)

```bash
# Semantic versioning is your friend
docker tag myapp:latest swr.../myapp:v1.0.0  # Major release
docker tag myapp:latest swr.../myapp:v1.0.1  # Bug fix
docker tag myapp:latest swr.../myapp:v1.1.0  # Feature addition

# Also maintain these tags
docker tag myapp:latest swr.../myapp:latest   # Latest stable
docker tag myapp:latest swr.../myapp:dev      # Development version
```

### Cleanup Your Local Images (Your Disk Will Thank You)

```bash
# Remove old versions locally
docker image prune -a

# Remove specific images
docker rmi swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:old-version

# Nuclear option: remove everything
docker system prune -a --volumes
# (Don't run this in production, genius)
```

## 🚨 Troubleshooting Hall of Fame

**"Login succeeded" but push still fails:**
- Your token expired during the push (yes, really)
- Solution: Login again, push faster

**Push succeeds but image doesn't appear in console:**
- You're probably looking in the wrong region
- Or the console is being slow (refresh harder)

**"Layer already exists" but push is still slow:**
- Welcome to Huawei's definition of "already exists"
- Just wait, it'll finish eventually

**"Connection reset by peer":**
- Huawei's servers are having a moment
- Try again in 5 minutes (or move to Singapore)

## ✅ Success Checklist

- [ ] Image builds locally without errors
- [ ] Image size is under 1GB (preferably under 500MB)
- [ ] Successfully authenticated to SWR (for now)
- [ ] Image is tagged with SWR URL
- [ ] Push completed without timeout
- [ ] Image appears in SWR console
- [ ] You haven't rage-quit yet

## 🎬 Final Thoughts

Congratulations! You've successfully pushed an image to Huawei SWR, which is like pushing to Docker Hub but with extra steps and occasional timeouts. Your image is now ready to be pulled by your ECS instances, assuming they can authenticate too.

Remember:
- Always tag with versions, not just `latest`
- Login tokens expire faster than your motivation
- The console is your friend (when it works)
- When in doubt, check the region

---

**May your pushes be swift and your tokens unexpired! 🚀**

*P.S. - If you're reading this at 3 AM debugging authentication issues, you're not alone. We've all been there.*