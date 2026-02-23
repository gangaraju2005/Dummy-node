# Deployment Guide - CI/CD Pipeline

## Overview
This document provides setup instructions for the AWS EC2 deployment pipeline using GitHub Actions.

---

## Prerequisites

1. **AWS EC2 Instance**
   - Running a Linux-based OS (Ubuntu recommended)
   - SSH access enabled
   - Node.js 18+ installed
   - PM2 installed globally

2. **GitHub Repository**
   - Repository secrets configured
   - Workflow file in `.github/workflows/ci.yml`

---

## Step 1: Setup Node.js and PM2 on EC2

### SSH into your EC2 instance:
```bash
ssh -i your-key.pem ec2-user@your-ec2-ip
# or for Ubuntu:
ssh -i your-key.pem ubuntu@your-ec2-ip
```

### Install Node.js 18:
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version
```

### Install PM2 globally:
```bash
sudo npm install -g pm2
pm2 --version
```

### Create application directory:
```bash
mkdir -p ~/app
cd ~/app
```

---

## Step 2: Initial Application Setup (First Time Only)

### Clone repository:
```bash
git clone https://github.com/YOUR_USERNAME/Dummy-node.git .
```

### Install dependencies:
```bash
npm install --production
```

### Start application with PM2:
```bash
pm2 start app.js --name app
pm2 status
```

### Enable PM2 auto-start on server reboot:
```bash
pm2 startup
pm2 save
```

This will create a startup script that runs PM2 on system reboot.

---

## Step 3: Configure GitHub Secrets

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**

Create the following secrets:

### Required Secrets:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `EC2_HOST` | EC2 instance IP or DNS | `123.45.67.89` or `ec2-12-345-67-89.compute-1.amazonaws.com` |
| `EC2_USERNAME` | SSH username for EC2 | `ubuntu` (for Ubuntu AMI) or `ec2-user` (for Amazon Linux) |
| `EC2_SSH_KEY` | Private SSH key (PEM file contents) | (see below) |

### How to add EC2_SSH_KEY:

1. Open your PEM file in a text editor:
   ```bash
   cat your-key-name.pem
   ```

2. Copy the entire contents (including `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----`)

3. In GitHub:
   - Go to **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `EC2_SSH_KEY`
   - Value: Paste the entire PEM file contents
   - Click **Add secret**

---

## Step 4: Workflow Execution

### Workflow Triggers:
- **CI** runs on every push and pull request to `main` branch
- **CD** runs only on successful push to `main` (not on PRs)
- CD depends on CI using the `needs: ci` keyword

### Deployment Flow:
1. Push code to `main` branch
2. GitHub Actions triggers CI job (tests + artifacts)
3. If CI succeeds → CD job starts automatically
4. CD job:
   - Connects to EC2 via SSH
   - Pulls latest code from GitHub
   - Runs `npm install`
   - Restarts PM2 process
   - Verifies deployment

### Monitoring:
- View workflow runs in **Actions** tab
- Check logs for each job and step
- Download artifacts from CI job

---

## Step 5: Monitoring and Troubleshooting

### SSH into EC2 to check PM2 status:
```bash
pm2 status
pm2 logs app
pm2 logs app --lines 50
```

### Manually restart the application:
```bash
pm2 restart app
```

### View PM2 process details:
```bash
pm2 show app
```

### Common Issues:

**Issue: "Permission denied (publickey)"**
- Verify EC2_SSH_KEY secret contains the full PEM file
- Check EC2_USERNAME matches your AMI (ubuntu, ec2-user, etc.)
- Ensure Security Group allows SSH (port 22)

**Issue: "npm install fails"**
- SSH to EC2 and run `node --version` to verify Node.js is installed
- Check disk space: `df -h`
- Check internet connectivity: `ping google.com`

**Issue: "Port already in use"**
- Change PORT in app.js or run: `sudo lsof -i :3000` to find process
- Kill process: `kill -9 <PID>`

---

## Security Best Practices

✅ **Implemented:**
- SSH key stored as GitHub Secret (never in code)
- EC2 credentials never logged or exposed
- SSH key file permissions set to 600 (read-only)
- SSH key deleted after deployment
- CD only runs on successful CI
- CD only deploys on push to main (not on PRs)

✅ **Additional Recommendations:**
- Regularly rotate SSH keys
- Use security groups to restrict SSH to known IPs
- Enable EC2 instance monitoring
- Set up CloudWatch alerts
- Keep Node.js and dependencies updated
- Use environment variables in app.js for sensitive configs

---

## Rollback Procedure

If deployment breaks:

```bash
# SSH into EC2
ssh -i your-key.pem ubuntu@your-ec2-ip

# Check PM2 history
pm2 logs app

# Restart previous version
pm2 restart app

# Or manually:
cd ~/app
git checkout previous-version
npm install
pm2 restart app
```

---

## Next Steps

1. ✅ Create GitHub Secrets (EC2_HOST, EC2_USERNAME, EC2_SSH_KEY)
2. ✅ Setup EC2 instance with Node.js and PM2
3. ✅ Test workflow by pushing to main branch
4. ✅ Monitor first deployment in Actions tab
5. ✅ Verify application running on EC2

