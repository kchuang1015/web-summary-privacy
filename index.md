# Privacy Policy

**"WholeTake" Chrome Extension**

Last updated: 2026-09-01

---

## Overview

"WholeTake" (formerly "WholeTake"; the "Extension") is maintained by an independent developer. It helps you summarize and translate web pages using a large language model (LLM) endpoint that **you configure yourself**.

We take your privacy seriously. This policy explains what data the Extension processes, how, and what your rights are.

In one precise line: **No developer server, no telemetry. In cloud mode your content goes only to the AI endpoint you choose; built-in AI inference runs on your device; results are kept locally by default.** Exactly what each feature sends, whether it is automatic, and how many model calls it makes are laid out in the **data-flow matrix (DATA_FLOW.md)**.

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

- AI provider choice (Chrome built-in AI, AWS Bedrock, or OpenAI-compatible) — **Chrome built-in AI (Gemini Nano) is key-free and runs entirely on-device**: choosing it requires no API key, and model inference for summaries/translations makes no external requests (see "Chrome built-in AI mode" below for details)
- Bedrock API Key (if using Bedrock; v1.14 uses a Bearer API key instead of Access/Secret keys)
- AWS region, Bedrock model ID
- OpenAI-compatible endpoint URL, API key, model name
- Summary / translation prompt templates (including per-length custom versions)
- Translation color, bilingual display preference, translation scope
- Whether to include page images, result display mode (dialog / side panel)

The Extension's own pages also use `window.localStorage` (not `chrome.storage`) for **8 display
preferences only** — numbers, booleans, layout coordinates and your interface language. They
contain **no page content, no credentials and no personal data**, are never synced and never leave
your device:
`wt-theme` (light/dark/auto theme, used to avoid a first-paint flash), `kgAskPanelW` /
`kgAskFontPx` (knowledge-graph ask panel width and font size), `kgLegendOpen` / `kgLegendBox`
(legend collapsed state, window position and size), `spFontPx` (side panel font size),
`wt-uilang` (the interface language you chose and its text direction, again to avoid a
first-paint flash), and `wt-uilang-pack` (an offline copy of **the Extension's own** interface
text file for that language — literally the `_locales/<lang>/messages.json` that ships inside the
Extension, cached so the first frame can already be in the right language; it contains none of
your data).
To clear them: use your browser's "Clear browsing data -> Cookies and other site data", or remove
the Extension.

Keyboard shortcuts are **not** in that list: since v1.39 the Extension has no in-page
shortcut handling of its own, and the three browser-level shortcuts are managed by
Chrome's own **Keyboard shortcuts** page (`chrome://extensions/shortcuts`) and stored
by Chrome, not by the Extension.

**Where your API keys live is your choice:**

Keys have **three** storage modes, chosen under Settings → Key Storage:

- **Default — "this device only"** (`chrome.storage.local`): API keys (Bedrock API Key and any OpenAI-compatible keys) **never leave this device** and are not synced through your Google account.
- **"Keep keys only until the browser closes"** (`sessionCreds` in `chrome.storage.session`, added in v1.37): keys are **never written to disk and never synced**, and are cleared automatically when the browser closes — you will need to re-enter them next time. Recommended for shared computers and high-sensitivity environments. Requires "Keep API keys only on this device" to be checked first.
  A note so this is not oversold: this is **not a password vault** — while the browser is open, the Extension's trusted contexts (background service worker, Settings page) still read it in order to call the AI endpoint you configured.
- If you uncheck "Keep API keys only on this device" under Settings → Key Storage, keys are stored in `chrome.storage.sync` and sync across your signed-in Chrome devices (**not additionally encrypted**).
- Other non-sensitive settings (model choice, prompts, etc.) always use `chrome.storage.sync` so they follow you across devices.

Whichever mode you choose, none of this is ever sent to a developer-operated server or any intermediary (only to the endpoint YOU configured, and only when you use a cloud endpoint). We still recommend:

- Do not store high-privilege API credentials on public/shared computers
- Rotate API keys regularly
- Use least-privilege IAM policies (for Bedrock)

### 2. Page content

When you actively trigger "Summarize" or "Translate", the Extension sends the following to **the AI endpoint you configured yourself**:

