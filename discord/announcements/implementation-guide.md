# Discord Announcements to Mintlify Implementation Guide

## Architecture Overview
**Flow:** Discord → n8n workflow → GitHub Repository → GitHub Pages → Mintlify (iframe embed)

## Step 1: GitHub Repository Setup

### 1.1 Create Repository
1. Create a new GitHub repository named `discord-announcements`
2. Make it public (required for GitHub Pages)
3. Initialize with README.md

### 1.2 Enable GitHub Pages
1. Go to Settings → Pages
2. Source: Deploy from branch
3. Branch: main
4. Folder: / (root)
5. Save and wait for deployment

### 1.3 Create GitHub Personal Access Token
1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Permissions needed: `repo` (full control)
4. Save token for n8n configuration

## Step 2: Repository Files

### File: `index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="300">
    <title>Discord Announcements</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            line-height: 1.6;
            color: #1a202c;
            background: transparent;
            padding: 20px;
        }

        /* Dark mode support */
        @media (prefers-color-scheme: dark) {
            body {
                color: #e2e8f0;
            }
            .announcement {
                background: rgba(45, 55, 72, 0.5);
                border-left-color: #7289da !important;
            }
            .timestamp {
                color: #a0aec0 !important;
            }
            .author {
                color: #cbd5e0 !important;
            }
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        .header {
            margin-bottom: 24px;
            padding-bottom: 12px;
            border-bottom: 1px solid #e2e8f0;
        }

        .header h2 {
            font-size: 1.5rem;
            font-weight: 600;
            color: #5865F2;
        }

        .last-updated {
            font-size: 0.875rem;
            color: #718096;
            margin-top: 4px;
        }

        .announcement {
            background: #f7fafc;
            border-left: 4px solid #5865F2;
            padding: 16px;
            margin-bottom: 16px;
            border-radius: 4px;
            transition: transform 0.2s ease;
        }

        .announcement:hover {
            transform: translateX(4px);
        }

        .announcement-content {
            margin-bottom: 8px;
            white-space: pre-wrap;
            word-wrap: break-word;
        }

        .announcement-meta {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 8px;
        }

        .author {
            font-weight: 500;
            color: #4a5568;
            font-size: 0.875rem;
        }

        .timestamp {
            color: #718096;
            font-size: 0.75rem;
        }

        .no-announcements {
            text-align: center;
            padding: 40px;
            color: #718096;
        }

        .loading {
            text-align: center;
            padding: 20px;
            color: #718096;
        }

        .error {
            background: #fed7d7;
            color: #c53030;
            padding: 12px;
            border-radius: 4px;
            margin-bottom: 16px;
        }

        /* Detect parent theme */
        .dark-theme body {
            background: #1a202c;
            color: #e2e8f0;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h2>📢 Latest Announcements</h2>
            <div class="last-updated" id="lastUpdated"></div>
        </div>
        <div id="announcements-container">
            <div class="loading">Loading announcements...</div>
        </div>
    </div>

    <script>
        // Try to detect parent window theme
        try {
            if (window.parent && window.parent.document) {
                const parentIsDark = window.parent.document.documentElement.classList.contains('dark');
                if (parentIsDark) {
                    document.documentElement.classList.add('dark-theme');
                }
            }
        } catch (e) {
            // Cross-origin, can't access parent
        }

        // Load announcements
        async function loadAnnouncements() {
            try {
                const response = await fetch('announcements.json?t=' + Date.now());
                const data = await response.json();
                
                const container = document.getElementById('announcements-container');
                const lastUpdatedEl = document.getElementById('lastUpdated');
                
                if (data.announcements && data.announcements.length > 0) {
                    container.innerHTML = data.announcements.map(announcement => `
                        <div class="announcement">
                            <div class="announcement-content">${escapeHtml(announcement.content)}</div>
                            <div class="announcement-meta">
                                <span class="author">@${escapeHtml(announcement.author)}</span>
                                <span class="timestamp">${formatDate(announcement.timestamp)}</span>
                            </div>
                        </div>
                    `).join('');
                } else {
                    container.innerHTML = '<div class="no-announcements">No announcements yet</div>';
                }
                
                lastUpdatedEl.textContent = `Last updated: ${formatDate(data.lastUpdated)}`;
                
            } catch (error) {
                document.getElementById('announcements-container').innerHTML = 
                    '<div class="error">Failed to load announcements. Please try again later.</div>';
                console.error('Error loading announcements:', error);
            }
        }

        function escapeHtml(text) {
            const map = {
                '&': '&amp;',
                '<': '&lt;',
                '>': '&gt;',
                '"': '&quot;',
                "'": '&#039;'
            };
            return text.replace(/[&<>"']/g, m => map[m]);
        }

        function formatDate(dateString) {
            const date = new Date(dateString);
            const now = new Date();
            const diff = now - date;
            
            // Less than 1 hour
            if (diff < 3600000) {
                const minutes = Math.floor(diff / 60000);
                return minutes <= 1 ? 'just now' : `${minutes} minutes ago`;
            }
            
            // Less than 24 hours
            if (diff < 86400000) {
                const hours = Math.floor(diff / 3600000);
                return hours === 1 ? '1 hour ago' : `${hours} hours ago`;
            }
            
            // Less than 7 days
            if (diff < 604800000) {
                const days = Math.floor(diff / 86400000);
                return days === 1 ? 'yesterday' : `${days} days ago`;
            }
            
            // Default to date
            return date.toLocaleDateString('en-US', { 
                month: 'short', 
                day: 'numeric', 
                year: date.getFullYear() !== now.getFullYear() ? 'numeric' : undefined 
            });
        }

        // Load on page load
        loadAnnouncements();
        
        // Refresh every 5 minutes
        setInterval(loadAnnouncements, 300000);
    </script>
</body>
</html>
```

### File: `announcements.json`
```json
{
    "lastUpdated": "2024-12-06T12:00:00Z",
    "announcements": [
        {
            "id": "1234567890",
            "content": "Welcome to our announcement system!",
            "author": "Admin",
            "timestamp": "2024-12-06T12:00:00Z",
            "channelId": "announcement-channel-id"
        }
    ]
}
```

## Step 3: n8n Workflow Configuration

### 3.1 Workflow Overview
```json
{
  "name": "Discord Announcements to GitHub",
  "nodes": [
    "Schedule Trigger",
    "Discord - Get Messages",
    "Format Announcements",
    "Update GitHub JSON",
    "Update GitHub HTML"
  ]
}
```

### 3.2 Detailed Node Configuration

#### Node 1: Schedule Trigger
- **Trigger Times:** Every 30 minutes
- **Or use:** Discord Webhook trigger for real-time updates

#### Node 2: Discord Configuration
- **Resource:** Message
- **Operation:** Get Many
- **Channel ID:** Your announcements channel ID
- **Limit:** 10
- **Return All:** false

#### Node 3: Code Node - Format Data
```javascript
// Format Discord messages for storage
const messages = $input.all();
const announcements = [];

for (const item of messages) {
  const msg = item.json;
  
  // Skip messages without content
  if (!msg.content || msg.content.trim() === '') continue;
  
  announcements.push({
    id: msg.id,
    content: msg.content,
    author: msg.author?.username || 'Unknown',
    timestamp: msg.timestamp || new Date().toISOString(),
    channelId: msg.channel_id
  });
}

// Sort by timestamp (newest first)
announcements.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

// Limit to 10 most recent
const limitedAnnouncements = announcements.slice(0, 10);

const output = {
  lastUpdated: new Date().toISOString(),
  announcements: limitedAnnouncements
};

return [{
  json: output,
  binary: {}
}];
```

#### Node 4: GitHub - Update JSON File
- **Resource:** File
- **Operation:** Edit
- **Repository Owner:** your-username
- **Repository Name:** discord-announcements
- **File Path:** announcements.json
- **Commit Message:** `Update announcements - {{$now}}`
- **Content:** `{{JSON.stringify($json, null, 2)}}`

### 3.3 Alternative: Using HTTP Request Nodes
```javascript
// If GitHub node isn't available, use HTTP Request:

// Get current file SHA (required for updates)
GET: https://api.github.com/repos/{owner}/{repo}/contents/announcements.json

// Update file
PUT: https://api.github.com/repos/{owner}/{repo}/contents/announcements.json
Headers: 
  Authorization: token YOUR_GITHUB_TOKEN
  Accept: application/vnd.github.v3+json
Body:
{
  "message": "Update announcements",
  "content": "BASE64_ENCODED_CONTENT",
  "sha": "CURRENT_FILE_SHA"
}
```

## Step 4: Discord Webhook Setup (Optional - for real-time)

### 4.1 Create Discord Webhook
1. Go to Discord Server Settings → Integrations → Webhooks
2. Create New Webhook
3. Copy Webhook URL

### 4.2 Configure in n8n
1. Use Webhook trigger instead of Schedule
2. Set Discord to send to your n8n webhook URL
3. Process incoming webhook data

## Step 5: Mintlify Integration

### 5.1 Basic iframe Embed
```mdx
---
title: Announcements
description: Latest updates from our Discord server
---

# 📢 Announcements

Stay updated with the latest news from our Discord community.

<iframe 
  src="https://YOUR-USERNAME.github.io/discord-announcements/"
  width="100%"
  height="600"
  style={{
    border: "1px solid #e2e8f0",
    borderRadius: "8px",
    backgroundColor: "transparent"
  }}
  frameBorder="0"
/>

[Join our Discord](https://discord.gg/your-invite) to participate in discussions!
```

### 5.2 Advanced Embed with Parameters
```mdx
<iframe 
  src="https://YOUR-USERNAME.github.io/discord-announcements/index.html?limit=5"
  width="100%"
  height="400"
  sandbox="allow-scripts allow-same-origin"
  loading="lazy"
/>
```

### 5.3 Using Mintlify's Frame Component
```mdx
<Frame caption="Latest Discord Announcements">
  <div 
    dangerouslySetInnerHTML={{
      __html: '<iframe src="https://YOUR-USERNAME.github.io/discord-announcements/" width="100%" height="500" style="border:none;" />'
    }} 
  />
</Frame>
```

## Step 6: Testing & Troubleshooting

### 6.1 Test GitHub Pages
1. Visit: `https://YOUR-USERNAME.github.io/discord-announcements/`
2. Check browser console for errors
3. Verify announcements.json loads

### 6.2 Test n8n Workflow
1. Execute manually first
2. Check GitHub repo for updated files
3. Monitor execution logs

### 6.3 Common Issues & Solutions

**CORS Issues:**
- GitHub Pages serves with proper CORS headers by default
- If issues persist, consider using raw.githubusercontent.com

**Authentication Failures:**
- Verify GitHub token permissions
- Check token expiration
- Ensure Discord bot has channel access

**Styling Issues:**
- Test in both light and dark modes
- Adjust CSS for your Mintlify theme
- Use browser DevTools to inspect iframe

## Step 7: Enhancements

### 7.1 Add Filtering
```javascript
// URL parameters for filtering
const params = new URLSearchParams(window.location.search);
const limit = parseInt(params.get('limit')) || 10;
const author = params.get('author');
const search = params.get('search');

// Apply filters
let filtered = data.announcements;
if (author) filtered = filtered.filter(a => a.author === author);
if (search) filtered = filtered.filter(a => a.content.includes(search));
filtered = filtered.slice(0, limit);
```

### 7.2 Add Categories/Tags
```javascript
// In Discord, use tags like [UPDATE], [FEATURE], [BUG]
const category = announcement.content.match(/\[([^\]]+)\]/)?.[1] || 'General';
```

### 7.3 Rich Content Support
- Parse Discord markdown
- Support embeds and attachments
- Display reactions count

## Security Considerations

1. **Sanitize all Discord content** to prevent XSS
2. **Use read-only GitHub tokens** where possible
3. **Implement rate limiting** in n8n workflow
4. **Add Content Security Policy** headers in HTML
5. **Validate webhook signatures** if using Discord webhooks

## Maintenance

- **Monitor:** Set up n8n execution failure alerts
- **Backup:** Keep local copies of announcements.json
- **Update:** Regularly update GitHub token before expiration
- **Clean:** Implement announcement archiving after X days

---

## Quick Start Checklist

- [ ] Create GitHub repository
- [ ] Enable GitHub Pages
- [ ] Create GitHub personal access token
- [ ] Upload index.html and announcements.json
- [ ] Import n8n workflow
- [ ] Configure Discord credentials in n8n
- [ ] Test workflow execution
- [ ] Add iframe to Mintlify docs
- [ ] Verify display in Mintlify
- [ ] Set up monitoring/alerts

## Support Resources

- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [n8n Discord Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.discord/)
- [Mintlify Components](https://mintlify.com/docs/components)
- [Discord Developer Portal](https://discord.com/developers/docs)
