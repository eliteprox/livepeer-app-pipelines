# Makefile for byoc-stream webapp build and deployment

# Default values - can be overridden via environment variables or command line
WEBAPP_DIR := webapp
DIST_DIR := $(WEBAPP_DIR)/dist
REMOTE_USER ?= deploy
REMOTE_HOST ?= your-server.com
REMOTE_PATH ?= /var/www/html/app
SSH_KEY ?= ~/.ssh/id_rsa

# Colors for output
GREEN := \033[0;32m
YELLOW := \033[1;33m
RED := \033[0;31m
NC := \033[0m # No Color

.PHONY: help webapp-build webapp-deploy webapp-build-deploy webapp-clean webapp-dev

help: ## Show this help message
	@echo "Available targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  $(YELLOW)%-20s$(NC) %s\n", $$1, $$2}'
	@echo ""
	@echo "Environment variables:"
	@echo "  $(YELLOW)REMOTE_USER$(NC)    Remote server username (default: deploy)"
	@echo "  $(YELLOW)REMOTE_HOST$(NC)    Remote server hostname (default: your-server.com)"
	@echo "  $(YELLOW)REMOTE_PATH$(NC)    Remote server path (default: /var/www/html/app)"
	@echo "  $(YELLOW)SSH_KEY$(NC)        SSH private key path (default: ~/.ssh/id_rsa)"
	@echo ""
	@echo "Example usage:"
	@echo "  make webapp-build-deploy REMOTE_HOST=myserver.com REMOTE_USER=ubuntu"

webapp-build: ## Build the webapp using npm ci and npm run build
	@echo "$(GREEN)Building webapp...$(NC)"
	@if [ ! -d "$(WEBAPP_DIR)" ]; then \
		echo "$(RED)Error: webapp directory not found at $(WEBAPP_DIR)$(NC)"; \
		exit 1; \
	fi
	cd $(WEBAPP_DIR) && npm ci
	cd $(WEBAPP_DIR) && npm run build
	@echo "$(GREEN)✓ Webapp build completed successfully$(NC)"
	@echo "$(GREEN)✓ Built files are in $(DIST_DIR)$(NC)"

webapp-deploy: ## Deploy the webapp dist folder to remote server using SCP
	@echo "$(GREEN)Deploying webapp to $(REMOTE_USER)@$(REMOTE_HOST):$(REMOTE_PATH)...$(NC)"
	@if [ ! -d "$(DIST_DIR)" ]; then \
		echo "$(RED)Error: dist directory not found at $(DIST_DIR)$(NC)"; \
		echo "$(YELLOW)Run 'make webapp-build' first$(NC)"; \
		exit 1; \
	fi
	@echo "$(YELLOW)Testing SSH connection...$(NC)"
	@ssh -i $(SSH_KEY) -o ConnectTimeout=10 -o BatchMode=yes $(REMOTE_USER)@$(REMOTE_HOST) exit || \
		(echo "$(RED)Error: Cannot connect to $(REMOTE_USER)@$(REMOTE_HOST)$(NC)" && exit 1)
	@echo "$(YELLOW)Creating remote directory if it doesn't exist...$(NC)"
	@ssh -i $(SSH_KEY) $(REMOTE_USER)@$(REMOTE_HOST) "mkdir -p $(REMOTE_PATH)"
	@echo "$(YELLOW)Uploading files...$(NC)"
	@scp -i $(SSH_KEY) -r $(DIST_DIR)/* $(REMOTE_USER)@$(REMOTE_HOST):$(REMOTE_PATH)/
	@echo "$(YELLOW)Cleaning up any empty directories...$(NC)"
	@ssh -i $(SSH_KEY) $(REMOTE_USER)@$(REMOTE_HOST) "find $(REMOTE_PATH) -type d -empty -delete 2>/dev/null || true"
	@echo "$(GREEN)✓ Webapp deployed successfully to $(REMOTE_USER)@$(REMOTE_HOST):$(REMOTE_PATH)$(NC)"

webapp-build-deploy: webapp-build webapp-deploy ## Build and deploy the webapp in one command
	@echo "$(GREEN)✓ Build and deployment completed successfully$(NC)"

webapp-clean: ## Clean the webapp build artifacts
	@echo "$(GREEN)Cleaning webapp build artifacts...$(NC)"
	@if [ -d "$(DIST_DIR)" ]; then \
		rm -rf $(DIST_DIR); \
		echo "$(GREEN)✓ Removed $(DIST_DIR)$(NC)"; \
	else \
		echo "$(YELLOW)No dist directory to clean$(NC)"; \
	fi
	@if [ -d "$(WEBAPP_DIR)/node_modules" ]; then \
		rm -rf $(WEBAPP_DIR)/node_modules; \
		echo "$(GREEN)✓ Removed $(WEBAPP_DIR)/node_modules$(NC)"; \
	else \
		echo "$(YELLOW)No node_modules directory to clean$(NC)"; \
	fi

webapp-dev: ## Start the webapp development server
	@echo "$(GREEN)Starting webapp development server...$(NC)"
	@if [ ! -d "$(WEBAPP_DIR)" ]; then \
		echo "$(RED)Error: webapp directory not found at $(WEBAPP_DIR)$(NC)"; \
		exit 1; \
	fi
	cd $(WEBAPP_DIR) && npm install && npm run dev

# Validation targets (internal use)
validate-remote-config:
	@if [ "$(REMOTE_HOST)" = "your-server.com" ]; then \
		echo "$(RED)Error: Please set REMOTE_HOST to your actual server hostname$(NC)"; \
		echo "$(YELLOW)Example: make webapp-deploy REMOTE_HOST=myserver.com$(NC)"; \
		exit 1; \
	fi

# Override deploy target to include validation
webapp-deploy: validate-remote-config
webapp-build-deploy: validate-remote-config
