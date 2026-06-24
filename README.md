# Slack Emoji → Notion Backlog

Capture any Slack message into a Notion backlog with a single emoji reaction. No copy-pasting, no tab-switching, no forgetting.

---

## What it does

When you react to a Slack message with one of the tracked emojis, n8n automatically creates (or updates) an entry in your Notion backlog database.

There are two types of emojis:

**Priority emojis** → create a new Notion entry
| Emoji | Slack name | Notion status |
|---|---|---|
| 🔴 | `:red_circle:` | Important |
| 📋 | `:clipboard:` | Urgent |
| 💡 | `:bulb:` | Ideas |

**Category emojis** → update the category on an existing entry
| Emoji | Slack name | Category |
|---|---|---|
| 🎯 | `:dart:` | Content |
| 🎓 | `:mortar_board:` | Academy |
| 👥 | `:busts_in_silhouette:` | Community |
| 📌 | `:pushpin:` | Other |

**How to use both together:**
1. React with 🔴 on a message → entry created with status "Important"
2. React with 🎯 on the same message → entry updated with category "Content"

The two emojis can be added in any order and at any time.

---

## Prerequisites

- An **n8n** instance (cloud or self-hosted). If you don't have one, sign up at [n8n.io](https://n8n.io).
- A **Slack workspace** where you have permission to create apps.
- A **Notion workspace** where you can create integrations.

---

## Setup — Step by step

Follow these steps in order.

---

### Step 1 — Create the Notion database

1. Open Notion and create a new page where you want the backlog to live.
2. Add a new **Database (full page)** inside that page.
3. Name it `Backlog Capture`.
4. Add the following properties (the names must match exactly):

| Property name | Type | Options |
|---|---|---|
| `Title` | Title | (default) |
| `Status` | Select | Important, Urgent, Ideas |
| `Category` | Select | Content, Academy, Community, Other |
| `Source` | URL | — |
| `Captured on` | Date | — |
| `Trigger emoji` | Text | — |

**Find your Database ID:**
Open the database in Notion. Look at the URL — it looks like:
```
https://www.notion.so/yourworkspace/ef271d4ec7954fd3aa83f7c11e7e2a59?v=...
```
The Database ID is the 32-character string between the last `/` and the `?`. Copy it — you'll need it in Step 4.

---

### Step 2 — Create a Notion integration

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **New integration**
3. Give it a name (e.g. `n8n Backlog`) and select your workspace
4. Copy the **Internal Integration Token** (starts with `secret_`)
5. Go back to your Notion database → click `...` in the top right → **Connections** → add your integration

---

### Step 3 — Create a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From scratch**
2. Give it a name (e.g. `Backlog Bot`) and select your workspace

**Add OAuth scopes:**
Go to **OAuth & Permissions → Bot Token Scopes** and add:
- `reactions:read`
- `channels:history`
- `groups:history`
- `im:history`
- `mpim:history`

**Install the app:**
Go to **Install App** → **Install to Workspace** → authorize. Copy the **Bot User OAuth Token** (starts with `xoxb-`).

---

### Step 4 — Import and configure the n8n workflow

1. In n8n, click **+** to create a new workflow → **Import from file** → select `workflow.json`
2. Open the **Notion - Create Page** node:
   - Connect your Notion credential (use the `secret_` token from Step 2)
   - Replace `YOUR_NOTION_DATABASE_ID` with the ID you copied in Step 1
3. Do the same for **Notion - Search Existing Page** and **Notion - Update Category**
4. Open the **HTTP Request - Get Message** node:
   - Set authentication to **Header Auth**
   - Header name: `Authorization` / Value: `Bearer xoxb-your-token-here`
5. **Activate the workflow** using the toggle in the top right → click **Publish**
6. Copy the webhook URL from the **Trigger - Slack Reaction Webhook** node — it looks like:
   ```
   https://YOUR-N8N-INSTANCE/webhook/slack-reaction
   ```

---

### Step 5 — Connect Slack to n8n

1. Go back to your Slack app → **Event Subscriptions**
2. Toggle **Enable Events** to On
3. Paste your n8n webhook URL in the **Request URL** field
4. Wait for the ✅ **Verified** status (n8n must be active for this to work)
5. Under **Subscribe to bot events**, add: `reaction_added`
6. Click **Save Changes**
7. Go to **Install App** → **Reinstall to Workspace** (required after any scope or event change)

---

### Step 6 — Invite the bot to your Slack channel

The bot can only see reactions in channels it's a member of.

In the Slack channel where you want to capture ideas, type:
```
/invite @your-app-name
```

> **Tip:** Create a dedicated `#capture` channel and invite the bot there. Forward any message you want to capture into that channel, then react with the emoji.

---

## Testing it works

1. Go to your `#capture` channel in Slack
2. Post any message
3. React with 🔴 on it
4. Check your Notion database — a new entry should appear within a few seconds
5. Now react with 🎯 on the same message → the entry should update with category "Content"

---

## Troubleshooting

**Slack says "challenge_failed" when I paste the webhook URL**
→ Make sure the workflow is **active** (published) in n8n. The test URL (`/webhook-test/...`) won't work for Slack verification — use the production URL (`/webhook/slack-reaction`).

**n8n receives nothing when I react**
→ The bot is probably not in the channel. Run `/invite @your-app-name` in the channel.

**Error: `not_authed` in the HTTP Request node**
→ Your Slack Bot Token is missing or wrong. Check the `Authorization` header in the HTTP Request node — it must be `Bearer xoxb-...` with no extra spaces.

**Error: `Credentials not found` on a Notion node**
→ The credential isn't assigned to that node. Click the node → look for the credential dropdown at the top → select your Notion credential.

**The entry is created but Status/Category are empty**
→ Check that the property names in your Notion database match exactly: `Status`, `Category`, `Source`, `Captured on`, `Trigger emoji`.

**Category emoji doesn't update the entry**
→ The workflow searches for the entry by its `Source` URL. Make sure the priority emoji was added first (to create the entry), and that the source URL format matches.

---

## Stack

- **n8n** — workflow automation
- **Slack API** — `reaction_added` event + `conversations.history`
- **Notion API** — database page create + update
