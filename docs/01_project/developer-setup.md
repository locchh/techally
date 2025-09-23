---
id: DEV-SETUP-001
title: Developer Setup Guide
category: project
tags: [development, setup, environment, tools]
version: 1.0.0
status: approved
created: 2025-09-23
updated: 2025-09-23
author: Developer Experience Team
references:
  - id: CONTRIB-001
    type: guide
    link: ./contributing.md
  - id: ARCH-BE-001
    type: architecture
    link: ../03_architecture/backend-architecture.md
  - id: ARCH-FE-001
    type: architecture
    link: ../03_architecture/frontend-architecture.md
---

# Developer Setup Guide

## 1. Prerequisites

### 1.1 System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **OS** | macOS 11+, Ubuntu 20.04+, Windows 10+ | macOS 13+, Ubuntu 22.04+ |
| **RAM** | 8 GB | 16 GB |
| **Storage** | 20 GB free | 50 GB free |
| **CPU** | 4 cores | 8 cores |
| **Network** | Broadband | Fiber/High-speed |

### 1.2 Required Software

```bash
# Check versions
node --version      # v18.0.0 or higher
npm --version       # v9.0.0 or higher
docker --version    # v24.0.0 or higher
git --version       # v2.30.0 or higher
```

### 1.3 Development Tools

- **IDE**: VS Code (recommended) or WebStorm
- **API Client**: Postman or Insomnia
- **Database Client**: DBeaver or pgAdmin
- **Container Management**: Docker Desktop or Rancher Desktop

## 2. Environment Setup

### 2.1 Clone Repository

```bash
# Clone the repository
git clone https://github.com/techally/techally-platform.git
cd techally-platform

# Add upstream remote
git remote add upstream https://github.com/techally/techally-platform.git

# Verify remotes
git remote -v
```

### 2.2 Install Dependencies

```bash
# Install Node.js dependencies
npm install

# Install global tools
npm install -g @nestjs/cli
npm install -g typescript
npm install -g pm2
npm install -g nodemon

# Verify installations
nest --version
tsc --version
pm2 --version
```

### 2.3 Environment Variables

```bash
# Copy environment template
cp .env.example .env.local

# Edit environment variables
nano .env.local
```

```env
# .env.local
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/techally_dev
DATABASE_POOL_SIZE=10

# Redis
REDIS_URL=redis://localhost:6379
REDIS_PREFIX=techally:dev:

# Authentication
JWT_SECRET=your-secret-key-here
JWT_EXPIRES_IN=1h
REFRESH_TOKEN_EXPIRES_IN=7d

# Services
AUTH_SERVICE_URL=http://localhost:3001
USER_SERVICE_URL=http://localhost:3002
PRODUCT_SERVICE_URL=http://localhost:3003
ORDER_SERVICE_URL=http://localhost:3004

# External APIs
STRIPE_API_KEY=sk_test_...
SENDGRID_API_KEY=SG...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1

# Monitoring
DATADOG_API_KEY=...
SENTRY_DSN=...
```

## 3. Docker Setup

### 3.1 Docker Compose Configuration

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: techally_dev
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: localpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  elasticsearch:
    image: elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  maildev:
    image: maildev/maildev
    ports:
      - "1080:1080"
      - "1025:1025"

volumes:
  postgres_data:
  redis_data:
  es_data:
  rabbitmq_data:
```

### 3.2 Start Docker Services

```bash
# Start all services
docker-compose -f docker-compose.dev.yml up -d

# Check status
docker-compose -f docker-compose.dev.yml ps

# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Stop services
docker-compose -f docker-compose.dev.yml down
```

## 4. Database Setup

### 4.1 Run Migrations

```bash
# Run database migrations
npm run db:migrate

# Seed development data
npm run db:seed

# Verify database
npm run db:status
```

### 4.2 Database Management

```bash
# Connect to database
psql $DATABASE_URL

# Common commands
\dt                 # List tables
\d+ table_name      # Describe table
\q                  # Quit

# Backup database
pg_dump $DATABASE_URL > backup.sql

# Restore database
psql $DATABASE_URL < backup.sql
```

## 5. Application Setup

### 5.1 Backend Services

```bash
# Start all microservices
npm run dev:services

# Or start individually
npm run dev:auth      # Port 3001
npm run dev:user      # Port 3002
npm run dev:product   # Port 3003
npm run dev:order     # Port 3004
npm run dev:payment   # Port 3005

# Start with debugging
npm run debug:auth
```

### 5.2 Frontend Application

```bash
# Navigate to frontend
cd packages/web

# Install dependencies
npm install

# Start development server
npm run dev           # Port 3000

# Build for production
npm run build

# Run tests
npm run test
npm run test:coverage
```

### 5.3 Mobile Application

```bash
# Navigate to mobile app
cd packages/mobile

# Install dependencies
npm install
cd ios && pod install && cd ..

# Start Metro bundler
npm start

# Run on iOS
npm run ios

# Run on Android
npm run android
```

## 6. Development Workflow

### 6.1 Git Workflow

```bash
# Create feature branch
git checkout -b feature/add-new-feature

