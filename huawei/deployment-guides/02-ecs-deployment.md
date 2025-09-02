# 🎭 MCP Server Deployment Guide on ECS

*Because deploying containers should totally be this complicated...*

## 📋 Prerequisites (The Boring Stuff)

- A Huawei Cloud ECS instance with Linux
- Docker and Docker Compose 
- Access to your SWR registry (and the patience to deal with it)
- Root access (because who needs security, right?)

## 🎯 Quick Start (Spoiler: It's Not That Quick)

### 1. SSH Into Your Server

```bash
ssh root@your-ecs-ip
# Living dangerously with root, I see
```

### 2. Create Your Directories

```bash
# Make the base directory
mkdir -p /opt/mcp-server
cd /opt/mcp-server

# Create ALL the subdirectories (yes, all of them)
mkdir -p data charts logs temp config

# Set permissions (this definitely won't cause issues later... 😏)
chmod -R 755 data charts logs temp config
```

### 3. The Docker Compose YAML 

Create `/opt/mcp-server/docker-compose.yml` and pray to the YAML gods you don't mess up the indentation:

```yaml
# No version field! Docker Compose will passive-aggressively warn you otherwise
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
      - MCP_LOG_LEVEL=INFO  # DEBUG if you enjoy suffering
      - MCP_MAX_DATAFRAME_SIZE_MB=100
      - MCP_MAX_FILE_SIZE_MB=100
    volumes:
      - ./data:/app/data        # Your precious data
      - ./charts:/app/charts    # Pretty pictures
      - ./logs:/app/logs        # Where dreams go to die
      - ./temp:/tmp            # Temporary... sure...
      - ./config:/app/config   # Config you'll never use
    restart: unless-stopped     # It'll stop when it wants to
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

### 4. SWR Login

Oh boy, here's where the fun begins! Huawei's SWR login is... special.

```bash
# The magic incantation (replace YOUR_ACCOUNT_ID, obviously)
docker login -u ap-southeast-3@YOUR_ACCOUNT_ID swr.ap-southeast-3.myhuaweicloud.com
```

**🚨 When Docker Login Decides to Be Difficult:**

Getting "unauthorized" or "authentication required"? Welcome to the club!

#### Option A: The "Did You Get Your Password Right?" Check
```bash
# Get your login key from Huawei Cloud Console:
# Console → Container Registry → Access Keys → Get Login Command
# It'll look something like this monstrosity:
docker login -u ap-southeast-3@ABCDEF1234567 -p [SOME_IMPOSSIBLY_LONG_TOKEN] swr.ap-southeast-3.myhuaweicloud.com
```

#### Option B: The "My Credentials Expired Again" Fix
```bash
# Your temporary login expires every 12 hours (because security!)
# Just regenerate it in the console. Again. And again. And again.
```

#### Option C: The "Wrong Region" Facepalm
```bash
# Make sure you're using the right region in the URL
# ap-southeast-3 = Singapore (not ap-southeast-1, that's somewhere else)
# Check your SWR console URL, genius
```

#### Option D: The Nuclear Option (My Favorite)
```bash
# Clear Docker's confused credentials
rm ~/.docker/config.json
# Try logging in again
# Pray to your deity of choice
```

### 5. Pull That Image Like Your Life Depends On It

```bash
# Finally pull the image (this might take a while, grab coffee ☕)
docker pull swr.ap-southeast-3.myhuaweicloud.com/wooyankit/pandas-mcp-server:latest

# Make sure it actually downloaded
docker images | grep pandas-mcp-server
# No results? Back to step 4, my friend
```

### 6. The Permission Wars 

Here's where Linux decides to remind you who's boss:

```bash
# Create the sacred log file
touch /opt/mcp-server/logs/pandas_mcp.log
chmod 666 /opt/mcp-server/logs/pandas_mcp.log  # Christians beware

# Try the "proper" way first
chown -R 1000:1000 /opt/mcp-server/logs
chown -R 1000:1000 /opt/mcp-server/data
chown -R 1000:1000 /opt/mcp-server/charts
chown -R 1000:1000 /opt/mcp-server/temp
chown -R 1000:1000 /opt/mcp-server/config

# Still getting permission errors? Time for the nuclear option:
chmod -R 777 /opt/mcp-server/logs    # Security? Never heard of her
chmod -R 777 /opt/mcp-server/data
chmod -R 777 /opt/mcp-server/charts
# Yes, 777. Fight me.
```

### 7. Fire It Up! 🚀

```bash
# The moment of truth
docker-compose up -d

# Is it actually running or just pretending?
docker-compose ps

# Let's see what disasters await in the logs
docker-compose logs -f --tail=100
# Errors? Shocked. Absolutely shocked.
```

### 8. Verify It's Actually Working

```bash
# Check if port 8000 is doing something
curl http://localhost:8000/sse
# Should see some SSE stuff, not a 404

