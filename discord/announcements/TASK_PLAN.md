# Task Plan: Discord Announcements Implementation

## Overview
This task plan outlines the steps to implement a Discord announcements system that integrates Discord → n8n → GitHub Pages → Mintlify for the livepeer-automations repository.

## Status: Planning Phase

---

## Phase 1: Repository Setup
**Goal:** Set up the basic repository structure and GitHub Pages

### Tasks:
- [x] Verify repository structure exists
  - Location: `discord/announcements/`
  - Contains: README.md, implementation-guide.md, repo-specific-implementation-guide.md
  
- [ ] Create required files in discord/announcements/
  - [ ] index.html - Frontend display for announcements
  - [ ] announcements.json - Data file with announcement content
  
- [ ] Verify GitHub Pages configuration
  - [ ] Check if GitHub Pages is enabled
  - [ ] Ensure deployment is from main branch, root directory
  - [ ] Verify URL: https://developerally.github.io/livepeer-automations/discord/announcements/

---

## Phase 2: Create Frontend Files
**Goal:** Build the announcement display interface

### Task 2.1: Create index.html
**Description:** Build responsive HTML page with:
- Dark mode support
- Auto-refresh every 5 minutes
- Responsive design
- Proper error handling
- XSS protection with escapeHtml function
- Relative path loading for announcements.json

