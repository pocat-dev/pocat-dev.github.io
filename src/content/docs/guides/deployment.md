---
title: Deployment Guide
description: Deploy Pocat to various platforms and environments
---

# Deployment Guide

Deploy Pocat to production environments with these platform-specific guides.

## Docker Deployment

### Basic Docker Setup

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

EXPOSE 3333
CMD ["npm", "start"]
```

### Docker Compose

```yaml
version: '3.8'
services:
  pocat:
    build: .
    ports:
      - "3333:3333"
    environment:
      - NODE_ENV=production
      - DB_CONNECTION=sqlite
    volumes:
      - ./storage:/app/storage
      - ./downloads:/app/downloads
```

## Cloud Platforms

### Railway

1. Connect your GitHub repository
2. Set environment variables:
   ```
   NODE_ENV=production
   PORT=3333
   ```
3. Deploy automatically on push

### Heroku

```bash
# Install Heroku CLI
heroku create your-pocat-app
heroku config:set NODE_ENV=production
git push heroku main
```

### DigitalOcean App Platform

```yaml
name: pocat
services:
- name: api
  source_dir: /
  github:
    repo: your-username/pocat
    branch: main
  run_command: npm start
  environment_slug: node-js
  instance_count: 1
  instance_size_slug: basic-xxs
  envs:
  - key: NODE_ENV
    value: production
```

## VPS Deployment

### Ubuntu/Debian Setup

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Clone and setup
git clone https://github.com/your-username/pocat.git
cd pocat
npm install
npm run build

# Start with PM2
pm2 start npm --name "pocat" -- start
pm2 startup
pm2 save
```

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Environment Variables

### Required Variables
```env
NODE_ENV=production
PORT=3333
DB_CONNECTION=sqlite
```

### Optional Variables
```env
# Database (if using MySQL/PostgreSQL)
DB_HOST=localhost
DB_PORT=3306
DB_USER=pocat
DB_PASSWORD=your_password
DB_DATABASE=pocat

# File Storage
STORAGE_PATH=/app/storage
DOWNLOADS_PATH=/app/downloads

# Rate Limiting
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW=900000
```

## Security Considerations

### API Security
- Use HTTPS in production
- Implement rate limiting
- Add authentication if needed
- Validate all inputs

### File Security
- Restrict file access permissions
- Use separate storage for downloads
- Implement file cleanup policies

### Network Security
```bash
# Firewall setup (Ubuntu)
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## Monitoring

### Health Check Endpoint
```javascript
// Add to your routes
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});
```

### PM2 Monitoring
```bash
# Monitor processes
pm2 monit

# View logs
pm2 logs pocat

# Restart if needed
pm2 restart pocat
```

## Scaling

### Horizontal Scaling
- Use load balancer (nginx, HAProxy)
- Shared storage for downloads
- Database clustering if needed

### Vertical Scaling
- Increase server resources
- Optimize Node.js memory usage
- Use clustering module
