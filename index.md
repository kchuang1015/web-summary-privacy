# Privacy Policy

**"AI Web Summary & Translate" Chrome Extension**

Last updated: 2026-07-17

---

## Overview

"AI Web Summary & Translate" (the "Extension") is maintained by an independent developer. It helps you summarize and translate web pages using a large language model (LLM) endpoint that **you configure yourself**.

We take your privacy seriously. This policy explains what data the Extension processes, how, and what your rights are.

In one precise line: **No developer server, no telemetry. In cloud mode your content goes only to the AI endpoint you choose; built-in AI inference runs on your device; results are kept locally by default.** Exactly what each feature sends, whether it is automatic, and how many model calls it makes are laid out in the **[data-flow matrix (DATA_FLOW.md)](DATA_FLOW.md)**.

---

## No developer collection, no telemetry

The developer collects **no user data on any server** (note, however, that **your page content is sent to the AI endpoint YOU choose** — that is the nature of BYOK, which is different from "sends no data anywhere"). The Extension:

- Contains no trackers or analytics (Google Analytics, Mixpanel, etc.)
- Shows no ads
- Sends nothing to the developer
- No developer-operated server and no intermediary (content goes only to the AI endpoint you configure; with Chrome built-in AI it stays fully on-device)

---

## Data processed locally on your device

### 1. Settings

Everything you enter on the Settings page is stored only in your Chrome browser's storage:

- AI provider choice (AWS Bedrock or OpenAI-compatible)
- Bedrock API Key (if using Bedrock; v1.14 uses a Bearer API key instead of Access/Secret keys)
- AWS region, Bedrock model ID
- OpenAI-compatible endpoint URL, API key, model name
- Summary / translation prompt templates (including per-length custom versions)
- Keyboard shortcuts
- Translation color, bilingual display preference, translation scope
- Whether to include page images, result display mode (dialog / side panel)

**Where your API keys live is your choice:**

- **Default — "this device only"** (`chrome.storage.local`): API keys (Bedrock API Key and any OpenAI-compatible keys) **never leave this device** and are not synced through your Google account.
- If you uncheck "Keep API keys only on this device" under Settings → Key Storage, keys are stored in `chrome.storage.sync` and sync across your signed-in Chrome devices (**not additionally encrypted**).
- Other non-sensitive settings (model choice, prompts, shortcuts, etc.) always use `chrome.storage.sync` so they follow you across devices.

In either mode, none of this is ever sent to a developer-operated server or any intermediary (only to the endpoint YOU configured, and only when you use a cloud endpoint). We still recommend:

- Do not store high-privilege API credentials on public/shared computers
- Rotate API keys regularly
- Use least-privilege IAM policies (for Bedrock)

### 2. Page content

When you actively trigger "Summarize" or "Translate", the Extension sends the following to **the AI endpoint you configured yourself**:

- The visible text of the current tab
- **The full page URL and title** — the summary prompt includes the page URL and title. ⚠️ A full URL can contain access tokens in query parameters, signed-URL parameters, or a `#fragment`; these are sent along to the model endpoint you chose.
- (If you enabled "include images" and use a vision-capable model) page images. Image fetching first goes through the service worker without your cookies (`credentials: 'omit'`); only if the image host blocks that (hotlink protection / WAF) is it re-fetched once from the page context. Fetched images are converted to base64. Image URLs pass a safety check (blocking non-http(s), localhost, private and link-local IPs); this check is based on the literal hostname and **cannot defend against DNS rebinding** (a public domain maliciously resolving to an internal IP) — this is a known, accepted residual risk that cannot be fully fixed in a pure browser extension with no intermediary server. Security-sensitive users can uncheck "include images" to disable all image fetching entirely.

The developer has no access to this data and operates no intermediary server.

**After a summary, two small extra calls run automatically by default** (both to your same model endpoint):

- **Auto tags** (on by default, can be turned off): after the summary completes, it sends the **full summary (up to ~4,000 characters)** plus the title and your existing tag list to extract 3–6 topic tags.
- **Suggested questions** (on by default, can be turned off): it sends the **full summary (up to ~4,000 characters)** to generate 3 follow-up questions. **This call still fires even when the main summary is served from the local cache (not re-run).**

Topic pages, mind maps, and knowledge-base Q&A are **manually triggered** (see Section 3 below and [DATA_FLOW.md](DATA_FLOW.md)). Exactly what each feature sends, whether it is automatic, and the estimated number of model calls are laid out in the **[data-flow matrix (DATA_FLOW.md)](DATA_FLOW.md)**.

**YouTube summaries**: on a YouTube video page, the Extension reads the video's caption data from YouTube within that page (the primary path does not use your signed-in credentials; only if it fails, a fallback retries using your existing YouTube session within that page — all reads stay inside the YouTube page and nothing is ever sent to the developer); the caption text is then sent to the AI endpoint you configured (or processed fully on-device by the built-in AI).

