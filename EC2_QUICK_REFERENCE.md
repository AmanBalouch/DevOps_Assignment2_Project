# EC2 Quick Reference Card

## **SSH Connection**
```bash
ssh -i rentease-key.pem ubuntu@YOUR_EC2_IP
```

## **Start Services**
```bash
cd ~/rentease
docker-compose up -d
```

## **Check Status**
```bash
docker-compose ps
```

## **View Logs**
```bash
docker-compose logs -f              # All services
docker-compose logs -f backend      # Backend only
docker-compose logs -f postgres     # Database only
```

## **Stop Services**
```bash
docker-compose down
```

## **Restart Services**
```bash
docker-compose restart
```

## **Update Images**
```bash
docker-compose pull
docker-compose up -d
```

## **Database Access**
```bash
docker-compose exec postgres psql -U postgres -d rentease
\dt                    # List tables
\q                     # Exit
```

## **View Volumes**
```bash
docker volume ls
docker inspect rentease_postgres_data
```

## **Check EC2 Health**
```bash
df -h                  # Disk space
free -h                # Memory
top                    # CPU usage
```

## **Access Application**
```
Frontend:  http://YOUR_EC2_IP
API Docs:  http://YOUR_EC2_IP:8000/docs
Backend:   http://YOUR_EC2_IP:8000
```

## **Backup Database**
```bash
docker-compose exec postgres pg_dump -U postgres -d rentease > backup.sql
```

## **Restore Database**
```bash
docker-compose exec -T postgres psql -U postgres -d rentease < backup.sql
```

## **Common Issues**

**Port 80 already in use:**
```bash
docker-compose down
docker-compose up -d
```

**Backend not responding:**
```bash
docker-compose logs backend
docker-compose restart backend
```

**Database connection failed:**
```bash
docker-compose logs postgres
docker-compose restart postgres
```

---

**Saved in:** `/Users/durmuhammadkhan/Desktop/rent-ease-main/EC2_DEPLOYMENT_GUIDE.md`
