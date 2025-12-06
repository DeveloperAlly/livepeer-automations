## Discord Anouncements Automation

Discord → GitHub Pages → Mintlify integration

1. Fetches Announcements from Discord

n8n workflow runs everyday (or triggered by webhook)
Retrieves last 10 messages from announcements channel


2. Formats and Processes Messages

Extracts content, author, timestamp from Discord messages
Sorts by newest first and sanitizes content
Creates JSON structure with announcements array


3. Updates GitHub Repository

Commits updated announcements.json to GitHub repo
GitHub Pages automatically serves the static HTML page
HTML page reads JSON and renders styled announcements


4. Displays in Mintlify Docs

Mintlify embeds GitHub Pages URL via iframe
No build scripts needed - just static embed
Auto-refreshes every 5 minutes to show latest updates


The entire flow is external to Mintlify, requiring only a simple iframe embed that points to the GitHub Pages URL. 
Updates happen automatically through n8n without any manual intervention.
