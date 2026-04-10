# RentEase EC2 Deployment Guide

## **PART 1: Launch EC2 Instance**

### Step 1: Go to AWS Console
1. Open [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. Login with your AWS account
3. Navigate to **EC2** service

### Step 2: Launch Instance
1. Click **Launch instances** button
2. Select Image: **Ubuntu 24.04 LTS** (free tier eligible)
3. Instance Type: **t3.micro** (free tier)
4. Key Pair: 
   - Click **Create new key pair**
   - Name: `rentease-key`
   - Format: `.pem`
   - Click **Create key pair**
   - File will download automatically (save it safely!)
5. Network Settings:
   - Allow SSH traffic from: **Anywhere (0.0.0.0/0)**
   - Allow HTTPS traffic: **Check**
   - Allow HTTP traffic: **Check**
6. Storage: 8 GB (default)
7. Click **Launch instance**

### Step 3: Wait for Instance to Start
1. Instance will show **Running** status after 1-2 minutes
2. Note the **Public IPv4 address** (e.g., `54.123.45.67`)

---

## **PART 2: Connect to EC2 via SSH**

### Step 4: Connect from Mac Terminal
```bash
# Navigate to where you saved the key pair
cd ~/Downloads

# Set correct permissions
chmod 400 rentease-key.pem

# Connect to EC2 (replace 54.123.45.67 with your IP)
ssh -i rentease-key.pem ubuntu@54.123.45.67
```

You should see:
```
ubuntu@ip-172-31-xx-xx:~$
```

---

## **PART 3: Install Docker & Docker Compose on EC2**

### Step 5: Update System
```bash
sudo apt update
sudo apt upgrade -y
```

### Step 6: Install Docker
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Verify Docker
docker --version
```

Logout and login again to apply group changes:
```bash
exit
ssh -i rentease-key.pem ubuntu@54.123.45.67
```

### Step 7: Install Docker Compose
```bash
# Download Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker-compose --version
```

---

## **PART 4: Setup Application on EC2**

### Step 8: Create Project Directory
```bash
mkdir -p ~/rentease
cd ~/rentease
```

### Step 9: Create docker-compose.yml
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:

  postgres:
    image: postgres:16-alpine
    container_name: rentease-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_DB: rentease
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: durmuhammad/rentease-backend:v1.0
    container_name: rentease-backend
    environment:
      DATABASE_URL: postgresql://postgres:postgres123@postgres:5432/rentease
      SECRET_KEY: your-secret-key-change-this-in-production
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy

  frontend:
    image: durmuhammad/rentease-frontend:v1.0
    container_name: rentease-frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  postgres_data:
    driver: local
EOF
```

### Step 10: Verify File Created
```bash
cat docker-compose.yml
```

---

## **PART 5: Start Services on EC2**

### Step 11: Pull Images from Docker Hub
```bash
docker-compose pull
```

### Step 12: Run All Services
```bash
docker-compose up -d
```

### Step 13: Verify Services Running
```bash
# Check status
docker-compose ps

# Should show:
# rentease-postgres   Up (healthy)
# rentease-backend    Up
# rentease-frontend   Up
```

### Step 14: Check Logs
```bash
# View all logs
docker-compose logs -f

# View backend logs only
docker-compose logs backend

# View frontend logs only
docker-compose logs frontend
```

Press **Ctrl+C** to exit logs.

---

## **PART 6: Access Application**

### Step 15: Get Your EC2 Public IP
```bash
# From SSH terminal
hostname -I

# Or check AWS Console for "Public IPv4 address"
```

### Step 16: Open in Browser
```
Frontend: http://your-ec2-ip
API Docs: http://your-ec2-ip:8000/docs
Database: localhost:5432 (from EC2)
```

Example: `http://54.123.45.67`

---

## **PART 7: Database Management**

### Step 17: Access PostgreSQL
```bash
# Connect to database
docker-compose exec postgres psql -U postgres -d rentease

# View tables
\dt

# Exit
\q
```

### Step 18: Backup Data
```bash
# Backup database
docker-compose exec postgres pg_dump -U postgres -d rentease > backup.sql

# Restore from backup
docker-compose exec -T postgres psql -U postgres -d rentease < backup.sql
```

---

## **PART 8: Useful Commands**

### View Running Containers
```bash
docker-compose ps
```

### Restart Services
```bash
docker-compose restart
```

### Stop All Services (data persists)
```bash
docker-compose down
```

### Stop & Remove Everything (volume persists)
```bash
docker-compose down
```

### Remove Volume (WARNING: deletes database!)
```bash
docker volume rm rentease_postgres_data
```

### View Backend Logs
```bash
docker-compose logs backend
docker-compose logs -f backend  # Follow logs
```

### Check Volume
```bash
docker volume ls
docker volume inspect rentease_postgres_data
```

### Update Image
```bash
# Rebuild/pull latest
docker-compose pull
docker-compose up -d
```

---

## **PART 9: Production Setup (Optional)**

### Step 19: Setup Domain (if you have one)
1. Point domain DNS to EC2 public IP
2. Update frontend API URL to your domain

### Step 20: Setup SSL/HTTPS
```bash
# Install Certbot
sudo apt install certbot python3-certbot-apache -y

# Get certificate
sudo certbot certonly --standalone -d your-domain.com

# Configure Apache to use SSL
```

### Step 21: Monitor Application
```bash
# Check disk space
df -h

# Check memory usage
free -h

# Check CPU usage
top
```

---

## **Troubleshooting**

### Port Already in Use
```bash
# If port 80 is in use
docker-compose down
# Wait 30 seconds
docker-compose up -d
```

### Database Connection Error
```bash
# Check PostgreSQL is healthy
docker-compose logs postgres

# Restart PostgreSQL
docker-compose restart postgres
```

### Container Exits Immediately
```bash
# Check logs
docker-compose logs backend

# Verify environment variables
docker-compose config
```

### Cannot Access Frontend
- Verify security group allows port 80
- Check EC2 instance status is "Running"
- Try accessing: `http://ec2-ip:8000` (backend should respond)

---

## **Summary**

✅ EC2 Instance Launched
✅ Docker & Docker Compose Installed
✅ Images Pulled from Docker Hub
✅ All Services Running
✅ Database Persistent
✅ Application Accessible

🎉 **Your RentEase application is now on AWS EC2!**

---

## **Cost Estimate (AWS Free Tier)**
- EC2 t3.micro: FREE (12 months)
- EBS Storage 8GB: FREE (12 months)
- Data Transfer: Mostly free
- **Total: $0/month** (for 12 months)

After free tier: ~$10-15/month for t3.micro instance

---

## **Next Steps**
1. ✅ Test application in browser
2. ✅ Monitor logs regularly
3. ✅ Setup automated backups
4. ✅ Setup domain + SSL
5. ✅ Monitor EC2 performance

