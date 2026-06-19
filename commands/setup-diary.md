Set up the self-contained "working diary" system in the current repo: a protocol in `CLAUDE.md` that tells you (Claude Code) to log every task, a `tasks.js` data file you maintain, and a no-server `diary.html` the human opens by double-clicking. Triggers: "/setup-diary", "set up the working diary", "add the diary to this repo", "codify my work log".

## What this creates

These files in the repo root:

- **`CLAUDE.md`** — gains a "Working diary protocol" section instructing future sessions to log tasks.
- **`tasks.js`** — the data file: sets `window.DIARY_PROJECT` (the repo name) and `window.DIARY_TASKS` (the task array). This is the file you append to as you work.
- **`diary.html`** — a dependency-free viewer that loads `tasks.js` via a `<script>` tag. Shows the project name, an auto-refresh, status filters, and per-task cards.
- **`diary-serve.sh`** — a one-line helper that serves the folder over `http://127.0.0.1` so the viewer works on a remote/SSH host (e.g. a cluster + VS Code Remote). See **Viewing** below for why this is needed.

## Safety / idempotency rules (read before writing anything)

This command is safe to re-run. Honor these:

1. **Never clobber an existing `tasks.js`.** It holds the diary history. If `tasks.js` already exists, leave it untouched (do NOT recreate it), but DO ensure its first line sets `window.DIARY_PROJECT` to the detected name — add that line if missing. Report that existing data was preserved.
2. **Append the `CLAUDE.md` protocol only once.** If `CLAUDE.md` already contains `## Working diary protocol`, do not append it again. If `CLAUDE.md` is absent, create it with the block.
3. **`diary.html` is the canonical viewer** — overwrite it with the version below (it carries no user data). If it already existed, mention that it was refreshed to the latest viewer.
4. **Timestamps come from the system clock** (`date +%Y-%m-%dT%H:%M:%S`), never from memory.

## Steps

1. **Detect the project name.** Prefer the GitHub repo name, fall back to the directory:
   ```bash
   name="$(git remote get-url origin 2>/dev/null | sed -E 's#.*/##; s#\.git$##')"; [ -z "$name" ] && name="$(basename "$PWD")"; echo "$name"
   ```
   Use this value wherever `__PROJECT__` appears below.

2. **`CLAUDE.md`** — if it lacks `## Working diary protocol`, append the *CLAUDE.md protocol block* below verbatim (create the file if absent).

3. **`tasks.js`** — only if it does NOT already exist, create it from the *tasks.js* template below, replacing `__PROJECT__` with the detected name. If it already exists, just make sure its `window.DIARY_PROJECT` line names the detected project (add/fix that one line; leave `window.DIARY_TASKS` alone).

4. **`diary.html`** — write the *diary.html* content below verbatim (overwrite if present).

5. **`diary-serve.sh`** — write the *diary-serve.sh* helper below verbatim (overwrite if present) and make it executable (`chmod +x diary-serve.sh`).

6. **Validate `tasks.js`.** Run `node --check tasks.js`. If `node` is a Bun shim (the check errors with `ReferenceError: window is not defined` on a valid file because Bun executes rather than parses), use `bun build --no-bundle tasks.js` instead, which parses without running. Confirm it passes.

7. **Seed the first entry** — only if you just created `tasks.js` (skip if it already had data). Following the protocol, add a `t-001` task object: `task` "Set up the working diary system", `status` "done", real `started`/`finished` from `date`, `files` `["CLAUDE.md", "tasks.js", "diary.html", "diary-serve.sh"]`, a `summary`, `outcome` "success", a `notes` line, and `tags` `["setup", "tooling"]`. Re-validate `tasks.js` after.

