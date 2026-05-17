# LLM Wiki Setup Guide

## Windows Desktop Setup

### Step 1: Clone the wiki

```powershell
cd ~
git clone https://github.com/heinhtethtoo/llm-wiki.git
```

### Step 2: Open in Obsidian

1. Open Obsidian → "Open folder as vault"
2. Pick `C:\Users\HP\llm-wiki\`
3. You'll see the full graph view with all the MOCs, projects, and cross-links

### Step 3: Install Obsidian Git plugin (auto-sync)

1. Settings → Community Plugins → Browse → search "Obsidian Git"
2. Install + Enable
3. Set these in Obsidian Git settings:
   - Auto pull interval: 5 minutes
   - Auto push interval: 5 minutes
   - Commit message: `obsidian: {{date}}`

### Step 4: Install enquire-mcp

```bash
npm install -g @oomkapwn/enquire-mcp
enquire-mcp setup --vault ~/llm-wiki
enquire-mcp install-model multilingual
```

### Step 5: Add MCP to Claude Code Desktop

Create/edit `%USERPROFILE%\.claude\.mcp.json`:

```json
{
  "mcpServers": {
    "obsidian-wiki": {
      "command": "enquire-mcp",
      "args": ["serve", "--vault", "C:\\Users\\HP\\llm-wiki", "--persistent-index", "--enable-reranker"],
      "env": {}
    }
  }
}
```

### Step 6: Add MCP to Cursor

Create/edit `%APPDATA%\Cursor\mcp.json`:

```json
{
  "mcpServers": {
    "obsidian-wiki": {
      "command": "enquire-mcp",
      "args": ["serve", "--vault", "C:\\Users\\HP\\llm-wiki", "--persistent-index", "--enable-reranker"],
      "env": {}
    }
  }
}
```

---

## VPS Setup

### MCP for Claude Code CLI

```bash
cat > ~/.claude/.mcp.json << 'EOF'
{
  "mcpServers": {
    "obsidian-wiki": {
      "command": "enquire-mcp",
      "args": ["serve", "--vault", "/root/llm-wiki", "--persistent-index", "--enable-reranker"],
      "env": {}
    }
  }
}
EOF
```

### Wiki rules in CLAUDE.md

```bash
cat >> ~/.claude/CLAUDE.md << 'RULES'

## Wiki Maintenance (Karpathy LLM Wiki)

LLM Wiki at `/root/llm-wiki/`. Every session:
1. Log changes to `sessions/YYYY-MM-DD.md`
2. Create atomic notes for new knowledge
3. Update MOCs when things change
4. Cross-link with `[[wikilinks]]`
RULES
```

### Auto-sync cron

```bash
crontab -e
# Add:
*/30 * * * * cd /root/llm-wiki && git add -A && git diff --cached --quiet || (git commit -m "auto-sync $(date +\%Y-\%m-\%d\ \%H:\%M)" && git push origin master) 2>/dev/null
```

---

## Architecture

```
Obsidian (Windows) ──git push──→ GitHub ──git pull──→ VPS (/root/llm-wiki/)
     ↑                                                    ↓
  enquire-mcp                                        enquire-mcp
     ↓                                                    ↓
Claude Code Desktop                              Claude Code CLI
Cursor                                            OpenClaw agents
```
