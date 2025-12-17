# VPS Installation & Deployment Guide

This guide will help you deploy The Graph Protocol Metrics Dashboard on your VPS server.

## Server Setup Overview

- **Source code location**: `/home/graph/ftpbox/metrics`
- **Live dashboard location**: `/var/www/iproot/metrics`
- **Web server**: Assumes Nginx or Apache is configured to serve from `/var/www/iproot/`

## Prerequisites

- VPS with SSH access
- Python 3.7+ installed on the VPS
- Nginx or Apache web server configured
- Git installed on the VPS
- A Graph API key from [The Graph Studio](https://thegraph.com/studio/)

## Installation Steps

### 1. Connect to Your VPS

```bash
ssh your_user@your_vps_ip
```

### 2. Navigate to Source Code Directory

```bash
cd /home/graph/ftpbox
```

### 3. Clone the Repository

```bash
git clone https://github.com/pdiomede/metrics.git
cd metrics
```

### 4. Install Python Dependencies

Check Python version:

```bash
python3 --version
```

Install required packages:

```bash
pip3 install requests python-dotenv
```

If `pip3` is not available, install it:

```bash
# For Ubuntu/Debian
sudo apt-get update
sudo apt-get install python3-pip

# For CentOS/RHEL
sudo yum install python3-pip
```

### 5. Configure Environment Variables

Create the `.env` file:

```bash
cp .env.example .env
nano .env
```

Add your Graph API key:

```bash
GRAPH_API_KEY=your_actual_api_key_here
```

Save and exit (Ctrl+X, then Y, then Enter in nano).

### 6. Set Up Web Server Directory

Create the live dashboard directory if it doesn't exist:

```bash
sudo mkdir -p /var/www/iproot/metrics
```

Set proper permissions:

```bash
sudo chown -R $USER:$USER /var/www/iproot/metrics
sudo chmod -R 755 /var/www/iproot/metrics
```

### 7. Create Deployment Script

Create a deployment script to automate the process:

```bash
nano /home/graph/ftpbox/metrics/deploy.sh
```

Add the following content:

```bash
#!/bin/bash
# Deploy script for The Graph Protocol Metrics Dashboard
# Version: v0.0.1

SOURCE_DIR="/home/graph/ftpbox/metrics"
WEB_DIR="/var/www/iproot/metrics"
LOG_DIR="/home/graph/ftpbox/metrics/logs"

# Create logs directory if it doesn't exist
mkdir -p "$LOG_DIR"

# Log file with timestamp
LOG_FILE="$LOG_DIR/deploy_$(date +%Y%m%d_%H%M%S).log"

echo "==================================" | tee -a "$LOG_FILE"
echo "Dashboard Deployment Started" | tee -a "$LOG_FILE"
echo "Date: $(date)" | tee -a "$LOG_FILE"
echo "==================================" | tee -a "$LOG_FILE"

# Navigate to source directory
cd "$SOURCE_DIR" || exit 1

# Generate the dashboard
echo "Generating dashboard..." | tee -a "$LOG_FILE"
python3 generate_protocol_metrics.py >> "$LOG_FILE" 2>&1

# Check if generation was successful
if [ ! -f "index.html" ]; then
    echo "ERROR: Dashboard generation failed!" | tee -a "$LOG_FILE"
    exit 1
fi

# Copy generated dashboard to web directory
echo "Copying dashboard to web directory..." | tee -a "$LOG_FILE"
cp index.html "$WEB_DIR/" >> "$LOG_FILE" 2>&1

# Copy any assets if they exist (logos, images, etc.)
if [ -d "images" ]; then
    cp -r images "$WEB_DIR/" >> "$LOG_FILE" 2>&1
fi

# Set proper permissions
chmod 644 "$WEB_DIR/index.html" >> "$LOG_FILE" 2>&1

echo "==================================" | tee -a "$LOG_FILE"
echo "Dashboard deployed successfully!" | tee -a "$LOG_FILE"
echo "Location: $WEB_DIR/index.html" | tee -a "$LOG_FILE"
echo "==================================" | tee -a "$LOG_FILE"

# Keep only last 30 days of logs
find "$LOG_DIR" -name "deploy_*.log" -mtime +30 -delete
```

Make the script executable:

```bash
chmod +x /home/graph/ftpbox/metrics/deploy.sh
```

### 8. Test the Deployment

Run the deployment script:

```bash
/home/graph/ftpbox/metrics/deploy.sh
```

Verify the dashboard is accessible:

```bash
ls -la /var/www/iproot/metrics/
```

You should see `index.html` in the directory.

### 9. Configure Web Server

#### For Nginx

Edit your Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/default
```

Add or verify this location block exists:

```nginx
server {
    listen 80;
    server_name your_domain.com;
    
    # Other configurations...
    
    location /metrics {
        alias /var/www/iproot/metrics;
        index index.html;
        try_files $uri $uri/ =404;
    }
}
```

Test Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

#### For Apache

Edit your Apache configuration:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Add:

```apache
<Directory /var/www/iproot/metrics>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

Alias /metrics /var/www/iproot/metrics
```

Reload Apache:

```bash
sudo systemctl reload apache2
```

### 10. Set Up Automated Updates (Cron Job)

Edit crontab:

```bash
crontab -e
```

Add this line to run the deployment script every 6 hours:

```cron
0 */6 * * * /home/graph/ftpbox/metrics/deploy.sh >> /home/graph/ftpbox/metrics/logs/cron.log 2>&1
```

Or to run daily at 2 AM:

```cron
0 2 * * * /home/graph/ftpbox/metrics/deploy.sh >> /home/graph/ftpbox/metrics/logs/cron.log 2>&1
```

Verify cron job is set:

```bash
crontab -l
```

## Testing the Deployment

### 1. Check if the dashboard is accessible

Open your browser and navigate to:

```
http://your_domain.com/metrics
```

Or with IP:

```
http://your_vps_ip/metrics
```

### 2. Verify the script runs correctly

```bash
cd /home/graph/ftpbox/metrics
python3 generate_protocol_metrics.py
```

### 3. Check logs

```bash
tail -f /home/graph/ftpbox/metrics/logs/deploy_*.log
```

## Updating the Dashboard

### Manual Update

To manually update the dashboard with latest data:

```bash
/home/graph/ftpbox/metrics/deploy.sh
```

### Pull Latest Code from GitHub

To update the script itself:

```bash
cd /home/graph/ftpbox/metrics
git pull origin main
/home/graph/ftpbox/metrics/deploy.sh
```

## Monitoring and Maintenance

### Check Dashboard Status

```bash
# Check if files exist
ls -la /var/www/iproot/metrics/

# Check file age
stat /var/www/iproot/metrics/index.html
```

### View Deployment Logs

```bash
# View latest deployment log
tail -50 /home/graph/ftpbox/metrics/logs/deploy_*.log

# View cron execution log
tail -50 /home/graph/ftpbox/metrics/logs/cron.log
```

### Monitor Disk Space

```bash
df -h /home/graph/ftpbox/metrics
df -h /var/www/iproot/metrics
```

### Clean Up Old Logs

The deployment script automatically removes logs older than 30 days, but you can manually clean up:

```bash
find /home/graph/ftpbox/metrics/logs -name "*.log" -mtime +30 -delete
```

## Troubleshooting

### Dashboard Not Updating

1. Check if cron is running:
```bash
sudo systemctl status cron
```

2. Check cron logs:
```bash
grep CRON /var/log/syslog
```

3. Manually run the deployment script:
```bash
/home/graph/ftpbox/metrics/deploy.sh
```

### Permission Issues

```bash
# Fix source directory permissions
sudo chown -R $USER:$USER /home/graph/ftpbox/metrics
sudo chmod -R 755 /home/graph/ftpbox/metrics

# Fix web directory permissions
sudo chown -R www-data:www-data /var/www/iproot/metrics
sudo chmod -R 755 /var/www/iproot/metrics
```

### Python Module Not Found

```bash
# Reinstall dependencies
pip3 install --upgrade requests python-dotenv
```

### Web Server Not Serving Files

For Nginx:
```bash
sudo nginx -t
sudo systemctl status nginx
sudo systemctl restart nginx
```

For Apache:
```bash
sudo apachectl configtest
sudo systemctl status apache2
sudo systemctl restart apache2
```

### API Key Issues

Verify your `.env` file:
```bash
cat /home/graph/ftpbox/metrics/.env
```

Make sure it contains:
```
GRAPH_API_KEY=your_actual_api_key
```

## Security Considerations

### Protect Your API Key

Ensure `.env` file has proper permissions:

```bash
chmod 600 /home/graph/ftpbox/metrics/.env
```

Never commit `.env` to git:

```bash
# Verify .gitignore includes .env
cat /home/graph/ftpbox/metrics/.gitignore | grep .env
```

### Web Server Security

Ensure web directory doesn't allow script execution:

For Nginx, add to location block:
```nginx
location /metrics {
    alias /var/www/iproot/metrics;
    location ~ \.(py|sh|env)$ {
        deny all;
    }
}
```

## Backup Strategy

Create a backup script:

```bash
nano /home/graph/ftpbox/metrics/backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/home/graph/backups/metrics"
DATE=$(date +%Y%m%d)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/metrics_backup_$DATE.tar.gz" \
    /home/graph/ftpbox/metrics \
    --exclude='*.pyc' \
    --exclude='.git'

# Keep only last 7 days of backups
find "$BACKUP_DIR" -name "metrics_backup_*.tar.gz" -mtime +7 -delete
```

Make executable and add to cron:

```bash
chmod +x /home/graph/ftpbox/metrics/backup.sh
crontab -e
```

Add weekly backup (every Sunday at 3 AM):
```cron
0 3 * * 0 /home/graph/ftpbox/metrics/backup.sh
```

## Directory Structure on VPS

After complete setup, your structure should look like:

```
/home/graph/ftpbox/metrics/
├── .env                          # API keys (not in git)
├── .env.example                  # Template
├── .git/                         # Git repository
├── .gitignore                    # Ignore rules
├── CHANGELOG.md
├── deploy.sh                     # Deployment script
├── generate_protocol_metrics.py # Main script
├── LICENSE
├── logs/                         # Log files
│   ├── deploy_*.log
│   └── cron.log
├── README.md
└── index.html                    # Generated (temporary)

/var/www/iproot/metrics/
├── index.html                    # Live dashboard
└── images/                       # Assets (if any)
```

## Support and Updates

- **GitHub**: https://github.com/pdiomede/metrics
- **Check for updates**: `cd /home/graph/ftpbox/metrics && git pull`
- **View version**: Check the VERSION variable in `generate_protocol_metrics.py`

## Version

This guide is for version **v0.0.1** (December 17, 2025)

---

**Need help?** Open an issue on [GitHub](https://github.com/pdiomede/metrics/issues)