# Are logs being written or is it all a lie?
ls -la /opt/mcp-server/logs/
tail -f /opt/mcp-server/logs/pandas_mcp.log
# No logs? Check step 6 again. Yes, again.
```

## 📁 What You Should Have (If Everything Went Right)

```
/opt/mcp-server/
├── docker-compose.yml    # Your configuration masterpiece
├── data/                 # Where data goes to live
├── charts/              # Pretty visualizations no one will look at
├── logs/                # Evidence of your failures
│   └── pandas_mcp.log  # The chronicle of despair
├── temp/                # "Temporary" files from 2023
└── config/              # Configurations you'll never change
```

## 🔧 Commands to Pretend You Know What You're Doing

### Watching Things Break in Real-Time

```bash
# Watch the logs scroll by
docker-compose logs -f pandas-mcp-server

# See if it's still alive
docker-compose ps

# Check how much RAM it's eating
docker stats pandas-mcp-server

# Read the application's diary
tail -f /opt/mcp-server/logs/pandas_mcp.log
```

### Turning It Off and On Again (The IT Classic)

```bash
# Stop everything
docker-compose down

# The good ol' restart
docker-compose restart

# Update (because the old version was "fine")
docker-compose pull
docker-compose up -d

# Poke around inside the container
docker-compose exec pandas-mcp-server /bin/bash
# Type 'exit' to escape
```

## 🚨 When Things Go Wrong (And They Will)

### The Classic "Permission Denied" Error

*Ah yes, the Linux special. Here we go again:*

```bash
# Stop the madness
docker-compose down

# Make everything world-writable (your security team will love this)
chmod -R 777 /opt/mcp-server/logs
touch /opt/mcp-server/logs/pandas_mcp.log
chmod 666 /opt/mcp-server/logs/pandas_mcp.log

# Try again
docker-compose up -d
# Still broken? Welcome to Linux!
```

### Docker Compose Being Dramatic About Versions

```bash
# Update Docker Compose (because the old one is "obsolete" apparently)
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -sf /usr/local/bin/docker-compose /usr/bin/docker-compose

# Check if it's happy now
docker-compose --version
```

### Container Won't Start (Surprise!)

```bash
# See what wisdom the logs offer
docker-compose logs --tail=200

# Inspect the crime scene
docker inspect pandas-mcp-server

# Is the disk full? (It's always the disk)
df -h

# Out of memory? (It's never enough)
free -h
```

## 🔐 Security (Optional)

### Firewall Configuration (Because We Care™)

```bash
# Open port 8000 (what could go wrong?)
sudo ufw allow 8000/tcp
sudo ufw reload
```

**Don't forget:** Also configure Huawei Cloud Security Groups in their console. Yes, you need to do it in two places. No, I don't know why.

### Environment Variables (For "Security")

```bash
# Create a super secret .env file
cat > /opt/mcp-server/.env << 'EOF'
# Definitely don't commit this to git
OPENAI_API_KEY=sk-totally-not-sharing-this
MCP_LOG_LEVEL=INFO  # Change to DEBUG when desperate
EOF

# "Secure" it
chmod 600 /opt/mcp-server/.env
# Now only root can see your secrets. Feel safer?
```

## 📊 Accessing Your Glorious Creation

Once it's miraculously running:

- **Internal:** `http://localhost:8000` (from the server)
- **External:** `http://YOUR_ECS_PUBLIC_IP:8000` (from the internet, if the stars align)

### Endpoints That Actually Exist

- `/sse` - The MCP SSE endpoint (the only one that matters)
- ~~`/health`~~ - Just kidding, this doesn't exist. Surprise!
- `/` - Probably returns something. Maybe.

## 🎉 The "It's Actually Working" Checklist

✅ Docker pulled the image without crying  
✅ Permissions are "fixed" (777 everything, right?)  
✅ Container is running (allegedly)  
✅ No permission denied errors (miracle!)  
✅ Logs exist and are growing (good sign!)  
✅ Port 8000 responds to curl (victory!)  
✅ You haven't thrown your laptop out the window (yet)  

## 📝 Fun Facts

- Sessions expire after 30 minutes (because nothing lasts forever)
- Logs rotate automatically (5 files, 10MB each, then poof!)
- Container auto-restarts (unless it really doesn't want to)
- All your data persists in volumes (until you accidentally `docker-compose down -v`)

## 🆘 Troubleshooting Flowchart

1. Is it broken? → Yes → Check the logs
2. Are the logs helpful? → No → Add more logging
3. Still broken? → Yes → Check permissions
4. Fixed? → No → chmod 777 everything
5. Still broken? → Yes → Restart Docker
6. Still broken? → Yes → Restart the server
7. Still broken? → Yes → Question your life choices
8. Still broken? → Yes → Ask Justin Trisno
9. Not resolved? → Give up and cry 

## 🏁 Final Words

Congratulations! You've successfully deployed a containerized application in 2024, which is somehow still as painful as it was in 2014. Your mcp-server is now running (hopefully), ready to process things for a greater purpose(or at least fill up your disk).

Remember: If it works, don't touch it. If it doesn't work, try turning it off and on again.

---

**May the logs be ever in your favor! 🎭**