- The **article body text** of the current tab (the primary extraction path checks actual rendering per paragraph and skips invisible ones; the Readability fallback path removes top-level CSS-hidden and `hidden`/`inert` blocks before parsing — small amounts of text hidden by deeply nested stylesheet rules may still be included, so we do not claim it is strictly identical to "what is visible on screen")
- **The page URL (sanitized) and title** — the URL placed in the summary prompt is **only `origin+pathname`**: before sending, the Extension **strips the query string (including access tokens / signed-URL parameters), the `#fragment`, and any userinfo credentials** (see `sanitizeUrlForModel` in `src/defaults.js`). The full original URL is kept **only on your device** (history and cache matching) and is never sent out.
- (If you enabled "include images" and use a vision-capable model) page images. Image fetching first goes through the service worker without your cookies (`credentials: 'omit'`); only if the image host blocks that (hotlink protection / WAF) is it re-fetched once from the page context. Fetched images are converted to base64. Image URLs pass a safety check (blocking non-http(s), localhost, private and link-local IPs); this check is based on the literal hostname and **cannot defend against DNS rebinding** (a public domain maliciously resolving to an internal IP) — this is a known, accepted residual risk that cannot be fully fixed in a pure browser extension with no intermediary server. Security-sensitive users can uncheck "include images" to disable all image fetching entirely.

The developer has no access to this data and operates no intermediary server.

**What happens with unencrypted (`http://`) endpoints** (as of v1.44): your content only ever goes
to the endpoint you configured yourself, but sending it in the clear is a risk in its own right, so
the Extension checks twice — once when you save the setting, and again immediately before every
request it sends:

- **`https://`** — always allowed.
- **Local loopback** (`localhost`, `127.0.0.1`, `[::1]`) — allowed: the traffic never leaves your
  computer (the default Ollama / LM Studio setup).
- **An IP address on your own LAN** (e.g. `http://192.168.1.50:11434`) — **refused by default**.
  To use one you must explicitly opt in under "Settings → Model settings → Advanced (risk options)",
  because other devices on the same network can sniff plaintext traffic.
- **A LAN endpoint addressed by hostname** (`http://ollama.local`, `http://nas:11434`) — content
  follows the rule above; but **if that endpoint has an API key, it is refused outright and no
  setting can allow it** — such names carry no authentication, so any device on the same network
  can claim the name and receive your key. Use the machine's LAN IP address instead, or `https://`.
- **An `http://` endpoint on the public internet** — **always refused, with or without a key, and
  there is no setting that turns this off.** (Before v1.44 an internal field allowed this case; it
  has been removed entirely.)

The technical criteria and where they are implemented are in the "endpoint transport policy" table
in SECURITY.md.

**Automatic calls beyond the summary itself** (on by default, all to your same model endpoint, each can be turned off in Settings):

- **Auto tags**: after the summary completes, one extra small call sends the **summary excerpt (up to ~4,000 characters)** plus the title and your existing tag list to extract 3–6 topic tags. When the main summary is served from cache and that record already has tags, this call is not re-run; an older record missing tags gets a one-time backfill call whose result is saved back (a failed backfill is remembered, so it is not re-billed on every cache hit).
- **Suggested questions**: since v1.35, on cloud models the 3 follow-up questions are produced **in the same single summary call** — no separate model call is made and no additional content is sent for them. A separate small call sending the **summary excerpt (up to ~4,000 characters)** happens only when the question section of the summary output fails to parse (one automatic fallback call) or with Chrome built-in AI (which deliberately keeps the two-stage flow). The questions are saved **on your device** alongside that summary; **when the main summary is served from the local cache, the saved questions are reused directly — no extra model call** (an older record missing questions gets a one-time backfill call whose result is saved back).

Topic pages, mind maps, and knowledge-base Q&A are **manually triggered** (see Section 3 below and DATA_FLOW.md). Exactly what each feature sends, whether it is automatic, and the estimated number of model calls are laid out in the **data-flow matrix (DATA_FLOW.md)**.

**YouTube summaries**: on a YouTube video page, the Extension reads the video's caption data from YouTube within that page (the primary path does not use your signed-in credentials; only if it fails, a fallback retries using your existing YouTube session within that page — all reads stay inside the YouTube page and nothing is ever sent to the developer); the caption text is then sent to the AI endpoint you configured (or processed fully on-device by the built-in AI).

