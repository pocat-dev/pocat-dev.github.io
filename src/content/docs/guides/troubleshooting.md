---
title: Troubleshooting
description: Common issues and solutions for Pocat API
---

# Troubleshooting

Common issues and their solutions when using Pocat.

## Download Issues

### Video Download Fails

**Symptoms:**
- API returns error status
- Download never completes
- File not created

**Solutions:**

1. **Check URL validity:**
   ```bash
   curl -X POST /v2/projects \
     -d '{"youtubeUrl": "VALID_YOUTUBE_URL"}'
   ```

2. **Try different downloader:**
   ```json
   {
     "youtubeUrl": "URL",
     "downloader": "yt-dlp"
   }
   ```

3. **Check logs:**
   ```bash
   tail -f storage/logs/app.log
   ```

### 403 Forbidden Errors

**Cause:** YouTube blocking requests

**Solutions:**
- Use yt-dlp downloader (more reliable)
- Add delays between requests
- Rotate IP addresses if possible

```json
{
  "youtubeUrl": "URL",
  "downloader": "yt-dlp"
}
```

### Quality Not Available

**Symptoms:**
- Requested quality returns lower resolution
- Error about format not found

**Solution:**
```bash
# Check available formats first
yt-dlp -F "YOUTUBE_URL"

# Use auto quality selection
curl -X POST /v2/projects \
  -d '{"youtubeUrl": "URL", "quality": "auto"}'
```

## API Issues

### Connection Refused

**Symptoms:**
```
Error: connect ECONNREFUSED 127.0.0.1:3333
```

**Solutions:**

1. **Check if server is running:**
   ```bash
   ps aux | grep node
   netstat -tlnp | grep 3333
   ```

2. **Start the server:**
   ```bash
   cd pocat
   npm start
   ```

3. **Check port availability:**
   ```bash
   lsof -i :3333
   ```

### CORS Errors

**Symptoms:**
```
Access to fetch blocked by CORS policy
```

**Solution:**
Add CORS headers in your frontend:
```javascript
fetch('http://localhost:3333/v2/projects', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'ngrok-skip-browser-warning': 'true' // if using ngrok
  },
  body: JSON.stringify(data)
});
```

### Rate Limiting

**Symptoms:**
```
HTTP 429 Too Many Requests
```

**Solutions:**
- Reduce request frequency
- Implement exponential backoff
- Contact admin for rate limit increase

## Performance Issues

### Slow Downloads

**Causes & Solutions:**

1. **Network bandwidth:**
   - Check internet connection
   - Use lower quality settings

2. **Server resources:**
   ```bash
   # Check CPU/Memory usage
   top
   htop
   
   # Check disk space
   df -h
   ```

3. **Concurrent downloads:**
   - Limit simultaneous downloads
   - Queue downloads instead

### High Memory Usage

**Solutions:**

1. **Restart the service:**
   ```bash
   pm2 restart pocat
   ```

2. **Increase memory limit:**
   ```bash
   node --max-old-space-size=4096 server.js
   ```

3. **Clean up old files:**
   ```bash
   find downloads/ -mtime +7 -delete
   ```

## Installation Issues

### Dependencies Not Installing

**Error:**
```
npm ERR! peer dep missing
```

**Solutions:**
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Use specific Node.js version
nvm use 18
npm install
```

### Python/yt-dlp Issues

**Error:**
```
yt-dlp: command not found
```

**Solutions:**

1. **Install yt-dlp:**
   ```bash
   # Using pip
   pip install yt-dlp
   
   # Using package manager
   sudo apt install yt-dlp  # Ubuntu/Debian
   brew install yt-dlp      # macOS
   ```

2. **Check PATH:**
   ```bash
   which yt-dlp
   echo $PATH
   ```

## Database Issues

### SQLite Locked

**Error:**
```
SQLITE_BUSY: database is locked
```

**Solutions:**
```bash
# Check for zombie processes
ps aux | grep node

# Kill hanging processes
pkill -f node

# Restart the application
npm start
```

### Migration Errors

**Solutions:**
```bash
# Reset database (development only)
rm -f database/database.sqlite
npm run migration:run

# Or run migrations manually
npm run migration:fresh
```

## File System Issues

### Permission Denied

**Error:**
```
EACCES: permission denied, open '/path/to/file'
```

**Solutions:**
```bash
# Fix permissions
chmod 755 storage/
chmod 755 downloads/
chown -R $USER:$USER storage/ downloads/

# Or run with sudo (not recommended for production)
sudo npm start
```

### Disk Space Full

**Error:**
```
ENOSPC: no space left on device
```

**Solutions:**
```bash
# Check disk usage
df -h

# Clean up old downloads
find downloads/ -mtime +7 -delete

# Clean up logs
truncate -s 0 storage/logs/*.log
```

## Getting Help

### Enable Debug Mode
```bash
DEBUG=* npm start
```

### Collect System Information
```bash
# System info
uname -a
node --version
npm --version
yt-dlp --version

# Process info
ps aux | grep node
netstat -tlnp | grep 3333
```

### Log Analysis
```bash
# View recent errors
tail -100 storage/logs/app.log | grep ERROR

# Monitor real-time logs
tail -f storage/logs/app.log
```

If issues persist, please create an issue on GitHub with:
- Error messages
- System information
- Steps to reproduce
- Log files (remove sensitive data)
