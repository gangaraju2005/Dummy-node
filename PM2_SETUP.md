# PM2 Setup Command Reference

## Quick Setup for AWS EC2

### One-line commands to setup PM2 and run your app:

```bash
# 1. Install PM2 globally (run once)
sudo npm install -g pm2

# 2. Navigate to your app directory
cd ~/app

# 3. Start the application with PM2
pm2 start app.js --name app

# 4. Enable auto-restart on server reboot (run once)
pm2 startup
pm2 save

# 5. Verify it's running
pm2 status
pm2 logs app
```

---

## PM2 Configuration File Method (Optional)

Create `ecosystem.config.js` in your app root for advanced configuration:

```javascript
module.exports = {
  apps: [
    {
      name: "app",
      script: "./app.js",
      instances: 1,
      exec_mode: "cluster",
      env: {
        NODE_ENV: "production",
        PORT: 3000
      },
      error_file: "./logs/err.log",
      out_file: "./logs/out.log",
      log_file: "./logs/combined.log",
      time_format: "YYYY-MM-DD HH:mm:ss Z",
      merge_logs: true
    }
  ]
};
```

Then use:
```bash
pm2 start ecosystem.config.js
pm2 save
```

---

## Essential PM2 Commands

### Manage Application:
```bash
# Start application
pm2 start app.js --name app

# Stop application
pm2 stop app

# Restart application
pm2 restart app

# Delete application from PM2
pm2 delete app

# View all processes
pm2 status

# View detailed information
pm2 show app
```

### View Logs:
```bash
# View live logs
pm2 logs app

# View last 50 lines
pm2 logs app --lines 50

# Clear logs
pm2 flush

# View error logs only
pm2 logs app --err
```

### System Management:
```bash
# Enable auto-start on server boot
pm2 startup

# Save current PM2 process list
pm2 save

# Resurrect saved process list
pm2 resurrect

# Kill all processes
pm2 kill

# Restart all processes
pm2 restart all
```

---

## Healthcare & Monitoring (Optional):

```bash
# Restart crashed apps after 10 seconds
pm2 install pm2-auto-pull

# Monitor CPU/Memory
pm2 monit

# Set max memory restart threshold (300MB)
pm2 start app.js --max-memory-restart 300M

# Watch for file changes and auto-restart
pm2 start app.js --watch

# Auto-restart on file changes in specific directories
pm2 start app.js --watch ./public --watch ./app.js
```

---

## Production Best Practices

### Full setup for production:

```bash
#!/bin/bash
# Setup script for production deployment

# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Install Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2 globally
sudo npm install -g pm2

# Create app directory with proper permissions
sudo mkdir -p /var/www/app
sudo chown $USER:$USER /var/www/app
cd /var/www/app

# Clone repository (replace with your repo)
git clone https://github.com/YOUR_USERNAME/Dummy-node.git .

# Install dependencies
npm install --production

# Start with PM2
pm2 start app.js --name app --max-memory-restart 300M

# Enable auto-start on reboot
pm2 startup
pm2 save

# Verify
pm2 status
pm2 logs app
```

### Environment Variables:

```bash
# Set environment variables for PM2 process
pm2 start app.js --name app --update-env

# Or in ecosystem.config.js:
# env: {
#   NODE_ENV: "production",
#   PORT: 3000
# }

# Then restart:
pm2 restart app --update-env
```

---

## Windows Development (for testing):

```bash
# Install PM2 globally
npm install -g pm2

# Install Windows service manager for PM2
npm install -g pm2-windows-service

# Start application
pm2 start app.js --name app

# Install as Windows service (optional)
pm2-service-install
```

---

## Continuous Deployment Integration

The GitHub Actions workflow automatically runs these commands:

```bash
# Executed on EC2 during CD job:
cd ~/app
git pull origin main         # Get latest code
npm install --production     # Update dependencies
pm2 restart app              # Restart the process
# OR if first time:
# pm2 start app.js --name app
pm2 save                      # Save process list
pm2 status                    # Verify
```

---

## Quick Reference - Workflow

```
GitHub Push
    ↓
CI Job (test + build)
    ↓ (if succeeded)
CD Job (deploy to EC2)
    ↓
SSH to EC2
    ↓
git pull origin main
    ↓
npm install --production
    ↓
pm2 restart app (or pm2 start)
    ↓
pm2 save
    ↓
Deployment Complete ✅
```

---

## Troubleshooting PM2

### Application won't start:
```bash
# Check if port is already in use
sudo lsof -i :3000

# Kill the process
kill -9 <PID>

# Try starting again
pm2 start app.js --name app
```

### Check what's going wrong:
```bash
# View logs
pm2 logs app

# View with timestamps
pm2 logs app --lines 100

# Check PM2 error file
cat ~/app/logs/err.log
```

### Reset PM2:
```bash
# Remove all processes
pm2 delete all

# Kill PM2 daemon
pm2 kill

# Start fresh
pm2 start app.js --name app
pm2 save
```