**Chrome built-in AI mode**: if you choose "Chrome built-in AI" (Gemini Nano) in Settings, the **model inference for the content above runs entirely on your computer** and **page content is not sent to any external AI endpoint**. Note, however, that **fetching source images, reading YouTube captions, and the one-time download of the built-in model itself can still generate network requests** (to image CDNs, to YouTube, and to the browser's model-download service, respectively).

**Where results are shown, and what the web page can see**: the summary result, streaming output, and your follow-up input are carried, in side-panel mode, by an **extension-owned page** (extension origin); the on-page popup **by default** carries its content in an **extension-owned iframe**, so scripts on the page you're browsing **cannot read or alter** the results or your input inside it — and the iframe handshake requires a **one-time nonce** obtained through the extension's internal channel, which the host page cannot obtain or hijack. If the iframe is removed by the host page **or fails to load for any reason**, the default behavior is **fail closed**: the Extension closes the window and cancels the task — content is **never** handed back to the page automatically. In-page rendering happens **only if you have explicitly opted in beforehand** via the "Compatibility mode" consent setting; with that consent, a **benign load failure** (e.g. iframe load timeout) falls back to in-page rendering clearly labeled "**Compatibility mode**" — in that mode the content lives in the host page's DOM and is readable by page scripts, so use side-panel mode for sensitive content. See SECURITY.md for the full trust-boundary description.

### 3. Local history & cache of summaries / translations

For review convenience and token savings, results are kept in **your browser's local** `chrome.storage.local`:

- **Recent results (local history)**: summaries and selection results — each entry contains the **page title, the full original URL (query string included; kept local only, never sent out), a timestamp, the full summary/translation text, topic tags, and (when saved) follow-up Q&A**; for article pages where paragraph citations apply automatically, short excerpts of the cited source paragraphs are also kept so the viewer can show where each claim came from; for YouTube summaries, the citation time-range mapping (`citeTimes`) is also kept — stored locally only, used for jump-to-segment in the viewer; for full-page translation a **lightweight record** is also saved (page title, URL, and the number of translated paragraphs — the translated text itself lives only in the translation cache and on the original page, no full text is stored here), while selection translation saves the selected source text and its translation; disable "Keep a local Recent results history" to stop storing them. As of v1.20 this history is stored locally in an "index + per-record shard" layout (a purely internal storage-layout change — what is stored, its local-only nature, and your deletion/clearing controls are unchanged; the one-time migration backup **expires automatically after 7 days**, and **deleting an individual entry also removes it from that backup**, so "deleted means unrecoverable" is never undermined by the backup). The total size budget is adjustable in Settings (5 MB–1 GB tiers, default 20 MB; upper bound raised in v1.22); the oldest entries are evicted automatically when over budget. You can delete entries individually or clear everything in the viewer page. Topic tags are auto-generated by your configured AI endpoint by default (one extra tiny call after each summary; can be turned off in Settings; to reduce synonym tags, that call includes your existing tag list) and are editable; tags also stay **local only** and power the Knowledge Graph page. Each entry can individually be excluded from the knowledge graph.
- **Tag relations (`kgRelations`)**: **this feature is suspended as of v1.16** (taken down for insufficient accuracy; the code flag `REL_FEATURE=false` hides its buttons and list in the UI; it will return after improvements). Existing data and handlers are retained but cannot be triggered. It was originally a **manually triggered** feature that sent tag text only (no page content) to your own endpoint; no such call happens today.
- **Topic pages (`topicPages`)**: "Generate / update" and "Generate / update mind map" on a topic page are features **you trigger manually** — they send the saved summaries under that tag to your own configured AI endpoint to synthesize one wiki page or one multi-level mind map; the result is stored **locally only** (the wiki is editable), and the whole topic page (including its mind map) can be deleted. "Clear recent results & translation cache" also deletes tag relations and topic pages.
- **Translation cache (`translationCache`)**: translations of identical paragraphs (same model, same language) so re-reading a page costs nothing; LRU eviction with a size cap.
- **Knowledge-base export / import (v1.21)**: under "Settings → Knowledge base → Backup & migration" you can export your knowledge base as a **JSON backup** (saved wherever you choose; when the total exceeds **40 MB** it is automatically split into **multiple volumes**, each a self-contained file that can be imported on its own and merged volume by volume). It contains: all recent-results records (including Q&A, tags, mind maps), topic pages, tag relations, and a **non-sensitive settings subset** (custom prompts and display preferences) — it **never contains any API key** (the export uses a field allowlist, so credential fields structurally cannot enter the file). The whole operation is **pure local file I/O — nothing is sent to any model or anywhere else**. The export file contains your history summaries in plain text, so store it safely; on import you can choose "merge" (keep the newer version of each record) or "replace" (swap your whole knowledge base for the contents of the backup file). **"Replace" does not wipe first and write afterwards.** The imported records are written to a staging area and read back for verification first; only once everything is proven writable is the new index switched in by a **single atomic write**. Deleting the old data (records not present in the backup, topic pages, tag relations) happens strictly *after* that switch. So **if any step fails before the switch, everything is rolled back and your existing knowledge base is left untouched**; if the process is interrupted *after* the switch (for example by closing the browser), the new knowledge base is already usable and the remaining cleanup is completed automatically from a recovery journal on the next start. One thing to be clear about: once "replace" succeeds, records that are not in the backup file are deleted by design — that is what "replace" means, not a failure.

**Non-persistent session context (`chrome.storage.session`)**: so that you can keep asking follow-ups after the Service Worker is recycled, and the side panel can restore its view when reopened, the Extension keeps **seven** kinds of data in `chrome.storage.session`:

- **Follow-up conversation context (`conv_*`)**: the current tab's follow-up conversation (including page context and a snapshot of the model settings). **As of v1.23 this snapshot always has every API-key field stripped out** (older snapshots written by earlier versions are sanitized in place on load) — **the follow-up context snapshot never contains a key**; follow-up calls re-resolve credentials from your current live settings instead. (Note: if you opt into the "keep keys only until the browser closes" mode described above, your keys are stored in a separate item, `sessionCreds`, in this same storage area — that is a storage location you chose deliberately, and is distinct from this conversation snapshot.)
- **Side-panel replay log (`panel_*`)**: the already-rendered result messages in side-panel display mode, so the panel can replay after a reopen / SW restart.
- **API keys in "keep keys only until the browser closes" mode (`sessionCreds`)**: present **only** if you chose that mode under Settings → Key Storage (see "Where your API keys live is your choice" above); never written to disk, never synced, and cleared automatically when the browser closes. This is a separate item from the conversation snapshot above.
- **Running-task snapshot (`bgActiveTasks`)**: which kinds of task are running in which tab and when each one started — **task kind plus a timestamp only; no page content and no URL**. It exists so the toolbar popup can show queue state without waking the Service Worker, and so running tasks survive an unexpected Service Worker restart. Removed as soon as no tasks remain.
- **Translation round bookkeeping (`trrun_*`)**: per-tab state that lets a "fill in the gaps" round know what the previous round already delivered. It holds **content hashes, counters, short flags, and the URL of the page being translated** — and **zero characters of the page text, zero characters of the translation** (the value that does hold translated text is deliberately not mirrored here). It also carries a data epoch, so clearing the translation cache makes any leftover entry expire immediately.
- **Last translation outcome (`bgLastRunOutcome`)**: for each tab, how the most recent full-page translation ended — outcome kind, paragraph counts, a run id, the status/error card text, and, so that a late result can be re-delivered to the right page and never to a different one, **the URL of the page that was being translated**. At most 20 entries. That URL is kept **solely** for this identity check, is never sent anywhere, and disappears with the browser session like everything else in this storage area.
- **Unsaved topic-page draft (`topicDraft_*`)**: while you are editing a topic page's wiki text, the in-progress draft is kept per tag so that an accidental reload or a Service Worker restart does not lose what you typed. It holds **the draft text you wrote**, nothing else; it is removed as soon as the draft is saved or discarded, and — like everything in this storage area — it is never synced, never written to disk, never sent anywhere, and gone when the browser session ends.

`chrome.storage.session` is **never synced, never written to disk, and never sent to the developer**, and it **disappears automatically when the tab / browser session ends** (navigating away or closing the tab also clears its entries).

**Your controls:**

- Settings → "Keep a local Recent results history" can **turn history off** (recommended for sensitive content such as intranet, medical, or financial pages).
- Settings → "Reuse recent results for repeated content" controls the **translation cache**.
- The "Recent results" viewer page lets you **delete entries individually** or **clear everything** (you can export a backup first); as of v1.23 the viewer's "Clear all" also **actively removes the follow-up conversation context (`conv_*`)** rather than passively waiting for the session to end.
- Settings → "Clear recent results & translation cache" deletes **history, cache, knowledge base (tag relations, topic pages), and conversation context** with **one click**; removing the Extension also deletes all local data. Transient `storage.session` context also disappears when the session ends.
- **Clearing is complete (v1.21, scoped)**: before any clear operation runs, the Extension **first aborts the in-flight tasks that correspond to what you are clearing** — "clear history / clear everything" aborts summaries and translations in every tab plus long-running knowledge-base tasks; "clear translation cache only" aborts only translation tasks (a paid summary running in another tab is unaffected); "clear knowledge-base data only" aborts only the long-running knowledge-base task — and advances the matching internal **data-generation gate**: a task still finishing at the moment you clear **will not write its data back**, so cleared data never "comes back to life".

All of this stays **on your device only** — never sent to a developer-operated server or intermediary (only when you actively use auto-tags / topic pages / knowledge Q&A does the relevant content go to the AI endpoint you chose).

---

## Third-party services

The Extension sends the page content described above to the AI endpoint you specify. You are responsible for understanding and agreeing to that endpoint's privacy policy. Common endpoints:

- **Chrome built-in AI (Gemini Nano)**: **not a third-party cloud service** — key-free, model inference runs entirely on your device, and summarizing/translating **generates no external request for inference** (only the one-time download of the model itself is performed by the browser from Google's model-download service; your content is never part of it)
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
| `storage` | Stores the settings above, plus "Recent results" and the translation cache (size budget adjustable in Settings: 5 MB–1 GB, default 20 MB, auto-eviction; local `chrome.storage.local`, never synced, deletable any time in the viewer) |
| `scripting` | Injects the content script into the page to show results and translations |
| `contextMenus` | Adds the right-click "Summarize / Translate selection" items; runs only when you select text and click them |
| `sidePanel` | Display mode (default since v1.28): summary results are shown in the browser side panel, **only on tabs where you triggered a summary** (per-tab content; the panel does not follow you to other tabs); you can switch back to the in-page dialog in Settings. The panel only renders results you explicitly requested; it reads nothing by itself |
| `unlimitedStorage` | Lets you raise the local "Recent results" size budget (5 MB–1 GB) in Settings. The data still lives **only on your device** — never synced or uploaded |
| `host_permissions: <all_urls>` | Lets the Extension work on any site you're reading, and lets the service worker fetch your configured AI endpoint and page-image CDNs (CORS bypass) |
| `declarativeNetRequestWithHostAccess` | Used ONLY to strip the `Origin` header from requests the Extension itself sends to your own configured local AI endpoint (loopback such as Ollama / LM Studio). Local servers reject the browser-added `Origin: chrome-extension://<id>` with a CORS 403; removing it lets your own local model accept the request. The rule is scoped by `initiatorDomains` to THIS extension's initiator only (never any web page), covers loopback automatically and LAN only after you explicitly opt in, and NEVER public endpoints — one `modifyHeaders(remove Origin)` rule per configured local endpoint origin (typically one, at most four - deduplicated across the main/summarize/translate/knowledge endpoints), and nothing else: it does not block, redirect, read ordinary web-page traffic, or read/modify cookies |

The content script is injected **on demand** (since v1.37; zero resident scripts since v1.39): **no code from this extension is resident on any web page** — the manifest declares no `content_scripts`. The full script loads into the current tab only when you trigger a summary/translation (keyboard shortcut, context menu, or popup). Content is extracted only when you explicitly trigger it, never read or uploaded in the background.

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

- Email: support@marlubie.com
- Website: https://wholetake.marlubie.com/

---

## Source code

The Extension's source code is not currently public. If it is open-sourced later, a link will be added here for review. If you have questions about how privacy is implemented, email us — we're happy to explain the technical details.
