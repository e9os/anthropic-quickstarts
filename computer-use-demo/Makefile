# Computer Use Demo - Development Makefile
# Provides easy commands for development workflow with auto-sync and restart

# Configuration
DOCKER_IMAGE := computer-use-demo:local
CONTAINER_NAME := computer-use-demo-dev
ENV_FILE := .env

# Default target
.PHONY: help
help:
	@echo "Computer Use Demo - Development Commands"
	@echo ""
	@echo "Setup & Development:"
	@echo "  make setup     - Configure venv, install dependencies, and pre-commit hooks"
	@echo "  make dev       - Start development environment with auto-sync"
	@echo "  make build     - Build Docker image locally"
	@echo ""
	@echo "Management:"
	@echo "  make stop      - Stop the development container"
	@echo "  make restart   - Restart the development container"
	@echo "  make logs      - Follow container logs"
	@echo "  make shell     - Get shell access to running container"
	@echo ""
	@echo "Cleanup:"
	@echo "  make clean     - Stop container and remove local image"
	@echo "  make clean-all - Deep clean: remove everything (with confirmation)"
	@echo ""
	@echo "Utilities:"
	@echo "  make status    - Show container status"
	@echo "  make config    - Show current configuration"

# Load environment variables from .env file if it exists
ifneq (,$(wildcard $(ENV_FILE)))
    include $(ENV_FILE)
    export
endif

# Setup development environment
.PHONY: setup
setup:
	@echo "🔧 Setting up development environment..."
	@if [ ! -f "$(ENV_FILE)" ]; then \
		echo "📝 Creating sample .env file..."; \
		cp .env.example $(ENV_FILE) 2>/dev/null || echo "# Copy this file to .env and add your API key\nANTHROPIC_API_KEY=your_api_key_here\nAPI_PROVIDER=anthropic\nWIDTH=1024\nHEIGHT=768" > $(ENV_FILE); \
		echo "⚠️  Please edit $(ENV_FILE) and add your API key"; \
	fi
	@echo "🐍 Running setup script..."
	@./setup.sh
	@echo "✅ Setup complete!"

# Build Docker image
.PHONY: build
build:
	@echo "🐳 Building Docker image..."
	@docker build . -t $(DOCKER_IMAGE)
	@echo "✅ Build complete!"

# Start development environment with auto-sync
.PHONY: dev
dev: check-env
	@echo "🚀 Starting development environment..."
	@echo "📁 Auto-sync enabled for computer_use_demo/ directory"
	@echo "🔄 Streamlit auto-reload is enabled"
	@echo ""
	@echo "Access points:"
	@echo "  - Combined interface: http://localhost:8080"
	@echo "  - Streamlit only:     http://localhost:8501"
	@echo "  - Desktop view:       http://localhost:6080/vnc.html"
	@echo "  - VNC direct:         vnc://localhost:5900"
	@echo ""
	@docker run --rm -d \
		--name $(CONTAINER_NAME) \
		-e ANTHROPIC_API_KEY=$(ANTHROPIC_API_KEY) \
		$(if $(API_PROVIDER),-e API_PROVIDER=$(API_PROVIDER)) \
		$(if $(WIDTH),-e WIDTH=$(WIDTH)) \
		$(if $(HEIGHT),-e HEIGHT=$(HEIGHT)) \
		$(if $(AWS_PROFILE),-e AWS_PROFILE=$(AWS_PROFILE)) \
		$(if $(AWS_REGION),-e AWS_REGION=$(AWS_REGION)) \
		$(if $(AWS_ACCESS_KEY_ID),-e AWS_ACCESS_KEY_ID=$(AWS_ACCESS_KEY_ID)) \
		$(if $(AWS_SECRET_ACCESS_KEY),-e AWS_SECRET_ACCESS_KEY=$(AWS_SECRET_ACCESS_KEY)) \
		$(if $(AWS_SESSION_TOKEN),-e AWS_SESSION_TOKEN=$(AWS_SESSION_TOKEN)) \
		$(if $(VERTEX_REGION),-e CLOUD_ML_REGION=$(VERTEX_REGION)) \
		$(if $(VERTEX_PROJECT_ID),-e ANTHROPIC_VERTEX_PROJECT_ID=$(VERTEX_PROJECT_ID)) \
		-v $(PWD)/computer_use_demo:/home/computeruse/computer_use_demo/ \
		-v $(HOME)/.anthropic:/home/computeruse/.anthropic \
		$(if $(AWS_PROFILE),-v $(HOME)/.aws:/home/computeruse/.aws) \
		$(if $(VERTEX_PROJECT_ID),-v $(HOME)/.config/gcloud/application_default_credentials.json:/home/computeruse/.config/gcloud/application_default_credentials.json) \
		-p 5900:5900 \
		-p 8501:8501 \
		-p 6080:6080 \
		-p 8080:8080 \
		$(DOCKER_IMAGE) || (echo "❌ Failed to start container. Try 'make build' first." && exit 1)
	@echo "✅ Development environment started!"
	@echo "📊 Use 'make logs' to follow the logs"

