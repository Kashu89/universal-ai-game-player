# Deployment Guide - Universal AI Game Player

## Overview

This guide covers deploying the Universal AI Game Player to various environments.

## Prerequisites

- Node.js 18+ or higher
- PostgreSQL 12+ or higher
- npm or yarn package manager
- Git (for version control)

## Environment Setup

### Development

```bash
# 1. Clone repository
git clone <your-repo-url>
cd universal-game-ai

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env

# Edit .env with development settings
DATABASE_URL=postgresql://postgres:password@localhost:5432/universal_game_ai
NODE_ENV=development
```

### Database Setup

```bash
# 1. Create PostgreSQL database
createdb universal_game_ai

# 2. Apply schema
npx drizzle-kit push

# 3. Verify schema
psql universal_game_ai -c "\dt"
```

### Running Locally

```bash
# Development server (with hot reload)
npm run dev
# Open http://localhost:3000

# Production build
npm run build

# Production server
npm start
# Open http://localhost:3000
```

## Docker Deployment

### Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

EXPOSE 3000

ENV NODE_ENV=production

CMD ["npm", "start"]
```

### Docker Compose

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: universal_game_ai
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/universal_game_ai
      NODE_ENV: production
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs

volumes:
  postgres_data:
```

### Running with Docker Compose

```bash
# Start services
docker-compose up -d

# Check logs
docker-compose logs -f app

# Stop services
docker-compose down
```

## Cloud Deployment

### Vercel (Recommended for Next.js)

```bash
# 1. Install Vercel CLI
npm i -g vercel

# 2. Deploy
vercel

# Follow prompts to connect GitHub repo
# Configure environment variables in Vercel dashboard
```

**Vercel Environment Variables**:
```
DATABASE_URL: postgresql://...
NODE_ENV: production
```

### Heroku

```bash
# 1. Install Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# 2. Login
heroku login

# 3. Create app
heroku create universal-game-ai

# 4. Add PostgreSQL
heroku addons:create heroku-postgresql:standard-0

# 5. Set environment variables
heroku config:set NODE_ENV=production

# 6. Deploy
git push heroku main

# 7. View logs
heroku logs --tail
```

### AWS EC2

```bash
# 1. Launch EC2 instance
# - Ubuntu 20.04 LTS
# - t3.small or larger
# - Allow ports 80, 443, 3000

# 2. Connect and setup
ssh -i key.pem ubuntu@<instance-ip>

# 3. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 4. Install PostgreSQL
sudo apt-get install -y postgresql postgresql-contrib

# 5. Clone and setup app
git clone <repo> ~/app
cd ~/app
npm install
cp .env.example .env
# Edit .env with correct DATABASE_URL

# 6. Build
npm run build

# 7. Use PM2 for process management
npm install -g pm2
pm2 start npm --name "game-ai" -- start
pm2 startup
pm2 save

# 8. Setup Nginx reverse proxy
sudo apt-get install -y nginx
# Configure /etc/nginx/sites-available/default
sudo systemctl restart nginx

# 9. Enable SSL with Certbot
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

**Nginx Configuration**:
```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### DigitalOcean

```bash
# 1. Create Droplet (Ubuntu 20.04, $5/month)

# 2. Connect
ssh root@<droplet-ip>

# 3. Initial setup
apt update && apt upgrade -y
apt install -y curl wget git

# 4. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
apt install -y nodejs

# 5. Install PostgreSQL
apt install -y postgresql postgresql-contrib

# 6. Clone repository
git clone <repo> /var/www/game-ai
cd /var/www/game-ai
npm install

# 7. Create .env
cp .env.example .env
# Edit with DigitalOcean database credentials

# 8. Build and start
npm run build
npm start &

# 9. Use systemd for auto-start
cat > /etc/systemd/system/game-ai.service << EOF
[Unit]
Description=Universal AI Game Player
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/var/www/game-ai
ExecStart=/usr/bin/npm start
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable game-ai
systemctl start game-ai
```

## Database Migrations

### Creating a Migration

```bash
# Make schema changes in src/db/schema.ts

# Push changes
npx drizzle-kit push

# Verify migration
psql $DATABASE_URL -c "\dt"
```

### Backup and Restore

```bash
# Backup database
pg_dump $DATABASE_URL > backup.sql

# Restore from backup
psql $DATABASE_URL < backup.sql

# Cloud backup with pg_dump
pg_dump postgresql://user:pass@host:5432/db > backup.sql
tar czf backup.tar.gz backup.sql
aws s3 cp backup.tar.gz s3://my-bucket/backups/
```

## Environment Configuration

### Production Environment Variables

```env
# Database
DATABASE_URL=postgresql://[user]:[password]@[host]:[port]/[database]

# Node environment
NODE_ENV=production

# Application
NEXT_PUBLIC_APP_NAME=Universal AI Game Player
NEXT_PUBLIC_APP_VERSION=0.1.0

# AI Settings
AI_CONFIDENCE_THRESHOLD=0.80
AI_MAX_DECISION_TIME=5000

# Voice Settings
NEXT_PUBLIC_VOICE_ENABLED=true
NEXT_PUBLIC_VOICE_LANGUAGE=en
NEXT_PUBLIC_VOICE_SPEED=1.0

# Logging
LOG_LEVEL=info
NEXT_PUBLIC_DEBUG=false

# Recording
NEXT_PUBLIC_RECORDING_ENABLED=true
NEXT_PUBLIC_MAX_SCREENSHOTS=100

# Features
NEXT_PUBLIC_FEATURE_VOICE_COMMANDS=false
NEXT_PUBLIC_FEATURE_LEARNING_MODE=false
NEXT_PUBLIC_FEATURE_EXTERNAL_GAMES=false
```

