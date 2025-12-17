# Local Installation Guide

This guide will help you set up and run The Graph Protocol Metrics Dashboard on your local machine.

## Prerequisites

- Python 3.7 or higher
- pip (Python package manager)
- Git
- A Graph API key from [The Graph Studio](https://thegraph.com/studio/)

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/pdiomede/metrics.git
cd metrics
```

### 2. Install Python Dependencies

```bash
pip install requests python-dotenv
```

Or if you're using Python 3 specifically:

```bash
pip3 install requests python-dotenv
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

Edit the `.env` file and add your Graph API key:

```bash
GRAPH_API_KEY=your_actual_api_key_here
```

**How to get a Graph API Key:**
1. Visit [The Graph Studio](https://thegraph.com/studio/)
2. Connect your wallet
3. Go to your account settings
4. Generate a new API key
5. Copy and paste it into your `.env` file

### 4. Run the Dashboard Generator

Execute the Python script:

```bash
python generate_protocol_metrics.py
```

Or with Python 3:

```bash
python3 generate_protocol_metrics.py
```

### 5. View the Dashboard

After running the script, an `index.html` file will be generated in the same directory.

Open it in your web browser:

```bash
# On macOS
open index.html

# On Linux
xdg-open index.html

# On Windows
start index.html
```

Or simply drag and drop the `index.html` file into your browser.

## Expected Output

When you run the script, you should see output similar to:

```
[2025-12-17 10:30:00 UTC] Starting The Graph Protocol Metrics Dashboard Generator...
[2025-12-17 10:30:00 UTC] Version: v0.0.1
[2025-12-17 10:30:00 UTC] Fetching network subgraph counts...
[2025-12-17 10:30:15 UTC] Fetched subgraph and indexer counts for 45 networks.
[2025-12-17 10:30:15 UTC] Dashboard saved to index.html
[2025-12-17 10:30:15 UTC] Dashboard generation completed successfully!
```

## Troubleshooting

### Python Not Found

If you get a "command not found" error for `python`, try:
- `python3` instead of `python`
- Check if Python is installed: `python --version` or `python3 --version`
- Install Python from [python.org](https://www.python.org/downloads/)

### Module Not Found Error

If you get "ModuleNotFoundError: No module named 'requests'" or similar:

```bash
pip install requests python-dotenv
# or
pip3 install requests python-dotenv
```

### API Key Error

If you see "ERROR: GRAPH_API_KEY not found in environment variables":
1. Make sure your `.env` file exists in the same directory as the script
2. Check that the `.env` file contains `GRAPH_API_KEY=your_key`
3. Verify there are no spaces around the `=` sign
4. Make sure your API key is valid

### Permission Denied

On Linux/macOS, you might need to make the script executable:

```bash
chmod +x generate_protocol_metrics.py
```

### Network/API Issues

If data fetching fails:
1. Check your internet connection
2. Verify your API key is valid and has not expired
3. Check The Graph Network status
4. Try running the script again after a few minutes

## Updating the Dashboard

To refresh the dashboard with the latest data:

```bash
python generate_protocol_metrics.py
```

This will regenerate the `index.html` file with current metrics.

## Scheduling Automatic Updates (Optional)

### On macOS/Linux (using cron)

Edit your crontab:

```bash
crontab -e
```

Add this line to run the script every 6 hours:

```cron
0 */6 * * * cd /path/to/metrics && /usr/bin/python3 generate_protocol_metrics.py >> logs/dashboard.log 2>&1
```

### On Windows (using Task Scheduler)

1. Open Task Scheduler
2. Create a new task
3. Set the trigger (e.g., daily at specific time)
4. Set the action to run: `python.exe` with arguments: `C:\path\to\metrics\generate_protocol_metrics.py`

## Development Tips

### Testing Changes

After modifying the script, regenerate the dashboard:

```bash
python generate_protocol_metrics.py
```

Then refresh your browser to see the changes.

### Viewing Raw Data

The script logs all fetched data. To see detailed information, check the console output when running the script.

## Next Steps

- Customize the dashboard styling (edit the script's HTML template)
- Add more metrics or filter options
- Deploy to a web server (see INSTALL_VPS.md)
- Set up automated updates with cron jobs

## Support

For issues or questions:
- Check the [GitHub repository](https://github.com/pdiomede/metrics)
- Review the [README.md](README.md) for general information
- Open an issue on GitHub

## Version

This guide is for version **v0.0.1** (December 17, 2025)