# Check if required environment variables are set
.PHONY: check-env
check-env:
	@if [ ! -f "$(ENV_FILE)" ]; then \
		echo "❌ $(ENV_FILE) file not found. Run 'make setup' first."; \
		exit 1; \
	fi
	@if [ -z "$(ANTHROPIC_API_KEY)" ] && [ "$(API_PROVIDER)" != "bedrock" ] && [ "$(API_PROVIDER)" != "vertex" ]; then \
		echo "❌ ANTHROPIC_API_KEY not set in $(ENV_FILE)"; \
		echo "   Please edit $(ENV_FILE) and add your API key"; \
		exit 1; \
	fi

# Stop the development container
.PHONY: stop
stop:
	@echo "🛑 Stopping development container..."
	@docker stop $(CONTAINER_NAME) 2>/dev/null || echo "Container not running"
	@echo "✅ Container stopped"

# Restart the development container
.PHONY: restart
restart: stop dev

# Follow container logs
.PHONY: logs
logs:
	@echo "📋 Following container logs (Ctrl+C to exit)..."
	@docker logs -f $(CONTAINER_NAME) 2>/dev/null || echo "❌ Container not running. Use 'make dev' to start."

# Get shell access to running container
.PHONY: shell
shell:
	@echo "🐚 Opening shell in container..."
	@docker exec -it $(CONTAINER_NAME) /bin/bash || echo "❌ Container not running. Use 'make dev' to start."

# Show container status
.PHONY: status
status:
	@echo "📊 Container Status:"
	@docker ps -a --filter name=$(CONTAINER_NAME) --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" || echo "No containers found"
	@echo ""
	@echo "🖼️  Docker Images:"
	@docker images $(DOCKER_IMAGE) --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}" || echo "No local image found"

# Show current configuration
.PHONY: config
config:
	@echo "⚙️  Current Configuration:"
	@echo "  Docker Image: $(DOCKER_IMAGE)"
	@echo "  Container:    $(CONTAINER_NAME)"
	@echo "  Config File:  $(ENV_FILE)"
	@if [ -f "$(ENV_FILE)" ]; then \
		echo ""; \
		echo "📝 Environment Variables:"; \
		grep -v '^#' $(ENV_FILE) | grep -v '^$$' | sed 's/^/  /' || true; \
	else \
		echo "  ❌ No $(ENV_FILE) file found"; \
	fi

# Basic cleanup - stop container and remove local image
.PHONY: clean
clean:
	@echo "🧹 Cleaning up development environment..."
	@echo "  - Stopping container..."
	@docker stop $(CONTAINER_NAME) 2>/dev/null || true
	@echo "  - Removing container..."
	@docker rm $(CONTAINER_NAME) 2>/dev/null || true
	@echo "  - Removing local image..."
	@docker rmi $(DOCKER_IMAGE) 2>/dev/null || true
	@echo "  - Cleaning up dangling images..."
	@docker image prune -f >/dev/null 2>&1 || true
	@echo "✅ Basic cleanup complete!"

# Deep cleanup - remove everything with confirmation
.PHONY: clean-all
clean-all:
	@echo "🚨 DEEP CLEAN WARNING"
	@echo "This will remove:"
	@echo "  - All project containers and images"
	@echo "  - Docker build cache"
	@echo "  - Python virtual environment (.venv/)"
	@echo "  - Pre-commit hooks"
	@echo ""
	@echo "This will NOT remove:"
	@echo "  - Your $(ENV_FILE) configuration"
	@echo "  - Your ~/.anthropic settings"
	@echo ""
	@read -p "Continue with deep clean? [y/N] " confirm && [ "$$confirm" = "y" ] || (echo "Cancelled." && exit 1)
	@echo ""
	@echo "🧹 Performing deep cleanup..."
	@echo "  - Stopping and removing containers..."
	@docker stop $(CONTAINER_NAME) 2>/dev/null || true
	@docker rm $(CONTAINER_NAME) 2>/dev/null || true
	@echo "  - Removing images..."
	@docker rmi $(DOCKER_IMAGE) 2>/dev/null || true
	@docker rmi ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest 2>/dev/null || true
	@echo "  - Cleaning Docker cache..."
	@docker system prune -f >/dev/null 2>&1 || true
	@echo "  - Removing Python virtual environment..."
	@rm -rf .venv/ 2>/dev/null || true
	@echo "  - Removing pre-commit hooks..."
	@rm -rf .git/hooks/pre-commit 2>/dev/null || true
	@echo "✅ Deep cleanup complete!"
	@echo "💡 Run 'make setup' to reinitialize the development environment"

# Force rebuild - clean and build
.PHONY: rebuild
rebuild: clean build

# Development with specific screen size
.PHONY: dev-large
dev-large:
	@$(MAKE) dev WIDTH=1920 HEIGHT=1080

.PHONY: dev-small
dev-small:
	@$(MAKE) dev WIDTH=800 HEIGHT=600
