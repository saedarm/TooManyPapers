TooManyPapers - AI-Powered Research Paper & News Aggregator

A production-ready Go application that automatically collects, summarizes, and delivers AI/ML research papers and industry news using MongoDB Atlas, and automated email digests.

## 🚀 Features

- **📚 Multi-Source Collection**
  - arXiv research papers (cs.AI, cs.LG, cs.CL, cs.CV)
  - RSS feeds from OpenAI, Anthropic, Google AI, etc.
  - Web scraping capabilities
  - Automatic deduplication

- **🤖 AI-Powered Intelligence**
  - OpenAI GPT-4 summaries
  - Key takeaways extraction
  - Relevance scoring
  - Smart categorization

- **📧 Automated Delivery**
  - Daily digests (7:30 AM)
  - Weekly summaries (Mondays 9:00 AM)
  - Beautiful HTML emails
  - Personalized preferences

- **💾 MongoDB Atlas Integration**
  - Cloud-hosted database
  - Automatic scaling
  - TTL indexes (90-day retention)
  - Full-text search
  - Analytics tracking

- **📊 Analytics & Insights**
  - Click/view tracking
  - Popular articles
  - Source performance
  - Category distribution

## 📋 Prerequisites

- **Go 1.21+** - [Download Go](https://golang.org/dl/)
- **MongoDB Atlas Account** - [Sign up free](https://www.mongodb.com/cloud/atlas/register)
- **OpenAI API Key** (optional) - [Get API key](https://platform.openai.com/api-keys)
- **Gmail with App Password** - [Create app password](https://myaccount.google.com/apppasswords)

## 🛠️ Complete Setup Guide

### Step 1: Create Your Own Repo

```bash
# Create project directory
mkdir repo
cd repo

# Initialize Go module
go mod init repo
```

### Step 2: Create Project Structure

```bash
# Create directory structure
mkdir -p cmd/api
mkdir -p internal/{config,database,models,services,handlers,scheduler,server}
mkdir -p scripts
mkdir -p web/templates
```

### Step 3: Create Essential Files

#### 3.1 Create `go.mod`
```go
module repo

go 1.21

require (
    github.com/PuerkitoBio/goquery v1.8.1
    github.com/gin-gonic/gin v1.9.1
    github.com/joho/godotenv v1.5.1
    github.com/mmcdole/gofeed v1.2.1
    github.com/robfig/cron/v3 v3.0.1
    go.mongodb.org/mongo-driver/v2 v2.0.0
    github.com/google/uuid v1.5.0
)
```

#### 3.2 Create `.env` file
```env
# MongoDB Atlas Connection (REQUIRED)
MONGODB_PASSWORD=your_mongodb_password_here
DATABASE_NAME=dbname

# Server Configuration
PORT=8080
ENV=production

# Email Configuration (REQUIRED for digests)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-gmail-app-password
EMAIL_FROM=REPO <your-email@gmail.com>
EMAIL_RECIPIENTS=recipient1@example.com,recipient2@example.com

# OpenAI Configuration (Optional - for AI summaries)
OPENAI_API_KEY=sk-your-openai-api-key
ENABLE_AI_SUMMARIES=true
OPENAI_MODEL=gpt-4o-mini

# ArXiv Configuration
ARXIV_ENABLED=true
ARXIV_CATEGORIES=cs.AI,cs.LG,cs.CL,cs.CV
ARXIV_MAX_RESULTS=10

# Digest Schedule
DAILY_DIGEST=true
DAILY_DIGEST_TIME=07:30
WEEKLY_DIGEST=true
WEEKLY_DIGEST_DAY=Monday
WEEKLY_DIGEST_TIME=09:00

# Options
RUN_ON_STARTUP=true
```

#### 3.3 Create `Makefile`
```makefile
.PHONY: help setup test-conn run build test clean install-deps

help:
	@echo "TooManyPapers - AI News Aggregator Setup"
	@echo ""
	@echo "Commands:"
	@echo "  make setup      - Complete initial setup"
	@echo "  make test-conn  - Test MongoDB connection"
	@echo "  make run        - Run the application"
	@echo "  make build      - Build binary"
	@echo "  make clean      - Clean build files"

setup:
	@echo "📦 Installing dependencies..."
	@go mod download
	@go mod tidy
	@echo "✅ Dependencies installed"
	@echo ""
	@echo "⚠️  Please ensure you have:"
	@echo "1. Added your MongoDB password to .env"
	@echo "2. Added your email credentials to .env"
	@echo "3. (Optional) Added OpenAI API key to .env"
	@echo ""
	@echo "Run 'make test-conn' to test MongoDB connection"

test-conn:
	@go run scripts/test_connection.go

run:
	@go run cmd/api/main.go

build:
	@go build -o bin/paperaggro cmd/api/main.go
	@echo "✅ Built: bin/paperaggro"

clean:
	@rm -rf bin/

install-deps:
	@go mod download
	@go mod tidy
```

### Step 4: Install Core Application Files

Save each of these files in their respective directories:

1. **`cmd/api/main.go`** - Main application entry point (from mongodb-v2-implementation artifact)
2. **`internal/config/config.go`** - Configuration management (from mongodb-v2-implementation artifact)
3. **`internal/database/database.go`** - MongoDB operations (from mongodb-v2-implementation artifact)
4. **`internal/models/models.go`** - Data models (from setup-files artifact)

### Step 5: Create Test Connection Script

#### `scripts/test_connection.go`
```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "time"
    
    "github.com/joho/godotenv"
    "go.mongodb.org/mongo-driver/v2/bson"
    "go.mongodb.org/mongo-driver/v2/mongo"
    "go.mongodb.org/mongo-driver/v2/mongo/options"
    "go.mongodb.org/mongo-driver/v2/mongo/readpref"
)

func main() {
    // Load .env file
    if err := godotenv.Load(); err != nil {
        log.Println("No .env file found")
    }
    
    // Get MongoDB password
    password := os.Getenv("MONGODB_PASSWORD")
    if password == "" {
        log.Fatal("❌ MONGODB_PASSWORD environment variable is required")
    }
    
    // Build connection string
    uri := fmt.Sprintf(
        "mongodb+srv://o.mongodb.net/?retryWrites=true&w=majority&appName=o",
        password,
    )
    
    fmt.Println("🔗 Connecting to MongoDB Atlas...")
    
    // Create client
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    serverAPI := options.ServerAPI(options.ServerAPIVersion1)
    opts := options.Client().ApplyURI(uri).SetServerAPIOptions(serverAPI)
    
    client, err := mongo.Connect(opts)
    if err != nil {
        log.Fatal("❌ Failed to create client:", err)
    }
    defer client.Disconnect(context.Background())
    
    // Ping the database
    if err := client.Ping(ctx, readpref.Primary()); err != nil {
        log.Fatal("❌ Failed to ping:", err)
    }
    
    fmt.Println("✅ Successfully connected to MongoDB Atlas (PaperAggro cluster)!")
    
    // Test write operation
    db := client.Database("paperaggro")
    collection := db.Collection("test")
    
    testDoc := bson.M{
        "test": true,
        "timestamp": time.Now(),
        "message": "Connection test successful",
    }
    
    result, err := collection.InsertOne(ctx, testDoc)
    if err != nil {
        log.Fatal("❌ Failed to insert test document:", err)
    }
    
    fmt.Printf("✅ Test document inserted with ID: %v\n", result.InsertedID)
    
    // Clean up test document
    _, err = collection.DeleteOne(ctx, bson.M{"_id": result.InsertedID})
    if err != nil {
        log.Printf("⚠️  Warning: Failed to delete test document: %v", err)
    }
    
    fmt.Println("🎉 MongoDB connection test passed! You're ready to run PaperAggro.")
}
```

### Step 6: Create Placeholder Service Files

#### `internal/services/aggregator.go`
```go
package services

import (
    "context"
    "paperaggro/internal/config"
    "paperaggro/internal/database"
)

type AggregatorService struct {
    db     *database.Database
    config *config.Config
}

func NewAggregatorService(db *database.Database, cfg *config.Config) *AggregatorService {
    return &AggregatorService{db: db, config: cfg}
}

func (s *AggregatorService) CollectArticles(ctx context.Context) error {
    // Implementation from enhanced-aggregator artifact
    return nil
}
```

#### `internal/services/newsletter.go`
```go
package services

import (
    "paperaggro/internal/config"
    "paperaggro/internal/database"
)

type NewsletterService struct {
    db     *database.Database
    config *config.Config
}

func NewNewsletterService(db *database.Database, cfg *config.Config) *NewsletterService {
    return &NewsletterService{db: db, config: cfg}
}
```

#### `internal/services/analytics.go`
```go
package services

import (
    "paperaggro/internal/database"
)

type AnalyticsService struct {
    db *database.Database
}

func NewAnalyticsService(db *database.Database) *AnalyticsService {
    return &AnalyticsService{db: db}
}
```

### Step 7: Create Handler and Server Stubs

#### `internal/handlers/handlers.go`
```go
package handlers

import (
    "paperaggro/internal/services"
)

type Handlers struct {
    aggregator *services.AggregatorService
    newsletter *services.NewsletterService
    analytics  *services.AnalyticsService
}

func New(agg *services.AggregatorService, news *services.NewsletterService, analytics *services.AnalyticsService) *Handlers {
    return &Handlers{
        aggregator: agg,
        newsletter: news,
        analytics:  analytics,
    }
}
```

#### `internal/scheduler/scheduler.go`
```go
package scheduler

import (
    "log"
    "paperaggro/internal/config"
    "paperaggro/internal/services"
    "github.com/robfig/cron/v3"
)

type Scheduler struct {
    cron       *cron.Cron
    aggregator *services.AggregatorService
    newsletter *services.NewsletterService
    config     *config.Config
}

func New(agg *services.AggregatorService, news *services.NewsletterService, cfg *config.Config) *Scheduler {
    return &Scheduler{
        cron:       cron.New(),
        aggregator: agg,
        newsletter: news,
        config:     cfg,
    }
}

func (s *Scheduler) Start() {
    log.Println("📅 Starting scheduler...")
    s.cron.Start()
}

func (s *Scheduler) Stop() {
    s.cron.Stop()
}
```

#### `internal/server/server.go`
```go
package server

import (
    "context"
    "net/http"
    "paperaggro/internal/config"
    "paperaggro/internal/handlers"
    "github.com/gin-gonic/gin"
)

type Server struct {
    router *gin.Engine
    config *config.Config
}

func New(cfg *config.Config, h *handlers.Handlers) *Server {
    router := gin.Default()
    
    // Setup routes
    router.GET("/health", func(c *gin.Context) {
        c.JSON(200, gin.H{"status": "healthy"})
    })
    
    return &Server{
        router: router,
        config: cfg,
    }
}

func (s *Server) Start() error {
    return s.router.Run(":" + s.config.Server.Port)
}

func (s *Server) Shutdown(ctx context.Context) error {
    // Graceful shutdown logic
    return nil
}
```

## 🚀 MongoDB Atlas Setup

### 1. Create MongoDB Atlas Cluster

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register)
2. Sign up for a free account
3. Create a new project called XXXX
4. Create a free M0 cluster
5. Choose your preferred region

### 2. Configure Database Access

1. Go to **Database Access** in the left menu
2. Add a database user:
   - Username: `user`
   - Password: Create a strong password
   - Database User Privileges: Atlas Admin

### 3. Configure Network Access

1. Go to **Network Access** in the left menu
2. Add IP Address:
   - Click "Add IP Address"
   - For development: Click "Add Current IP Address"
   - For production: Add your server's IP
   - Or allow from anywhere: `0.0.0.0/0` (less secure)

### 4. Get Connection String

1. Go to **Database** → **Connect**
2. Choose "Connect your application"
3. Select "Go" as the driver
4. Copy the connection string (already configured in our code)

## 🔧 Installation & Running

### Quick Start

```bash
# 1. Clone and setup
git clone <your-repo-url>
cd repo

# 2. Install dependencies
make setup

# 3. Configure environment
cp .env.example .env
# Edit .env with your credentials:
# - MongoDB password
# - Email credentials
# - OpenAI API key (optional)

# 4. Test MongoDB connection
make test-conn

# 5. Run the application
make run
```

### First Run Checklist

- [ ] MongoDB password added to `.env`
- [ ] Email credentials configured
- [ ] OpenAI API key added (optional)
- [ ] MongoDB IP whitelist configured
- [ ] Test connection successful





## MongoDB Atlas Dashboard

1. Go to your [Atlas Dashboard](https://cloud.mongodb.com)
2. Monitor:
   - Database size
   - Number of operations
   - Performance metrics
   - Index usage

### API Endpoints

```bash
# Health check
curl http://localhost:8080/health

# Get recent articles (when implemented)
curl http://localhost:8080/api/articles

# Search articles (when implemented)
curl http://localhost:8080/api/articles/search?q=transformer

# Trigger manual collection (when implemented)
curl -X POST http://localhost:8080/api/collect \
  -H "X-API-Key: your-api-key"
```

## 🐳 Docker Deployment (Optional)


```

## 🔍 Troubleshooting

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **MongoDB connection failed** | Check password in `.env`, verify IP whitelist |
| **Email not sending** | Use Gmail app password, not regular password |
| **No articles collected** | Check RSS feed URLs are accessible |
| **OpenAI errors** | Verify API key, check rate limits |
| **Port already in use** | Change PORT in `.env` or stop other services |

### Debug Commands

```bash
# Test MongoDB connection only
go run scripts/test_connection.go

# Check environment variables
go run -e cmd/api/main.go

# Run with verbose logging
ENV=development go run cmd/api/main.go
```

## 📈 Production Deployment

### Cloud Platforms

**Heroku**
```bash
heroku create paperaggro
heroku config:set MONGODB_PASSWORD=xxx
heroku config:set OPENAI_API_KEY=xxx
git push heroku main
```

**Google Cloud Run**
```bash
gcloud run deploy paperaggro \
  --source . \
  --set-env-vars MONGODB_PASSWORD=xxx
```

**AWS EC2**
```bash
# On EC2 instance
sudo yum install golang
git clone <repo>
cd paperaggro
make build
nohup ./bin/paperaggro &
```

### Production Checklist

- [ ] Use environment variables for secrets
- [ ] Enable MongoDB Atlas backups
- [ ] Set up monitoring alerts
- [ ] Configure rate limiting
- [ ] Enable HTTPS
- [ ] Set up log aggregation
- [ ] Configure auto-scaling

## 📚 Data Flow

```
1. Scheduler triggers collection (cron)
   ↓
2. Fetch from sources (parallel)
   - RSS feeds
   - arXiv API
   - Web scraping
   ↓
3. Process articles
   - Deduplicate
   - Categorize
   - Generate AI summaries
   ↓
4. Store in MongoDB Atlas
   - Articles collection
   - Update metrics
   ↓
5. Send digests
   - Query recent articles
   - Generate HTML email
   - Send to subscribers
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

MIT License - see LICENSE file

## 🆘 Support

- Create an issue on GitHub
- Check MongoDB Atlas documentation
- Review OpenAI API documentation

