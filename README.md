# 🎬 Pocat - AI Video Clipper Platform

> Transform long videos into engaging clips with AI-powered analysis

Pocat adalah platform video clipper AI yang memungkinkan users untuk mengubah video panjang menjadi clips pendek yang engaging, mirip seperti OpusClip atau Vizard AI.

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │     Backend      │    │   Database      │
│   (React/TS)    │◄──►│   (AdonisJS)     │◄──►│   (Turso)       │
│                 │    │                  │    │                 │
│ • Video Editor  │    │ • YouTube API    │    │ • Projects      │
│ • AI Analysis   │    │ • FFmpeg         │    │ • Clips         │
│ • Timeline      │    │ • File Storage   │    │ • Users         │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 📦 Components

### 🎨 Frontend
- **Repository**: [pocat-frontend](./frontend) (Submodule)
- **Tech Stack**: React, TypeScript, Tailwind CSS
- **Features**: Video editor, AI analysis, timeline, export

### ⚙️ Backend  
- **Repository**: [pocat-backend](./backend) (Submodule)
- **Tech Stack**: AdonisJS, TypeScript, FFmpeg, Turso
- **Features**: YouTube processing, clip generation, file serving

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- pnpm
- FFmpeg
- Git

### Setup
```bash
# Clone main repository with submodules
git clone --recursive git@github.com:konxc/pocat.git
cd pocat

# Setup backend
cd backend
pnpm install
cp .env.example .env
# Edit .env with your Turso credentials
pnpm run ace migrate:turso

# Setup frontend  
cd ../frontend
pnpm install

# Start development
# Terminal 1: Backend
cd backend && pnpm run dev

# Terminal 2: Frontend
cd frontend && pnpm run dev
```

## 🎯 Features

### ✅ MVP Features (Current)
- **YouTube Import** - Extract video info and thumbnails
- **Demo Clip Generation** - Create sample clips with custom timing
- **Multiple Aspect Ratios** - 9:16, 16:9, 1:1 support
- **Real-time Status** - Progress tracking and download links
- **Cross-platform** - Works with VLC, browsers, mobile

### 🔄 Enhanced Features (V2)
- **Full Video Download** - Cache complete videos locally
- **Live Streaming** - Stream downloaded videos for editing
- **AI Batch Processing** - Process multiple clips from AI analysis
- **Quality Selection** - Choose video quality for downloads
- **Parallel Processing** - Multiple clips processed simultaneously

### 🎨 AI Features (Frontend)
- **Auto-detect Viral Clips** - AI analysis for engagement potential
- **Smart Timestamps** - Automatic highlight detection
- **Sentiment Analysis** - Content analysis for optimal clips
- **GPU Acceleration** - Fast frame processing

## 🌐 Live Demo

- **Backend API**: https://nonimitating-corie-extemporary.ngrok-free.dev/
- **Frontend**: [Coming Soon]

## 📚 Documentation

- [Backend API Documentation](./backend/README.md)
- [Frontend Integration Guide](./backend/FRONTEND_INTEGRATION.md)
- [MVP Status & Testing](./backend/MVP_STATUS.md)
- [Community Guidelines](./COMMUNITY.md)

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Developer

**[@sandikodev](https://twitter.com/sandikodev)** - Full-stack developer passionate about AI and video technology

- 🐦 Twitter: [@sandikodev](https://twitter.com/sandikodev)
- 🎵 TikTok: [@sandikodev](https://tiktok.com/@sandikodev)
- 🐙 GitHub: [@sandikodev](https://github.com/sandikodev)
- 💼 LinkedIn: [@sandikodev](https://linkedin.com/in/sandikodev)

## 🙏 Acknowledgments

- **AdonisJS** - Robust Node.js framework
- **Turso** - Edge SQLite database
- **FFmpeg** - Video processing engine
- **YouTube** - Video content source

---

**Built with ❤️ for the creator economy by [@sandikodev](https://twitter.com/sandikodev)**
