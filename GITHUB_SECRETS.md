# GitHub Secrets Configuration

## Required Secrets for CI/CD Pipeline

Configure these secrets in your GitHub repository before deploying.

### Location:
**Settings** → **Secrets and variables** → **Actions** → **New repository secret**

---

## Secrets to Create

### 1. EC2_HOST
**Description:** The IP address or DNS hostname of your AWS EC2 instance

**Example values:**
- `123.45.67.89` (IP address)
- `ec2-12-345-67-89.compute-1.amazonaws.com` (AWS DNS)
- `my-app-server.example.com` (Custom domain)

**How to find:**
- Log into AWS Console
- Go to EC2 Dashboard
- Select your instance
- Copy "Public IPv4 address" or "Public IPv4 DNS"

---

### 2. EC2_USERNAME
**Description:** SSH username for connecting to EC2 instance

**Example values by AMI:**
- `ubuntu` (Ubuntu AMI - most common)
- `ec2-user` (Amazon Linux 2 AMI)
- `admin` (Debian AMI)
- `centos` (CentOS AMI)

**How to find:**
- Check your EC2 AMI details in AWS Console
- Or try connecting first: `ssh -i key.pem ubuntu@your-ip`

---

### 3. EC2_SSH_KEY
**Description:** The complete contents of your PEM private key file

**For Security:**
- 🔐 NEVER commit this to version control
- 🔐 NEVER share publicly
- 🔐 GitHub Actions masks this in logs
- 🔐 Only the GitHub Actions runner can access it

**How to create (if you don't have a key pair):**

**In AWS Console:**
1. Go to EC2 Dashboard → **Key Pairs** (under Security)
2. Click **Create key pair**
3. Name it (e.g., `my-app-deploy-key`)
4. Choose format: **pem** (for Linux/Mac) or **ppk** (for PuTTY)
5. Click **Create key pair**
6. Save the downloaded `.pem` file securely

**How to extract key contents for GitHub Secret:**

**On macOS/Linux:**
```bash
cat ~/.ssh/my-app-deploy-key.pem
```

**On Windows (PowerShell):**
```powershell
Get-Content C:\Users\YourUsername\.ssh\my-app-deploy-key.pem
```

**On Windows (Command Prompt):**
```cmd
type C:\Users\YourUsername\.ssh\my-app-deploy-key.pem
```

**What to copy:**
```
-----BEGIN PRIVATE KEY-----
MIIEvAIBADANBgkqhkiG9w0BAQE...
... (many lines) ...
-----END PRIVATE KEY-----
```

**Steps to add to GitHub:**
1. Copy the entire PEM file contents (including BEGIN and END lines)
2. Go to GitHub → Settings → Secrets and variables → Actions
3. Click **New repository secret**
4. Name: `EC2_SSH_KEY`
5. Value: Paste the entire PEM contents
6. Click **Add secret**

---

## Verification Checklist

After adding all three secrets, verify they're configured:

1. ✅ Go to **Settings** → **Secrets and variables** → **Actions**
2. ✅ You should see three secrets listed:
   - `EC2_HOST`
   - `EC2_USERNAME`
   - `EC2_SSH_KEY`

3. ✅ Click on each to verify values are set (they will be masked)

---

## Testing the Configuration

1. Push a commit to the `main` branch
2. Go to **Actions** tab
3. Click on the latest workflow run
4. If secrets are configured correctly:
   - CI job should complete successfully
   - CD job should start after CI succeeds
   - CD job should connect to EC2 and deploy

---

## Troubleshooting

**Error: "ssh: connect to host X.X.X.X port 22: Connection timed out"**
- Verify EC2_HOST is correct
- Check EC2 Security Group allows SSH (port 22) inbound
- Verify EC2 instance is running in AWS Console

**Error: "Permission denied (publickey)"**
- Verify EC2_SSH_KEY contains the full PEM file contents
- Verify EC2_USERNAME matches your AMI type
- Ensure no extra spaces/newlines when copying PEM contents

**Error: "No such file or directory"**
- Verify EC2_HOST value (no extra spaces)
- Check SSH key file path during troubleshooting

---

## Security Notes

✅ **What GitHub does with secrets:**
- Encrypted at rest
- Hidden in workflow logs
- Only accessible to Actions workflows
- Can't be viewed after creation (only modified/deleted)

✅ **Best practices:**
- Use dedicated SSH key for CI/CD (not your personal key)
- Regularly rotate SSH keys
- Delete compromised keys immediately
- Monitor GitHub Actions logs for suspicious activity
- Use restricted SSH key permissions (600)

---

## Reference

**Workflow file:** `.github/workflows/ci.yml`
**Workflow jobs:**
- `ci` - Build and test (always runs)
- `deploy` - Deploy to EC2 (only after CI succeeds on push to main)