## Monitoring and Maintenance

### Health Checks

```bash
# Check API health
curl https://yourdomain.com/api/health

# Expected response:
# {"status": "ok"}
```

### Log Monitoring

```bash
# View application logs
pm2 logs game-ai

# View PostgreSQL logs
tail -f /var/log/postgresql/postgresql.log

# View Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

### Backup Strategy

**Automated Daily Backups**:
```bash
# Add to crontab
0 2 * * * pg_dump $DATABASE_URL | gzip > /backups/db-$(date +\%Y\%m\%d).sql.gz
0 3 * * * aws s3 sync /backups s3://my-bucket/backups/
```

### Performance Monitoring

```bash
# Monitor CPU and memory
htop

# Monitor database connections
psql -d universal_game_ai -c "SELECT count(*) as connections FROM pg_stat_activity;"

# Monitor disk usage
df -h
du -sh /var/lib/postgresql/
```

## Scaling Considerations

### Vertical Scaling

Increase server resources:
- CPU cores
- RAM
- Storage

### Horizontal Scaling

For future use:

1. **Database Read Replicas**
   - Set up PostgreSQL streaming replication
   - Use read replicas for analytics queries

2. **Load Balancing**
   - Deploy multiple app instances
   - Use Nginx/HAProxy for load balancing
   - Share session state in Redis

3. **Caching**
   - Implement Redis caching
   - Cache game profiles
   - Cache session data

### Database Optimization

```sql
-- Create indexes for common queries
CREATE INDEX idx_game_profiles_game_name ON game_profiles(game_name);
CREATE INDEX idx_game_profiles_platform ON game_profiles(platform);
CREATE INDEX idx_game_sessions_profile_id ON game_sessions(profile_id);
CREATE INDEX idx_game_sessions_start_time ON game_sessions(start_time);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM game_profiles WHERE game_name = 'tic-tac-toe';
```

## Disaster Recovery

### Backup Plan

1. **Daily Backups**
   - Automated PostgreSQL dumps
   - Stored in multiple locations
   - Encrypted in transit and at rest

2. **Point-in-Time Recovery**
   - Enable PostgreSQL WAL archiving
   - Backup WAL files to S3
   - Restore to any point in time

3. **Testing Restores**
   - Monthly restore from backup
   - Verify data integrity
   - Document recovery time

### High Availability

```yaml
# Example HA setup with patroni
services:
  postgres-1:
    image: patroni:latest
    environment:
      PATRONI_POSTGRES_ADMIN_USERNAME: admin
      PATRONI_POSTGRES_ADMIN_PASSWORD: secret
      PATRONI_POSTGRESQL_PG_CTL: /usr/lib/postgresql/13/bin/pg_ctl

  postgres-2:
    image: patroni:latest
    depends_on:
      - postgres-1

  postgres-3:
    image: patroni:latest
    depends_on:
      - postgres-1

  etcd:
    image: quay.io/coreos/etcd:latest
```

## Security Best Practices

### SSL/TLS

```bash
# Use Let's Encrypt
certbot certonly --standalone -d yourdomain.com
# Auto-renew
certbot renew --quiet --no-self-upgrade
```

### Database Security

```bash
# Change default password
psql -U postgres -c "ALTER USER postgres WITH PASSWORD 'new_strong_password';"

# Create limited-privilege user
psql -U postgres -c "CREATE USER app_user WITH PASSWORD 'app_password';"
psql -U postgres -c "GRANT CONNECT ON DATABASE universal_game_ai TO app_user;"
```

### Environment Variables

- Never commit .env files
- Use secure vaults (AWS Secrets Manager, HashiCorp Vault)
- Rotate secrets regularly
- Use different secrets per environment

### Network Security

- Enable firewall (ufw)
- Whitelist necessary ports
- Use VPC/private networks
- Enable DDoS protection

## Troubleshooting

### Common Issues

**Database Connection Error**
```bash
# Check database is running
psql -U postgres -l

# Check connection string
echo $DATABASE_URL

# Test connection
psql $DATABASE_URL -c "SELECT 1"
```

**Out of Memory**
```bash
# Increase Node.js heap size
NODE_OPTIONS=--max-old-space-size=4096 npm start

# Analyze memory usage
node --trace-gc app.js
```

**High Database Load**
```bash
# Check active connections
psql -d universal_game_ai -c "SELECT pid, usename, query FROM pg_stat_activity WHERE state = 'active';"

# Kill long-running queries
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE duration > interval '1 hour';
```

## Performance Optimization

### Frontend

- Enable compression (gzip/brotli)
- Minify JavaScript/CSS
- Optimize images
- Lazy load components

### Backend

- Database query optimization
- Connection pooling
- Caching strategies
- Rate limiting

### Network

```nginx
# Enable compression
gzip on;
gzip_types text/plain text/css application/json application/javascript;

# Enable caching
add_header Cache-Control "public, max-age=3600";

# Enable HTTP/2
listen 443 ssl http2;
```

## Checklist

- [ ] Database configured and tested
- [ ] Environment variables set
- [ ] SSL certificate installed
- [ ] Build passes all tests
- [ ] Backups configured
- [ ] Monitoring set up
- [ ] Firewall rules applied
- [ ] Domain name configured
- [ ] Load testing completed
- [ ] Team trained on deployment
- [ ] Documentation updated
- [ ] Rollback plan documented

---

**Maintenance Schedule**:
- Daily: Monitor logs and performance
- Weekly: Database optimization check
- Monthly: Backup restore test
- Quarterly: Security audit
- Annually: Major upgrade evaluation

**Last Updated**: October 2026
