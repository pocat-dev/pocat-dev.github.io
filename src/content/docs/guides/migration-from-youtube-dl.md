---
title: Migration from youtube-dl
description: Step-by-step guide to migrate from youtube-dl to Pocat API
---

# Migration from youtube-dl

This guide helps you migrate from `youtube-dl` command-line tool to Pocat's API-based approach.

## Why Migrate to Pocat?

- **Better reliability**: 95%+ success rate with yt-dlp engine
- **API-first**: RESTful endpoints for easy integration
- **Multiple engines**: Fallback system ensures downloads work
- **Quality options**: Granular control over video quality
- **Scalable**: Handle multiple downloads efficiently

## Migration Steps

### 1. Replace Command Line Calls

**Before (youtube-dl):**
```bash
youtube-dl "https://www.youtube.com/watch?v=VIDEO_ID"
```

**After (Pocat API):**
```bash
curl -X POST http://localhost:3333/v2/projects \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My Video",
    "youtubeUrl": "https://www.youtube.com/watch?v=VIDEO_ID",
    "userId": 1
  }'
```

### 2. Quality Selection Migration

**Before:**
```bash
youtube-dl -f "best[height<=720]" URL
```

**After:**
```bash
curl -X POST http://localhost:3333/v2/projects \
  -H "Content-Type: application/json" \
  -d '{
    "title": "720p Video",
    "youtubeUrl": "URL",
    "userId": 1,
    "quality": "720p"
  }'
```

### 3. Batch Processing

**Before:**
```bash
youtube-dl -a urls.txt
```

**After:**
```javascript
const urls = ['url1', 'url2', 'url3'];
const promises = urls.map(url => 
  fetch('http://localhost:3333/v2/projects', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: `Video from ${url}`,
      youtubeUrl: url,
      userId: 1
    })
  })
);
await Promise.all(promises);
```

## Common Patterns

### Error Handling
```javascript
try {
  const response = await fetch('/v2/projects', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  
  const result = await response.json();
} catch (error) {
  console.error('Download failed:', error);
}
```

### Progress Monitoring
```javascript
// Poll for download status
const checkStatus = async (projectId) => {
  const response = await fetch(`/v2/projects/${projectId}`);
  const project = await response.json();
  return project.status;
};
```

## Benefits After Migration

- **Programmatic control**: Full API access
- **Better error handling**: Detailed error responses
- **Multiple download engines**: Automatic fallback
- **Web integration**: Easy to integrate in web apps
- **Monitoring**: Track download progress and status