**Chrome built-in AI mode**: if you choose "Chrome built-in AI" (Gemini Nano) in Settings, the **model inference for the content above runs entirely on your computer** and **page content is not sent to any external AI endpoint**. Note, however, that **fetching source images, reading YouTube captions, and the one-time download of the built-in model itself can still generate network requests** (to image CDNs, to YouTube, and to the browser's model-download service, respectively).

### 3. Local history & cache of summaries / translations

For review convenience and token savings, results are kept in **your browser's local** `chrome.storage.local`:

- **Recent results (`resultHistory`)**: summaries and selection results (including follow-up Q&A, topic tags, and — for article pages where paragraph citations apply automatically — short excerpts of the cited source paragraphs so the viewer can show where each claim came from; disable "Keep a local Recent results history" to stop storing them). The total size budget is adjustable in Settings (5–100 MB, default 20 MB); the oldest entries are evicted automatically when over budget. You can delete entries individually or clear everything in the viewer page. Topic tags are auto-generated by your configured AI endpoint by default (one extra tiny call after each summary; can be turned off in Settings; to reduce synonym tags, that call includes your existing tag list) and are editable; tags also stay **local only** and power the Knowledge Graph page. Each entry can individually be excluded from the knowledge graph.
- **Tag relations (`kgRelations`)**: **this feature is suspended as of v1.16** (taken down for insufficient accuracy; the code flag `REL_FEATURE=false` hides its buttons and list in the UI; it will return after improvements). Existing data and handlers are retained but cannot be triggered. It was originally a **manually triggered** feature that sent tag text only (no page content) to your own endpoint; no such call happens today.
- **Topic pages (`topicPages`)**: "Generate / update" and "Generate / update mind map" on a topic page are features **you trigger manually** — they send the saved summaries under that tag to your own configured AI endpoint to synthesize one wiki page or one multi-level mind map; the result is stored **locally only** (the wiki is editable), and the whole topic page (including its mind map) can be deleted. "Clear recent results & translation cache" also deletes tag relations and topic pages.
- **Translation cache (`translationCache`)**: translations of identical paragraphs (same model, same language) so re-reading a page costs nothing; LRU eviction with a size cap.

**Non-persistent session context (`chrome.storage.session`)**: so that you can keep asking follow-ups after the Service Worker is recycled, and the side panel can restore its view when reopened, the Extension mirrors two kinds of transient data to `chrome.storage.session`:

- **Follow-up conversation context (`conv_*`)**: the current tab's follow-up conversation (including page context).
- **Side-panel replay log (`panel_*`)**: the already-rendered result messages in side-panel display mode, so the panel can replay after a reopen / SW restart.

`chrome.storage.session` is **never synced, never written to disk, and never sent to the developer**, and it **disappears automatically when the tab / browser session ends** (navigating away or closing the tab also clears its entries).

**Your controls:**

- Settings → "Keep a local Recent results history" can **turn history off** (recommended for sensitive content such as intranet, medical, or financial pages).
- Settings → "Reuse recent results for repeated content" controls the **translation cache**.
- Settings → "Clear recent results & translation cache" deletes **history, cache, knowledge base (tag relations, topic pages), and conversation context** with **one click**; removing the Extension also deletes all local data. Transient `storage.session` context also disappears when the session ends.

All of this stays **on your device only** — never sent to a developer-operated server or intermediary (only when you actively use auto-tags / topic pages / knowledge Q&A does the relevant content go to the AI endpoint you chose).

---

## Third-party services

The Extension sends the page content described above to the AI endpoint you specify. You are responsible for understanding and agreeing to that endpoint's privacy policy. Common endpoints:

- **AWS Bedrock**: see the [AWS Service Terms](https://aws.amazon.com/service-terms/)
- **Official OpenAI API**: see the [OpenAI Privacy Policy](https://openai.com/privacy)
- **Self-hosted / local LLMs** (vLLM, Ollama, LM Studio, etc.): governed by your own deployment
- **Third-party OpenAI-compatible cloud services**: see that provider's privacy policy

The developer is not responsible for any third-party service's data handling.

---

## Permissions

The Extension requests the following Chrome permissions, each used only for its core features:

| Permission | Purpose |
|---|---|
| `storage` | Stores the settings above, plus "Recent results" and the translation cache (size budget adjustable in Settings: 5–100 MB, default 20 MB, auto-eviction; local `chrome.storage.local`, never synced, deletable any time in the viewer) |
| `scripting` | Injects the content script into the page to show results and translations |
| `contextMenus` | Adds the right-click "Summarize / Translate selection" items; runs only when you select text and click them |
| `sidePanel` | Optional display mode: summary results can be shown in the browser side panel (independent per tab). The panel only renders results you explicitly requested; it reads nothing by itself |
| `unlimitedStorage` | Lets you raise the local "Recent results" size budget (5–100 MB) in Settings. The data still lives **only on your device** — never synced or uploaded |
| `host_permissions: <all_urls>` | Lets the Extension work on any site you're reading, and lets the service worker fetch your configured AI endpoint and page-image CDNs (CORS bypass) |

To respond to keyboard shortcuts instantly, the content script loads on all pages, but it **only listens for the shortcuts you configured**; it never reads or uploads page content in the background — content is extracted only when you explicitly trigger a summary/translation.

---

## Your rights & controls

- **Remove at any time**: removing the Extension at `chrome://extensions` deletes all its settings data
- **View / change settings**: right-click the icon → Options
- **Clear data**: remove the Extension, or clear manually via Chrome DevTools → Application → Storage
- **Withdraw Google-account sync**: turn off sync in Chrome settings

---

## Children's privacy

The Extension is not designed for children under 13 and does not knowingly collect children's data.

---

## Policy changes

Material changes will be reflected on this page with an updated "Last updated" date. Please check back periodically.

---

## Contact

For any privacy questions:

- Email: kchuang1015@gmail.com

---

## Source code

The Extension's source code is not currently public. If it is open-sourced later, a link will be added here for review. If you have questions about how privacy is implemented, email us — we're happy to explain the technical details.
