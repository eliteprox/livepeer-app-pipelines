# Webapp Build and Deployment Guide

This guide explains how to build and deploy the webapp using the provided Makefile tasks.

## Prerequisites

- Node.js and npm installed locally
- SSH access to your remote server
- SSH key configured for passwordless authentication

## Quick Start

1. **Build the webapp:**
   ```bash
   make webapp-build
   ```

2. **Deploy to remote server:**
   ```bash
   make webapp-deploy REMOTE_HOST=your-server.com REMOTE_USER=your-username
   ```

3. **Build and deploy in one command:**
   ```bash
   make webapp-build-deploy REMOTE_HOST=your-server.com REMOTE_USER=your-username
   ```

## Configuration

The deployment can be configured using environment variables or command-line parameters:

| Variable | Default | Description |
|----------|---------|-------------|
| `REMOTE_USER` | `deploy` | Username for SSH connection |
| `REMOTE_HOST` | `your-server.com` | Remote server hostname |
| `REMOTE_PATH` | `/var/www/html/app` | Remote directory path |
| `SSH_KEY` | `~/.ssh/id_rsa` | SSH private key path |

## Available Tasks

Run `make help` to see all available tasks:

- `webapp-build` - Build the webapp using npm ci and npm run build
- `webapp-deploy` - Deploy the webapp dist folder to remote server using SCP
- `webapp-build-deploy` - Build and deploy the webapp in one command
- `webapp-clean` - Clean the webapp build artifacts
- `webapp-dev` - Start the webapp development server

## Examples

### Basic deployment
```bash
make webapp-build-deploy REMOTE_HOST=myserver.com REMOTE_USER=ubuntu
```

### Using custom SSH key
```bash
make webapp-deploy REMOTE_HOST=myserver.com SSH_KEY=~/.ssh/my-custom-key
```

### Using environment variables
```bash
export REMOTE_HOST=myserver.com
export REMOTE_USER=ubuntu
export REMOTE_PATH=/var/www/my-app
make webapp-build-deploy
```

### Development workflow
```bash
# Start development server
make webapp-dev

# Clean build artifacts
make webapp-clean

# Build for production
make webapp-build
```

## Server Setup

Ensure your remote server is properly configured:

1. **Create the deployment directory:**
   ```bash
   sudo mkdir -p /var/www/html/app
   sudo chown your-username:your-username /var/www/html/app
   ```

2. **Configure web server (nginx example):**
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;
       root /var/www/html/app;
       index index.html;
       
       location / {
           try_files $uri $uri/ /index.html;
       }
   }
   ```

3. **Test SSH connection:**
   ```bash
   ssh -i ~/.ssh/id_rsa your-username@your-server.com
   ```

## Troubleshooting

### SSH Connection Issues
- Verify SSH key permissions: `chmod 600 ~/.ssh/id_rsa`
- Test SSH connection manually: `ssh -i ~/.ssh/id_rsa user@host`
- Check if the remote host is in your `~/.ssh/known_hosts`

### Build Issues
- Ensure Node.js and npm are installed
- Clear npm cache: `npm cache clean --force`
- Delete node_modules and reinstall: `make webapp-clean && make webapp-build`

### Permission Issues
- Ensure the remote user has write permissions to the deployment directory
- Check if sudo is required for the remote path

## Integration with CI/CD

You can integrate these tasks into your CI/CD pipeline:

```yaml
# GitHub Actions example
- name: Build and Deploy Webapp
  run: |
    make webapp-build-deploy \
      REMOTE_HOST=${{ secrets.REMOTE_HOST }} \
      REMOTE_USER=${{ secrets.REMOTE_USER }} \
      SSH_KEY=${{ secrets.SSH_PRIVATE_KEY }}
```
