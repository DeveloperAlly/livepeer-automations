## Discord Anouncements Automation

Discord → GitHub Pages → Mintlify integration

1. Fetches Announcements from Discord

n8n workflow runs every X minutes (or triggered by webhook)
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

## Implement this in your own projects
See the [Implementation Guide]("https://github.com/DeveloperAlly/livepeer-automations/blob/main/discord/announcements/implementation-guide.md")

## Architecture Flow
```mermaid
graph TD
    A[Discord<br/>Announcements Channel] 
    A -->|Webhook/API| B[n8n Workflow<br/>Every X min]
    
    B --> C[Process Messages]
    C --> D[Extract & Format<br/>• Content<br/>• Author<br/>• Timestamp]
    D --> E[Create JSON<br/>Structure]
    
    E --> F[GitHub API<br/>Update Files]
    F --> G[(GitHub Repo<br/>index.html<br/>announcements.json)]
    
    G -->|Auto Deploy| H[GitHub Pages<br/>Static Site]
    
    H -->|iframe| I[Mintlify Docs]
    
    I --> J[Users View<br/>Live Announcements]
    
    style A fill:#5865F2,color:#fff
    style B fill:#ff6d5a,color:#fff
    style G fill:#24292e,color:#fff
    style H fill:#0969da,color:#fff
    style I fill:#10b981,color:#fff
    style J fill:#8b5cf6,color:#fff
```
