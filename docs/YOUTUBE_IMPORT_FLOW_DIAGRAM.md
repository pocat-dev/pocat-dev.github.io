# 🎬 YouTube Import Flow Diagram - Pocat

## 📊 Complete System Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend (React)
    participant API as Backend API
    participant DB as Database
    participant VP as Video Processor
    participant FS as File System

    Note over U,FS: YouTube Import Process Flow

    %% Step 1: User Input
    U->>F: 1. Paste YouTube URL
    U->>F: 2. Click "Import" button
    
    %% Step 2: Frontend Processing
    F->>F: 3. Validate URL
    F->>F: 4. Set loading state
    Note over F: setIsImporting(true)<br/>setImportStatus("Initializing...")
    
    %% Step 3: API Call
    F->>API: 5. POST /v2/projects
    Note over F,API: {<br/>  title: "Project...",<br/>  youtubeUrl: "https://...",<br/>  userId: 1,<br/>  quality: "720p",<br/>  downloader: "auto"<br/>}
    
    %% Step 4: Backend Processing
    API->>VP: 6. getVideoInfo(youtubeUrl)
    VP-->>API: 7. Video metadata
    Note over VP,API: {<br/>  title, duration,<br/>  thumbnail, format<br/>}
    
    API->>DB: 8. Create VideoProject
    DB-->>API: 9. Project created (ID: 123)
    
    %% Step 5: Background Download
    API->>VP: 10. downloadVideoAsync()
    Note over API,VP: Background process starts
    API-->>F: 11. Immediate response
    Note over API,F: {<br/>  success: true,<br/>  projectId: 123,<br/>  status: "downloading"<br/>}
    
    %% Step 6: Frontend State Update
    F->>F: 12. Update video state
    Note over F: setVideoState({<br/>  sourceType: 'youtube',<br/>  thumbnail: videoInfo.thumbnail,<br/>  projectId: 123<br/>})
    
    F->>U: 13. Show thumbnail + loading
    
    %% Step 7: Polling Loop
    loop Every 2 seconds
        F->>API: 14. GET /v2/projects/123/download-status
        API->>DB: 15. Check project status
        DB-->>API: 16. Current status
        API-->>F: 17. Status response
        Note over API,F: {<br/>  readyForEditing: false,<br/>  status: "downloading",<br/>  progress: 45<br/>}
        F->>U: 18. Update progress
    end
    
    %% Step 8: Download Process (Background)
    par Background Download
        VP->>VP: 19. Try yt-dlp download
        alt yt-dlp success
            VP->>FS: 20a. Save video file
            FS-->>VP: 21a. File saved
        else yt-dlp fails
            VP->>VP: 20b. Try ytdl-core
            alt ytdl-core success
                VP->>FS: 21b. Save video file
            else ytdl-core fails
                VP->>VP: 22b. Try puppeteer
                VP->>FS: 23b. Save video file
            end
        end
        
        VP->>DB: 24. Update project status
        Note over VP,DB: status = 'completed'<br/>videoFilePath = '/path/to/video.mp4'
    end
    
    %% Step 9: Download Complete
    F->>API: 25. GET /v2/projects/123/download-status
    API->>DB: 26. Check status
    DB-->>API: 27. readyForEditing: true
    API-->>F: 28. Download complete!
    Note over API,F: {<br/>  readyForEditing: true,<br/>  status: "completed",<br/>  video: { source: "fresh" }<br/>}
    
    %% Step 10: Switch to Video Stream
    F->>F: 29. Update video state
    Note over F: setVideoState({<br/>  sourceType: 'backend-stream',<br/>  url: '/v2/projects/123/stream',<br/>  isPlaying: true<br/>})
    
    F->>U: 30. Show success + video ready
    U->>F: 31. Start editing video
```

## 🏗️ Architecture Components

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[User Interface]
        VS[Video State]
        API_CLIENT[API Client]
    end
    
    subgraph "Backend Layer"
        ROUTER[Routes Handler]
        CONTROLLER[Enhanced Projects Controller]
        VP[Video Processor]
    end
    
    subgraph "Data Layer"
        DB[(SQLite Database)]
        FS[(File System)]
        CACHE[(Reference Cache)]
    end
    
    subgraph "External Services"
        YT[YouTube API]
        YTDLP[yt-dlp]
        YTDL[ytdl-core]
        PUPPET[Puppeteer]
    end
    
    %% Frontend Flow
    UI --> VS
    VS --> API_CLIENT
    API_CLIENT --> ROUTER
    
    %% Backend Flow
    ROUTER --> CONTROLLER
    CONTROLLER --> VP
    CONTROLLER --> DB
    
    %% Video Processing
    VP --> YT
    VP --> YTDLP
    VP --> YTDL
    VP --> PUPPET
    VP --> FS
    VP --> CACHE
    
    %% Data Flow
    DB --> CACHE
    FS --> CACHE
```

## 🔄 State Transitions