8. **Report**: list what was created vs. preserved, and give the human the **Viewing** guidance below (this is the part people get wrong — surface it, don't bury it).

## Viewing the diary

`diary.html` is driven entirely by JavaScript: it injects a `<script src="tasks.js">` at runtime and builds the page from it. That means **a CSP-sandboxed webview preview will show a blank shell** — the inline script never runs. So:

- **Do NOT** use a webview "HTML Preview" extension (e.g. `george-alisson.html-preview-vscode`). Its strict Content-Security-Policy blocks the inline script and the dynamic `tasks.js` load, so you get an empty page. This is the #1 cause of "it doesn't load".
- **Local machine:** just double-click `diary.html` — the code special-cases `file://` and works straight from disk.
- **Remote / SSH host (e.g. a cluster, working on a network filesystem like lustre):** the file is not on your laptop, so `file://` and webviews won't help. Serve it over HTTP and let the editor forward the port:
  ```bash
  ./diary-serve.sh            # picks a free port (stable per repo) and prints the URL
  ```
  The script chooses a port derived from this folder's path — so several repos served at once never collide and each repo keeps the same port across restarts — then prints `http://localhost:<port>/diary.html`. Open that in your local browser. VS Code Remote-SSH auto-forwards listened ports; if not, open the **PORTS** tab → Forward a Port → the printed port. Served over `http://localhost` it is same-origin, so `tasks.js` loads and the scripts run regardless of where the file physically lives. Pass an explicit port (`./diary-serve.sh 9001`) to override. (`ms-vscode.live-server` also works, but only if the repo is a workspace folder so it can resolve `tasks.js`; the standalone server roots itself in the folder and avoids that pitfall.)

Note: because `diary.html` is parsed once when the page opens, editing the HTML later needs a full page reload (F5); `tasks.js` data changes are picked up by the in-page reload button / auto-refresh without reopening.

═══════════════════════════════════════════════════════
CLAUDE.md protocol block — append verbatim (it is fine for a `[[name]]`-free plain section)
═══════════════════════════════════════════════════════

~~~~markdown
## Working diary protocol

Keep a running diary of your work in `tasks.js`. Treat it as your lab notebook: log what you are about to do, then reflect and record the result when you finish. A human reads this through `diary.html`, so write for a colleague who will read it later.

**Work in tasks.** A task is one unit of work with a clear goal: a bugfix, a refactor, an investigation, a feature. If you are doing something substantial with no open task, open one first. Do not log trivial actions (reading a file, listing a dir).

**Before starting**, add an object to the `window.DIARY_TASKS` array in `tasks.js`:
- `id`: next free `t-NNN` (increment the highest in the file).
- `task`: imperative one-liner.
- `status`: `"in_progress"`. `finished`: `null`. `outcome`: `"pending"`. Other fields empty.
- `started`: real local datetime, ISO `YYYY-MM-DDTHH:MM:SS`, from `date +%Y-%m-%dT%H:%M:%S`. Never guess the time.

**When finished**, edit that same object in place (match on `id`, never add a duplicate):
- `status`: `"done"`. `finished`: current datetime from `date`.
- `files`: every file created or changed, repo-relative.
- `summary`: 2 to 4 sentences on what changed and WHY, understandable without the diff.
- `outcome`: honest self-assessment, one of `"success"`, `"partial"`, `"blocked"`, `"failed"`.
- `notes`: a one or two line mini-retrospective: caveats, surprises, follow-ups, why this approach, what to check on regression. Highest-value field; do not leave empty on a done task.
- `tags` (optional): a short array of lowercase labels (e.g. `["bugfix", "perf"]`). The viewer renders them as chips; omit or leave `[]` if none.

**Rules:**
- One object per task; updating means editing the existing object.
- `tasks.js` must stay valid at all times. After editing, syntax-check it: run `node --check tasks.js`. (On a machine where `node` is a Bun shim, `node --check` *executes* the file and errors on `window is not defined` — there, use `bun build --no-bundle tasks.js` instead, which parses without running.)
- Timestamps come from the system clock, never memory.
- Be honest in `outcome`. A logged `failed` with a clear note beats a quiet false `success`.
~~~~

═══════════════════════════════════════════════════════
tasks.js — create only if absent; replace __PROJECT__ with the detected name
═══════════════════════════════════════════════════════

~~~~js
// Working diary data. Maintained by Claude Code per CLAUDE.md.
// Plain JS file so the viewer opens straight from disk (no server).
window.DIARY_PROJECT = "__PROJECT__";
window.DIARY_TASKS = [];
~~~~

═══════════════════════════════════════════════════════
diary.html — write verbatim
═══════════════════════════════════════════════════════

~~~~html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>working diary</title>
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Crect width='32' height='32' rx='7' fill='%2311140f'/%3E%3Cpath fill='%23d97757' d='M16 4c1.1 7.4 4.5 10.9 11 12-6.5 1.1-9.9 4.6-11 12-1.1-7.4-4.5-10.9-11-12 6.5-1.1 9.9-4.6 11-12z'/%3E%3C/svg%3E">
<style>
  :root {
    --bg: #11140f; --panel: #1a1e16; --panel-hover: #20251b;
    --ink: #e8e6dd; --ink-dim: #8c907f; --rule: #2c3225;
    --accent: #b9d762; --done: #b9d762; --progress: #e0a755;
    --error: #d96c5f; --blocked: #c98bd0;
    --mono: ui-monospace, "SF Mono", "JetBrains Mono", "Cascadia Code", Menlo, monospace;
  }
  * { box-sizing: border-box; }
  body { margin: 0; background: var(--bg); color: var(--ink); font-family: var(--mono); font-size: 14px; line-height: 1.5; -webkit-font-smoothing: antialiased; }
  .wrap { max-width: 840px; margin: 0 auto; padding: 48px 20px 96px; }
  header { margin-bottom: 30px; }
  .prompt { color: var(--accent); }
  h1 { font-size: 15px; font-weight: 600; letter-spacing: 0.02em; margin: 0; display: inline; }
  .project { color: var(--ink-dim); font-size: 13px; }
  .project::before { content: "— "; }
  .project b { color: var(--accent); font-weight: 600; }
  .meta-line { color: var(--ink-dim); font-size: 12.5px; margin-top: 10px; }
  .meta-line b { color: var(--ink); font-weight: 600; }
  .bar { display: flex; gap: 16px; align-items: center; margin: 22px 0 4px; flex-wrap: wrap; }
  .filters { display: flex; gap: 8px; flex-wrap: wrap; }
  .filters button, .refresh button {
    font-family: var(--mono); font-size: 12px; color: var(--ink-dim);
    background: transparent; border: 1px solid var(--rule); border-radius: 2px; padding: 4px 10px; cursor: pointer;
  }
  .filters button[aria-pressed="true"] { color: var(--ink); border-color: var(--ink-dim); }
  .filters button:focus-visible, .refresh button:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
  .refresh { margin-left: auto; display: flex; gap: 8px; align-items: center; color: var(--ink-dim); font-size: 12px; }
  .card { border: 1px solid var(--rule); border-radius: 3px; margin-top: 10px; background: var(--panel); overflow: hidden; }
  summary { list-style: none; }
  summary::-webkit-details-marker { display: none; }
  .card-head { display: grid; grid-template-columns: 14px 1fr auto; gap: 12px; align-items: baseline; padding: 13px 16px; cursor: pointer; width: 100%; text-align: left; background: none; border: none; color: inherit; font: inherit; }
  .card-head:hover { background: var(--panel-hover); }
  .card-head:focus-visible { outline: 2px solid var(--accent); outline-offset: -2px; }
  .chev { color: var(--ink-dim); transition: transform .15s ease; line-height: 1.4; }
  .card[open] .chev { transform: rotate(90deg); }
  .task-title { font-weight: 600; }
  .when { color: var(--ink-dim); font-size: 12px; white-space: nowrap; }
  .dot { display: inline-block; width: 7px; height: 7px; border-radius: 50%; margin-right: 7px; vertical-align: middle; }
  .s-done .dot { background: var(--done); }
  .s-in_progress .dot { background: var(--progress); }
  .o-partial .dot { background: var(--progress); }
  .o-blocked .dot { background: var(--blocked); }
  .o-failed .dot { background: var(--error); }
  .card-body { padding: 0 16px 18px 42px; display: none; }
  .card[open] .card-body { display: block; }
  .status-row { font-size: 11px; letter-spacing: 0.04em; text-transform: uppercase; color: var(--ink-dim); margin-bottom: 12px; }
  .outcome { padding: 1px 7px; border-radius: 2px; border: 1px solid var(--rule); margin-left: 4px; }
  .outcome.success { color: var(--done); border-color: var(--done); }
  .outcome.partial { color: var(--progress); border-color: var(--progress); }
  .outcome.blocked { color: var(--blocked); border-color: var(--blocked); }
  .outcome.failed { color: var(--error); border-color: var(--error); }
  .outcome.pending { color: var(--ink-dim); }
  .summary { color: var(--ink); margin: 0 0 14px; }
  .field-label { color: var(--ink-dim); font-size: 11px; letter-spacing: 0.04em; text-transform: uppercase; margin-bottom: 5px; }
  .notes { color: var(--ink); background: var(--bg); border-left: 2px solid var(--rule); padding: 8px 12px; margin: 0 0 14px; font-size: 13px; }
  .files { list-style: none; padding: 0; margin: 0 0 14px; }
  .files li { color: var(--accent); font-size: 13px; padding: 1px 0; }
  .tags { display: flex; gap: 6px; flex-wrap: wrap; }
  .tag { font-size: 11px; color: var(--ink-dim); border: 1px solid var(--rule); border-radius: 2px; padding: 1px 7px; }
  .empty { color: var(--ink-dim); padding: 40px 0; text-align: center; }
  @media (prefers-reduced-motion: reduce) { .chev { transition: none; } }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <div><span class="prompt">$ </span><h1>working diary</h1><span class="project" id="project"></span></div>
    <div class="meta-line" id="meta">loading…</div>
    <div class="bar">
      <div class="filters" id="filters" role="group" aria-label="Filter by status"></div>
      <div class="refresh">
        <label><input type="checkbox" id="auto"> auto</label>
        <button id="reload">reload</button>
        <span id="stamp"></span>
      </div>
    </div>
  </header>
  <main id="list"></main>
</div>
<script>
let entries = [];
let activeFilter = "all";
let timer = null;
let lastJson = null;
const pad = n => String(n).padStart(2, "0");
function fmtDate(iso) { if (!iso) return "—"; return new Date(iso).toLocaleDateString(undefined, { year: "numeric", month: "short", day: "numeric" }); }
function fmtDateTime(iso) { if (!iso) return "—"; return new Date(iso).toLocaleString(undefined, { month: "short", day: "numeric", hour: "2-digit", minute: "2-digit" }); }
function fmtDuration(a, b) {
  if (!a || !b) return null;
  const mins = Math.round((new Date(b) - new Date(a)) / 60000);
  if (!Number.isFinite(mins) || mins < 0) return null;
  if (mins < 1) return "<1 min";
  if (mins < 60) return mins + " min";
  return Math.floor(mins / 60) + "h " + pad(mins % 60) + "m";
}
function statusText(s) { return ({ done: "done", in_progress: "in progress" })[s] || (s ? String(s) : "unknown"); }
function esc(str) { return String(str ?? "").replace(/[&<>"']/g, c => ({ "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;" }[c])); }
function render() {
  const list = document.getElementById("list");
  const shown = entries.filter(e => activeFilter === "all" || e.status === activeFilter);
  if (!shown.length) {
    list.innerHTML = '<div class="empty">No entries' + (activeFilter !== "all" ? " with this status." : " yet. Claude Code will write them to tasks.js.") + '</div>';
    return;
  }
  const openIds = new Set([...list.querySelectorAll("details[open]")].map(d => d.dataset.id));
  shown.sort((a, b) => new Date(b.finished || b.started || 0) - new Date(a.finished || a.started || 0));
  list.innerHTML = shown.map(e => {
    const isDone = e.status === "done";
    const when = isDone ? (e.finished || e.started) : e.started;
    const day = isDone ? fmtDate(when) : fmtDateTime(when);
    const dur = fmtDuration(e.started, e.finished);
    const dateLine = isDone ? `completed ${day}${dur ? " · " + dur : ""}` : `started ${day}`;
    const outcome = e.outcome ? `<span class="outcome ${esc(e.outcome)}">${esc(e.outcome)}</span>` : "";
    const files = (e.files || []).map(f => `<li>${esc(f)}</li>`).join("");
    const tags = (e.tags || []).map(t => `<span class="tag">${esc(t)}</span>`).join("");
    return `
    <details class="card s-${esc(e.status)} o-${esc(e.outcome || "")}" data-id="${esc(e.id)}">
      <summary class="card-head">
        <span class="chev">▸</span>
        <span class="task-title">${esc(e.task)}</span>
        <span class="when">${day}</span>
      </summary>
      <div class="card-body">
        <div class="status-row"><span class="dot"></span>${esc(statusText(e.status))} · ${dateLine} ${outcome}</div>
        ${e.summary ? `<p class="summary">${esc(e.summary)}</p>` : ""}
        ${e.notes ? `<div class="field-label">Notes</div><div class="notes">${esc(e.notes)}</div>` : ""}
        ${files ? `<div class="field-label">Files touched</div><ul class="files">${files}</ul>` : ""}
        ${tags ? `<div class="tags">${tags}</div>` : ""}
      </div>
    </details>`;
  }).join("");
  list.querySelectorAll("details").forEach(d => { if (openIds.has(d.dataset.id)) d.open = true; });
}
function renderFilters() {
  const counts = { all: entries.length };
  entries.forEach(e => counts[e.status] = (counts[e.status] || 0) + 1);
  if (activeFilter !== "all" && !counts[activeFilter]) activeFilter = "all";
  const order = ["all", "in_progress", "done"];
  document.getElementById("filters").innerHTML = order
    .filter(k => k === "all" || counts[k])
    .map(k => `<button data-f="${k}" aria-pressed="${k === activeFilter}">${k === "all" ? "all" : statusText(k)} ${counts[k] || 0}</button>`)
    .join("");
}
function stamp() {
  document.getElementById("stamp").textContent = "updated " + new Date().toLocaleTimeString(undefined, { hour: "2-digit", minute: "2-digit", second: "2-digit" });
}
function paint() {
  const done = entries.filter(e => e.status === "done").length;
  document.getElementById("meta").innerHTML = `<b>${entries.length}</b> tasks · <b>${done}</b> complete · reads <b>tasks.js</b>`;
  stamp();
  showProject();
  renderFilters();
  render();
}
function load() {
  const old = document.getElementById("data-script");
  if (old) old.remove();
  // Sentinel: a tasks.js with a syntax error loads but never runs its assignment,
  // so DIARY_TASKS stays undefined and we can surface the breakage instead of
  // silently re-showing the previous (stale) data. Cache-bust always so a reload
  // actually re-reads the file rather than serving a cached copy.
  window.DIARY_TASKS = undefined;
  const s = document.createElement("script");
  s.id = "data-script";
  s.src = "tasks.js?t=" + Date.now();
  s.onload = () => {
    if (!Array.isArray(window.DIARY_TASKS)) {
      entries = [];
      lastJson = null;
      document.getElementById("meta").textContent = "tasks.js loaded but defined no DIARY_TASKS array — likely a syntax error in the file.";
      renderFilters();
      render();
      return;
    }
    const json = JSON.stringify(window.DIARY_TASKS);
    if (json === lastJson) { stamp(); return; }  // data unchanged: don't rebuild, leave open cards as they are
    lastJson = json;
    entries = window.DIARY_TASKS;
    paint();
  };
  s.onerror = () => {
    document.getElementById("meta").textContent = "Could not load tasks.js. Keep it in the same folder as this page.";
    document.getElementById("list").innerHTML = '<div class="empty">No diary loaded.</div>';
  };
  document.head.appendChild(s);
}
document.getElementById("filters").addEventListener("click", e => {
  const b = e.target.closest("button"); if (!b) return;
  activeFilter = b.dataset.f;
  document.querySelectorAll("#filters button").forEach(btn => btn.setAttribute("aria-pressed", btn.dataset.f === activeFilter));
  render();
});
function showProject() {
  let name = window.DIARY_PROJECT || "";
  if (!name) {
    try {
      const parts = decodeURIComponent(location.pathname).split("/").filter(Boolean);
      if (parts.length && parts[parts.length - 1].includes(".")) parts.pop();  // drop a trailing filename
      name = parts[parts.length - 1] || "";
    } catch (_) { /* leave name empty */ }
  }
  const el = document.getElementById("project");
  if (!name) { el.textContent = ""; return; }
  el.innerHTML = `<b>${esc(name)}</b>`;
  document.title = name + " · working diary";
}
showProject();
document.getElementById("reload").addEventListener("click", load);
document.getElementById("auto").addEventListener("change", e => {
  clearInterval(timer);
  if (e.target.checked) timer = setInterval(load, 5000);
});
load();
</script>
</body>
</html>
~~~~

═══════════════════════════════════════════════════════
diary-serve.sh — write verbatim, then `chmod +x diary-serve.sh`
═══════════════════════════════════════════════════════

The viewer is JS-driven, so on a remote/SSH host it must be served over HTTP (a webview preview shows a blank shell). This helper serves the folder on `127.0.0.1` — and binding to localhost is exactly what makes VS Code Remote-SSH auto-forward the port back to your laptop. The script prints the URL to open and the manual port-forward fallback.

~~~~bash
#!/usr/bin/env sh
# Serve this folder so diary.html renders in a browser.
#
# Why: diary.html builds the page in JavaScript (it injects <script src="tasks.js">
# at runtime), so a CSP-sandboxed webview "HTML Preview" shows a blank shell. It
# needs file:// (local double-click) or HTTP (remote). This serves over HTTP.
#
# Port: no fixed port, so serving several repos at once never collides. The port
# is derived from THIS folder's path (stable across restarts → same repo keeps
# its port and forward), then probed upward until one is actually free. Pass an
# explicit port to override.
#   Usage:  ./diary-serve.sh [port]
#
# Port-forwarding: we bind to 127.0.0.1, which is the trigger for VS Code
# Remote-SSH (and most editors) to auto-forward the port to your local machine.
set -e
DIR="$(cd "$(dirname "$0")" && pwd)"
PY="$(command -v python3 || command -v python || true)"
if [ -z "$PY" ]; then
  echo "No python found. On your LOCAL machine you can instead just double-click diary.html." >&2
  exit 1
fi
PORT="$("$PY" - "$DIR" "${1:-}" <<'PYEOF'
import sys, socket, zlib
folder, want = sys.argv[1], sys.argv[2]
def free(p):
    s = socket.socket()
    try:
        s.bind(("127.0.0.1", p)); return True
    except OSError:
        return False
    finally:
        s.close()
if want:
    print(int(want)); raise SystemExit
base = 8000 + (zlib.crc32(folder.encode()) % 1000)  # stable per-folder (8000-8999)
p = base
for _ in range(500):
    if free(p):
        print(p); raise SystemExit
    p += 1
print(base)  # give up probing; http.server will report the conflict loudly
PYEOF
)"
echo "Serving:  $DIR"
echo
echo "  →  Open in your LOCAL browser:   http://localhost:${PORT}/diary.html"
echo
if [ -n "$VSCODE_IPC_HOOK_CLI" ] || [ -n "$SSH_CONNECTION" ]; then
  echo "Remote session detected. VS Code auto-forwards listened ports to your laptop."
  echo "If the link doesn't open: open the PORTS tab (next to TERMINAL) → Forward a Port →"
  echo "enter ${PORT} → click the 🌐 globe. The localhost URL above then works on your machine."
  echo
fi
echo "(Ctrl-C to stop.)"
cd "$DIR"
exec "$PY" -m http.server "$PORT" --bind 127.0.0.1
~~~~