# Make changes and commit
git add .
git commit -m "feat: add new feature"

# Push to remote
git push origin feature/add-new-feature

# Create pull request
# Use GitHub/GitLab UI
```

### 6.2 Code Quality

```bash
# Run linter
npm run lint
npm run lint:fix

# Run formatter
npm run format
npm run format:check

# Type checking
npm run type-check

# Run all checks
npm run quality
```

### 6.3 Testing

```bash
# Unit tests
npm run test:unit

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# Test coverage
npm run test:coverage

# Watch mode
npm run test:watch
```

## 7. Development Tools

### 7.1 VS Code Extensions

```json
// .vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "prisma.prisma",
    "graphql.vscode-graphql",
    "eamodio.gitlens",
    "usernamehw.errorlens",
    "christian-kohler.path-intellisense",
    "mikestead.dotenv",
    "ms-azuretools.vscode-docker",
    "redhat.vscode-yaml",
    "streetsidesoftware.code-spell-checker"
  ]
}
```

### 7.2 VS Code Settings

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.updateImportsOnFileMove.enabled": "always",
  "files.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.next": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/coverage": true
  }
}
```

### 7.3 Debug Configuration

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Auth Service",
      "program": "${workspaceFolder}/services/auth/src/main.ts",
      "preLaunchTask": "npm: build:auth",
      "outFiles": ["${workspaceFolder}/services/auth/dist/**/*.js"],
      "env": {
        "NODE_ENV": "development",
        "DEBUG": "app:*"
      }
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Frontend",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/packages/web",
      "sourceMaps": true
    }
  ]
}
```

## 8. Common Issues

### 8.1 Troubleshooting

| Issue | Solution |
|-------|----------|
| **Port already in use** | `lsof -i :3000` then `kill -9 <PID>` |
| **Docker permission denied** | Add user to docker group: `sudo usermod -aG docker $USER` |
| **Node modules issues** | `rm -rf node_modules package-lock.json && npm install` |
| **Database connection failed** | Check Docker containers: `docker ps` |
| **TypeScript errors** | `npx tsc --noEmit` to check types |
| **Git merge conflicts** | Use VS Code merge editor or `git mergetool` |

### 8.2 Performance Tips

```bash
# Increase Node.js memory
export NODE_OPTIONS="--max-old-space-size=4096"

# Use npm ci for faster installs
npm ci

# Enable parallel builds
npm config set jobs 8

# Cache Docker layers
DOCKER_BUILDKIT=1 docker build .
```

## 9. Useful Scripts

### 9.1 Development Scripts

```json
// package.json scripts
{
  "scripts": {
    "dev": "concurrently \"npm:dev:*\"",
    "dev:services": "pm2 start ecosystem.config.js --env development",
    "dev:web": "cd packages/web && npm run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "clean": "turbo run clean && rm -rf node_modules",
    "reset": "npm run clean && npm install && npm run db:reset",
    "update-deps": "npx npm-check-updates -u && npm install"
  }
}
```

### 9.2 Utility Scripts

```bash
#!/bin/bash
# scripts/dev-setup.sh

echo "🚀 Setting up TechAlly development environment..."

# Check prerequisites
command -v node >/dev/null 2>&1 || { echo "Node.js is required"; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "Docker is required"; exit 1; }

# Install dependencies
echo "📦 Installing dependencies..."
npm ci

# Setup environment
echo "⚙️  Setting up environment..."
cp .env.example .env.local

# Start Docker services
echo "🐳 Starting Docker services..."
docker-compose -f docker-compose.dev.yml up -d

# Wait for services
echo "⏳ Waiting for services..."
sleep 10

# Run migrations
echo "🗄️  Running database migrations..."
npm run db:migrate
npm run db:seed

echo "✅ Setup complete! Run 'npm run dev' to start developing."
```

## 10. Resources

### 10.1 Documentation

- [API Documentation](http://localhost:3000/api-docs)
- [Storybook](http://localhost:6006)
- [Database Schema](../05_database/schema-overview.md)
- [Architecture Overview](../03_architecture/system-overview.md)

### 10.2 Development URLs

| Service | URL | Description |
|---------|-----|-------------|
| **Web App** | http://localhost:3000 | Main application |
| **API Gateway** | http://localhost:3001 | API endpoint |
| **GraphQL Playground** | http://localhost:3001/graphql | GraphQL IDE |
| **RabbitMQ** | http://localhost:15672 | Message queue UI |
| **MailDev** | http://localhost:1080 | Email testing |
| **Elasticsearch** | http://localhost:9200 | Search engine |

## 11. References

- [Contributing Guide](./contributing.md) - `CONTRIB-001`
- [Backend Architecture](../03_architecture/backend-architecture.md) - `ARCH-BE-001`
- [Frontend Architecture](../03_architecture/frontend-architecture.md) - `ARCH-FE-001`
- [Team Onboarding](../08_team/onboarding.md) - `ONBOARD-001`

---
*This developer setup guide is maintained by the Developer Experience Team.*