```mermaid
stateDiagram-v2
    [*] --> Idle
    
    Idle --> Importing : User clicks Import
    Importing --> Validating : URL validation
    Validating --> Creating : API call
    Creating --> Downloading : Project created
    Downloading --> Polling : Background download starts
    
    state Downloading {
        [*] --> YtDlp
        YtDlp --> YtdlCore : Fallback
        YtdlCore --> Puppeteer : Fallback
        Puppeteer --> Failed : All methods fail
        YtDlp --> Success : Download complete
        YtdlCore --> Success : Download complete
        Puppeteer --> Success : Download complete
    }
    
    Polling --> Downloading : Status check
    Polling --> Ready : Download complete
    Ready --> Streaming : Video available
    Streaming --> Editing : User starts editing
    
    Failed --> Idle : Reset/Retry
    Editing --> Idle : New import
```

## 📱 Frontend Component Flow

```mermaid
graph LR
    subgraph "EditorView Component"
        INPUT[YouTube URL Input]
        BUTTON[Import Button]
        PLAYER[Video Player]
    end
    
    subgraph "App Component State"
        YT_LINK[youtubeLink]
        IS_IMPORTING[isImporting]
        IMPORT_STATUS[importStatus]
        VIDEO_STATE[videoState]
    end
    
    subgraph "API Services"
        CREATE_PROJECT[createProject()]
        GET_STATUS[getProjectDownloadStatus()]
    end
    
    INPUT --> YT_LINK
    BUTTON --> IS_IMPORTING
    IS_IMPORTING --> CREATE_PROJECT
    CREATE_PROJECT --> IMPORT_STATUS
    IMPORT_STATUS --> GET_STATUS
    GET_STATUS --> VIDEO_STATE
    VIDEO_STATE --> PLAYER
```

## 🗄️ Database Schema Flow

```mermaid
erDiagram
    VIDEO_PROJECTS {
        int id PK
        int userId FK
        string title
        string youtubeUrl
        text videoMetadata
        int duration
        string thumbnailUrl
        string status
        string videoFilePath
        datetime createdAt
        datetime updatedAt
    }
    
    VIDEO_REFERENCES {
        int id PK
        string youtubeUrl
        string filePath
        int referenceCount
        datetime createdAt
    }
    
    CLIPS {
        int id PK
        int videoProjectId FK
        string title
        float startTime
        float endTime
        string outputUrl
        string status
        datetime createdAt
    }
    
    VIDEO_PROJECTS ||--o{ CLIPS : "has many"
    VIDEO_PROJECTS }o--|| VIDEO_REFERENCES : "references"
```

## 🚀 Performance Optimization Flow

```mermaid
graph TD
    START[YouTube URL Input] --> CHECK_CACHE{Check Reference Cache}
    
    CHECK_CACHE -->|Cache Hit| INSTANT[Instant Access]
    CHECK_CACHE -->|Cache Miss| DOWNLOAD[Start Download]
    
    INSTANT --> READY[Video Ready]
    
    DOWNLOAD --> YTDLP{Try yt-dlp}
    YTDLP -->|Success| SAVE[Save to Storage]
    YTDLP -->|Fail| YTDL{Try ytdl-core}
    
    YTDL -->|Success| SAVE
    YTDL -->|Fail| PUPPET{Try Puppeteer}
    
    PUPPET -->|Success| SAVE
    PUPPET -->|Fail| ERROR[Download Failed]
    
    SAVE --> CREATE_REF[Create Reference]
    CREATE_REF --> UPDATE_DB[Update Database]
    UPDATE_DB --> READY
    
    READY --> STREAM[Stream to Frontend]
    STREAM --> EDIT[User Can Edit]
```

## 📊 API Response Flow

```mermaid
graph LR
    subgraph "POST /v2/projects Response"
        CREATE_RESP["{<br/>success: true,<br/>projectId: 123,<br/>status: 'downloading',<br/>videoInfo: {...}<br/>}"]
    end
    
    subgraph "GET /v2/projects/123/download-status"
        STATUS_RESP["{<br/>readyForEditing: false,<br/>status: 'downloading',<br/>progress: 45,<br/>video: null<br/>}"]
        
        COMPLETE_RESP["{<br/>readyForEditing: true,<br/>status: 'completed',<br/>progress: 100,<br/>video: {source: 'fresh'}<br/>}"]
    end
    
    CREATE_RESP --> STATUS_RESP
    STATUS_RESP --> STATUS_RESP
    STATUS_RESP --> COMPLETE_RESP
```

## 🎯 Error Handling Flow

```mermaid
graph TD
    ERROR_START[Error Occurs] --> ERROR_TYPE{Error Type}
    
    ERROR_TYPE -->|Invalid URL| URL_ERROR[Show URL validation error]
    ERROR_TYPE -->|Network Error| NETWORK_ERROR[Show connection error]
    ERROR_TYPE -->|Download Failed| DOWNLOAD_ERROR[Try alternative downloader]
    ERROR_TYPE -->|Server Error| SERVER_ERROR[Show server error]
    
    URL_ERROR --> RESET[Reset form]
    NETWORK_ERROR --> RETRY[Offer retry]
    DOWNLOAD_ERROR --> FALLBACK[Use fallback method]
    SERVER_ERROR --> SUPPORT[Show support contact]
    
    RETRY --> ERROR_START
    FALLBACK --> ERROR_START
    RESET --> IDLE[Return to idle state]
    SUPPORT --> IDLE
```

---

**Diagram Version**: 1.0  
**Created**: December 18, 2025  
**Components**: Frontend (React) + Backend (AdonisJS) + Video Processing  
**Flow Type**: YouTube Import & Download Process