**Key Features:**
- Discord-themed styling (#5865F2)
- Timestamp formatting (relative times)
- Author display
- Loading and error states
- Parent theme detection for iframe embedding

### Task 2.2: Create announcements.json
**Description:** Initial data file with schema:
```json
{
  "lastUpdated": "ISO 8601 timestamp",
  "announcements": [
    {
      "id": "message-id",
      "content": "announcement text",
      "author": "username",
      "timestamp": "ISO 8601 timestamp",
      "channelId": "channel-id"
    }
  ]
}
```

**Requirements:**
- Valid JSON structure
- Sample announcement for testing
- Proper date formatting

---

## Phase 3: n8n Workflow Configuration
**Goal:** Set up automation to fetch Discord messages and update GitHub

### Task 3.1: GitHub Personal Access Token
**Actions Required:**
1. Navigate to GitHub Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Required permissions: `repo` (full control)
4. Save token securely for n8n configuration
5. Document token creation process

### Task 3.2: n8n Workflow Design
**Nodes Required:**
1. **Schedule Trigger Node**
   - Frequency: Every 30 minutes (configurable)
   - Alternative: Discord webhook for real-time

2. **Discord Node**
   - Resource: Message
   - Operation: Get Many
   - Channel ID: [To be configured]
   - Limit: 10 messages
   - Return All: false

3. **Code Node - Format Data**
   - Extract: id, content, author, timestamp, channelId
   - Sort by newest first
   - Limit to 10 most recent
   - Create JSON structure
   - Handle empty messages

4. **GitHub Node - Update JSON**
   - Repository: DeveloperAlly/livepeer-automations
   - File Path: discord/announcements/announcements.json
   - Operation: Edit
   - Commit message: "Update announcements - {{$now}}"
   - Content: Formatted JSON

### Task 3.3: Alternative HTTP Request Configuration
**Fallback if GitHub node unavailable:**
- GET: Retrieve current file SHA
- PUT: Update file with new content
- Headers: Authorization, Accept
- Body: Base64 encoded content + SHA

---

## Phase 4: Mintlify Integration
**Goal:** Embed announcements in Mintlify documentation

### Task 4.1: Create Integration Documentation
**Content includes:**
- Basic iframe embed code
- Responsive styling
- Parameter options (limit, author, search)
- Frame component examples
- Best practices for embedding

### Task 4.2: Example Implementations
**Provide code samples for:**
1. Basic embed (full width, 600px height)
2. Advanced embed with parameters
3. Mintlify Frame component usage
4. Custom styling options

**Example iframe URL:**
```
https://developerally.github.io/livepeer-automations/discord/announcements/
```

---

## Phase 5: Testing & Validation
**Goal:** Ensure all components work correctly

### Task 5.1: GitHub Pages Testing
- [ ] Verify URL accessibility
- [ ] Check announcements.json loads correctly
- [ ] Test in multiple browsers
- [ ] Validate dark/light mode switching
- [ ] Check mobile responsiveness

### Task 5.2: n8n Workflow Testing
- [ ] Manual workflow execution
- [ ] Verify Discord message fetching
- [ ] Confirm GitHub file updates
- [ ] Check error handling
- [ ] Monitor execution logs

### Task 5.3: Integration Testing
- [ ] Test iframe embed in Mintlify
- [ ] Verify cross-origin functionality
- [ ] Check auto-refresh behavior
- [ ] Test with real Discord data
- [ ] Validate styling in embedded context

---

## Phase 6: Documentation
**Goal:** Provide comprehensive guides for users

### Task 6.1: Troubleshooting Guide
**Common Issues to Document:**
- CORS problems (subdirectory considerations)
- Authentication failures
- Styling issues in iframe
- Deployment delays
- JSON parsing errors

**Solutions for:**
- Relative vs absolute paths
- Token permissions
- GitHub Pages configuration
- n8n node configuration
- Discord API access

### Task 6.2: Security Documentation
**Key Points:**
- XSS prevention (escapeHtml function)
- Token security (read-only where possible)
- Rate limiting in n8n
- Content Security Policy headers
- Webhook signature validation

### Task 6.3: Maintenance Guide
**Ongoing Tasks:**
- n8n execution monitoring
- Alert setup for failures
- Token rotation schedule
- Backup strategy for announcements.json
- Archiving old announcements

---

## Phase 7: Enhancements (Optional)
**Goal:** Add advanced features

### Potential Enhancements:
- [ ] URL parameter filtering (author, search, limit)
- [ ] Category/tag support ([UPDATE], [FEATURE], [BUG])
- [ ] Discord markdown parsing
- [ ] Embed and attachment support
- [ ] Reaction count display
- [ ] Real-time webhook integration
- [ ] Analytics integration
- [ ] Notification system

---

## Success Criteria

### Must Have:
- [ ] index.html displaying announcements
- [ ] announcements.json with valid data structure
- [ ] GitHub Pages serving files correctly
- [ ] n8n workflow fetching and updating data
- [ ] Mintlify iframe integration working
- [ ] Documentation for setup and maintenance

### Nice to Have:
- Real-time Discord webhook integration
- Advanced filtering options
- Rich content support
- Analytics tracking

---

## Timeline Estimates

| Phase | Estimated Time | Priority |
|-------|---------------|----------|
| Phase 1 | 1 hour | High |
| Phase 2 | 2-3 hours | High |
| Phase 3 | 3-4 hours | High |
| Phase 4 | 1-2 hours | Medium |
| Phase 5 | 2-3 hours | High |
| Phase 6 | 2-3 hours | Medium |
| Phase 7 | 4-6 hours | Low |

**Total Estimated Time:** 15-22 hours (excluding optional enhancements)

---

## Dependencies

### External Services:
- GitHub Pages (enabled)
- n8n instance (configured)
- Discord API access
- GitHub Personal Access Token

### Technical Requirements:
- HTML/CSS/JavaScript knowledge
- n8n workflow experience
- GitHub API familiarity
- Discord bot/webhook setup

---

## Risk Assessment

### High Risk:
- **GitHub Pages not enabled** → Verify in repository settings
- **Discord API access issues** → Confirm bot permissions
- **n8n authentication failures** → Test token before deployment

### Medium Risk:
- **CORS issues with iframe** → Use relative paths, test thoroughly
- **Rate limiting on APIs** → Implement proper delays
- **Token expiration** → Set up renewal reminders

### Low Risk:
- **Styling inconsistencies** → Test in multiple browsers
- **Mobile responsiveness** → Use standard CSS practices

---

## Next Steps

1. [x] Create this task plan document
2. [ ] Begin Phase 2: Create index.html and announcements.json
3. [ ] Verify GitHub Pages configuration (Phase 1)
4. [ ] Document n8n workflow setup (Phase 3)
5. [ ] Test end-to-end integration (Phase 5)

---

## Notes

- This implementation uses the existing `livepeer-automations` repository
- Files go in `/discord/announcements/` subdirectory
- GitHub Pages URL includes subdirectory path
- All file paths in n8n must include full subdirectory
- Use relative paths (`./announcements.json`) in HTML/JS
- The repo-specific guide contains the actual implementation details
- This task plan provides the execution roadmap

---

## References

- [repo-specific-implementation-guide.md](./repo-specific-implementation-guide.md) - Detailed implementation guide
- [implementation-guide.md](./implementation-guide.md) - General implementation guide
- [README.md](./README.md) - Project overview and architecture
