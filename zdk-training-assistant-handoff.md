# ZDK Training Assistant — Claude Code Handoff

## What We're Building

A self-contained HTML training assistant that replaces an instructor-led class on the Zenoss ZenPack Development Kit (ZDK). Students open the HTML file, enter their Anthropic API key, and get an AI tutor that knows the ZDK tutorial material deeply and walks them through it interactively.

**Goal:** Near-zero ongoing cost (~$5/month covers a few students), no server required, deploys as a single HTML file.

---

## Source Material

All content is publicly available at:

- **Background/overview:** https://zenpacks.zenoss.io/zdk/background/
- **Tutorials index:** https://zenpacks.zenoss.io/zdk/start/tutorials/
- **SNMP tutorial** (the primary one, beginner-friendly):
  - Overview: https://zenpacks.zenoss.io/zdk/start/snmp/monitoring-an-snmp-device/
  - Step 1 – Tools: https://zenpacks.zenoss.io/zdk/start/snmp/tools/
  - Step 2 – Device monitoring: https://zenpacks.zenoss.io/zdk/start/snmp/device-monitoring/
  - Step 3 – Device modeling: https://zenpacks.zenoss.io/zdk/start/snmp/device-modeling/
  - Step 4 – Component modeling: https://zenpacks.zenoss.io/zdk/start/snmp/component-modeling/
  - Step 5 – Component monitoring: https://zenpacks.zenoss.io/zdk/start/snmp/component-monitoring/
  - Step 6 – SNMP traps: https://zenpacks.zenoss.io/zdk/start/snmp/traps/
- **HTTP tutorial** (more advanced):
  - Overview: https://zenpacks.zenoss.io/zdk/start/http/monitoring-an-http-api/
  - NWS API: https://zenpacks.zenoss.io/zdk/start/http/nws-api/
  - Create ZenPack: https://zenpacks.zenoss.io/zdk/start/http/create-zenpack/
  - Modeler plugin: https://zenpacks.zenoss.io/zdk/start/http/create-modeler-plugin/
  - Add device class: https://zenpacks.zenoss.io/zdk/start/http/add-device/
  - Datasource (events): https://zenpacks.zenoss.io/zdk/start/http/plugin-events/
  - Datasource (data points): https://zenpacks.zenoss.io/zdk/start/http/plugin-data-points/
  - Datasource (modeling): https://zenpacks.zenoss.io/zdk/start/http/plugin-modeling/
- **Reference:**
  - Troubleshooting: https://zenpacks.zenoss.io/zdk/reference/troubleshooting/
  - CLI: https://zenpacks.zenoss.io/zdk/reference/cli-reference/
  - zenpack.yaml: https://zenpacks.zenoss.io/zdk/reference/zenpack-yaml/

---

## What Was Already Built (Claude Artifact Prototype)

A working prototype was built as a Claude artifact. It has:

- **Sidebar navigation** with all 15 tutorial steps across SNMP + HTTP tracks
- **API key entry screen** (key goes directly to Anthropic, never stored)
- **Chat interface** with per-step AI context — when a student selects a step, the system prompt switches to a detailed context block for that step
- **Quick-prompt buttons** per step (common questions students ask)
- **Greeting message** per step that acts as a natural conversation opener
- **Calls `claude-sonnet-4-20250514`** via the Anthropic messages API

The prototype works but needs to move out of the artifact sandbox into a real deployable HTML file.

---

## Architecture for Claude Code

### Single HTML file

Everything self-contained: HTML + CSS + JS in one file. No build step, no npm, no server. Students just open it in a browser or you host it on GitHub Pages / S3 / internal web server.

### API call pattern

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": apiKey,
    "anthropic-version": "2023-06-01",
    "anthropic-dangerous-direct-browser-access": "true"  // required for browser CORS
  },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    system: systemPrompt,   // per-step context (see below)
    messages: conversationHistory
  })
});
```

> **Note:** The `anthropic-dangerous-direct-browser-access: true` header is required to make direct browser-to-API calls work (bypasses CORS restrictions Anthropic normally enforces). This is fine for an internal tool where you control the key.

### System prompt structure

Each tutorial step has its own context block that gets injected as the system prompt:

```
You are an expert ZDK instructor...
[shared instructor persona and style guidelines]

CURRENT LESSON CONTEXT:
[step-specific content: key concepts, code examples, common gotchas, doc URL]
```

The prototype has context blocks written for all 15 steps. Claude Code should fetch and read each tutorial page URL to expand/improve these context blocks before writing the final file.

### Conversation management

- Keep last 20 messages in history to stay within context limits
- Reset history when student switches steps (fresh context per step)
- "New session" button also resets history for the current step

---

## Things to Improve Over the Prototype

1. **Scrape and embed richer content** — Claude Code can `fetch` each tutorial URL and extract the actual step-by-step instructions, code samples, and YAML examples to make the context blocks much more detailed than what was hand-written.

2. **Streaming responses** — The prototype waits for the full response. Claude Code version should stream tokens using the Anthropic streaming API so students see output as it's generated.

3. **Markdown rendering** — The prototype has a basic formatter. Use a proper markdown renderer (marked.js from CDN) for clean code blocks, lists, and headers.

4. **Local storage for API key** — Optionally remember the key in localStorage so students don't re-enter it every session (with a clear warning/opt-in).

5. **Progress tracking** — Simple checkboxes in the sidebar that students can mark as they complete steps (stored in localStorage).

6. **Mobile layout** — Sidebar should collapse on small screens.

7. **Copy button on code blocks** — Students copy-paste a lot of code.

---

## Cost Estimate

- Model: `claude-sonnet-4-20250514`
- Input tokens per message: ~3,000–5,000 (system prompt + history)
- Output tokens per message: ~300–600
- Price (as of early 2026): ~$3/MTok input, ~$15/MTok output
- Estimated cost per full tutorial session (30–50 messages): **$0.05–0.20**
- Budget of $5/month covers roughly **25–100 sessions**

For a few students per month, $5 is very comfortable headroom.

---

## Deployment Options (All Free)

1. **GitHub Pages** — Push the HTML file to a repo, enable Pages. Students access via URL.
2. **Netlify drop** — Drag the HTML file onto netlify.com/drop. Instant URL, no account needed.
3. **Internal web server** — Drop it on any existing web server or shared drive.
4. **Just email the file** — It works opened directly from disk in a browser (no server needed).

---

## Suggested Claude Code Workflow

```bash
# 1. Create the project
mkdir zdk-training-assistant
cd zdk-training-assistant

# 2. Have Claude Code fetch all tutorial pages and build context blocks
# 3. Write index.html with full implementation
# 4. Test locally: open index.html in browser, enter API key, verify each step
# 5. Deploy to GitHub Pages or preferred host
```

Key prompt for Claude Code to get started:
> "Build a single self-contained HTML file called index.html. It's an AI-powered training assistant for the Zenoss ZenPack Development Kit. Fetch each of the tutorial URLs listed in the handoff doc to extract the actual content, then build a chat interface with sidebar navigation. Use the Anthropic API with claude-sonnet-4-20250514, streaming responses, and marked.js for markdown rendering. The system prompt for each step should include the real content scraped from that step's doc page."

---

## Contact / Context

- The original instructor who taught this class is no longer available
- Students are Zenoss customers learning to build custom ZenPacks
- They follow along with the tutorials locally on their own Zenoss dev server
- The main value of the original class was answering questions in real time as students hit errors
- This tool replaces that Q&A layer
