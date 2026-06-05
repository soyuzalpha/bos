```
 ____   ___   _____
|  _ \ / _ \ / ____|
| |_) | | | | (___  
|  _ <| | | |\___ \ 
| |_) | |_| |____) |
|____/ \___/|_____/ 
                    
✨ Backend & Frontend Magic ✨
```

# BOS - Backend & Frontend Project

A full-stack web application built with a modern tech stack featuring a **NestJS backend** and a **Next.js frontend**.

## 📋 Project Overview

**bos** is a TypeScript-based monorepo that combines:
- **Backend**: NestJS REST API server
- **Frontend**: Next.js React application with Tailwind CSS

The repository is primarily written in TypeScript (75.4%), with JavaScript (18.4%) and CSS (6.2%) as supporting languages.

## 🚀 Tech Stack

### Backend
- **Runtime**: Node.js
- **Framework**: [NestJS](https://nestjs.com/) ^11.0.1
- **Language**: TypeScript ^5.7.3
- **Testing**: Jest ^30.0.0
- **Code Quality**: ESLint, Prettier
- **Additional**: RxJS for reactive programming

### Frontend
- **Framework**: [Next.js](https://nextjs.org/) 16.2.7
- **UI Library**: React 19.2.4
- **Styling**: Tailwind CSS ^4
- **Language**: TypeScript ^5
- **Code Quality**: ESLint

## 📁 Project Structure

```
bos/
├── backend/          # 🔧 NestJS application
│   ├── src/
│   ├── test/
│   └── package.json
├── frontend/         # 🎨 Next.js application
│   ├── src/
│   ├── public/
│   └── package.json
└── README.md
```

## 🛠️ Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn

### Installation & Setup

#### Backend
```bash
cd backend
npm install
npm run build
npm run start:dev      # Development mode with watch
npm run start:prod     # Production mode
```

**Available Scripts:**
- `npm run build` - Build the application
- `npm run start` - Start the server
- `npm run start:dev` - Start with watch mode
- `npm run start:debug` - Debug mode
- `npm run start:prod` - Production server
- `npm run lint` - Run ESLint with auto-fix
- `npm run format` - Format code with Prettier
- `npm test` - Run unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:cov` - Generate coverage report
- `npm run test:e2e` - Run end-to-end tests

#### Frontend
```bash
cd frontend
npm install
npm run dev           # Development server (http://localhost:3000)
npm run build         # Production build
npm run start         # Start production server
npm run lint          # Run ESLint
```

**Available Scripts:**
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## 🧪 Testing

### Backend Testing
```bash
cd backend
npm test              # Run all tests
npm run test:watch   # Watch mode
npm run test:cov     # Coverage report
npm run test:e2e     # E2E tests
```

## 📊 Code Quality

Both frontend and backend use ESLint and Prettier for consistent code formatting and linting.

### Format Code
```bash
# Backend
cd backend && npm run format

# Frontend
cd frontend && npm run lint
```

## 🔧 Configuration

### Backend (NestJS)
- Main entry point: `backend/src/main.ts`
- Configuration uses TypeScript path mapping (`tsconfig-paths`)
- Jest configuration for testing included

### Frontend (Next.js)
- Pages router: `frontend/app/`
- Tailwind CSS for styling
- TypeScript strict mode enabled

## 📝 License

UNLICENSED

## 🤝 Contributing

Contributions are welcome! Please ensure code follows the linting and formatting standards defined in the project.

## 📞 Support

For issues or questions, please open an issue in the repository.

---

**Last Updated**: June 2026

```
Made with ❤️ & ☕
```
