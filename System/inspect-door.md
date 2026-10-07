# Door inspect (masked, temporary)

## server.mjs
```
// viOS brain - simple MCP server (read + write notes). No delete.
import express from "express";
import fs from "node:fs/promises";
import path from "node:path";
import { Server } from "@[LONG-MASKED].js";
import { StreamableHTTPServerTransport } from "@[LONG-MASKED].js";
import { ListToolsRequestSchema, CallToolRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const ROOT = path.resolve(process.env.BRAIN_DIR || "/brain");
const PORT = Number(process.env.PORT || 8000);

const INSTRUCTIONS = `This is Vishnu's viOS brain (his second brain).
Before any work: read RULES.md, "About me.md" and "Goals.md".
Paths are relative to the brain, for example "Projects/halle/STATE.md".
Follow RULES.md for where and how to write. Nothing can be deleted; move old things to Archive/.`;

function safe(rel) {
  const p = path.resolve(ROOT, String(rel || "").replace(/^\/+/, "").replace(/^brain\//, ""));
  if (p !== ROOT && !p.startsWith(ROOT + path.sep)) throw new Error("Path is outside the brain");
  if (p.split(path.sep).includes(".git")) throw new Error("Not allowed");
  return p;
}
const relOf = (p) => path.relative(ROOT, p) || ".";

async function walk(dir, out = []) {
  for (const e of await fs.readdir(dir, { withFileTypes: true })) {
    if (e.name.startsWith(".")) continue;
    const full = path.join(dir, e.name);
    if (e.isDirectory()) await walk(full, out);
    else out.push(full);
  }
  return out;
}

const str = (d) => ({ type: "string", description: d });
const TOOLS = [
  { name: "list_notes", description: "List all notes (files) in the brain, or in one folder.",
    inputSchema: { type: "object", properties: { folder: str("Folder, e.g. 'Projects'. Leave empty for everything.") } } },
  { name: "read_note", description: "Read one note.",
    inputSchema: { type: "object", properties: { path: str("e.g. 'RULES.md' or 'Projects/halle/STATE.md'") }, required: ["path"] } },
  { name: "search_notes", description: "Search all notes for a word or phrase (not case sensitive). Returns matching lines.",
    inputSchema: { type: "object", properties: { query: str("Text to find") }, required: ["query"] } },
  { name: "write_note", description: "Create a note or replace a whole note. Folders are created when needed. Use .md files.",
    inputSchema: { type: "object", properties: { path: str("e.g. 'Inbox/2026-10-02 idea.md'"), content: str("Full Markdown text") }, required: ["path", "content"] } },
  { name: "append_note", description: "Add text to the end of a note (creates it if missing). Good for logs.",
    inputSchema: { type: "object", properties: { path: str("Note path"), text: str("Text to add") }, required: ["path", "text"] } },
  { name: "move_note", description: "Move or rename a note, e.g. to Archive/. Will not overwrite an existing note.",
    inputSchema: { type: "object", properties: { from: str("Current path"), to: str("New path") }, required: ["from", "to"] } },
];

const ok = (text) => ({ content: [{ type: "text", text }] });

async function call(name, a = {}) {
  switch (name) {
    case "list_notes": {
      const files = await walk(safe(a.folder || ""));
      return ok(files.map(relOf).sort().join("\n") || "(empty)");
    }
    case "read_note":
      return ok(await fs.readFile(safe(a.path), "utf8"));
    case "search_notes": {
      const q = String(a.query || "").toLowerCase();
      if (!q) throw new Error("Empty query");
      const hits = [];
      for (const f of await walk(ROOT)) {
        let t; try { t = await fs.readFile(f, "utf8"); } catch { continue; }
        t.split("\n").forEach((line, i) => {
          if (hits.length < 200 && line.toLowerCase().includes(q)) hits.push(`${relOf(f)}:${i + 1}: ${line.trim()}`);
        });
      }
      return ok(hits.join("\n") || "No matches");
    }
    case "write_note": {
      const p = safe(a.path);
      await fs.mkdir(path.dirname(p), { recursive: true });
      await fs.writeFile(p, String(a.content ?? ""), "utf8");
      return ok(`Saved ${relOf(p)}`);
    }
    case "append_note": {
      const p = safe(a.path);
      await fs.mkdir(path.dirname(p), { recursive: true });
      let pre = ""; try { const cur = await fs.readFile(p, "utf8"); if (cur && !cur.endsWith("\n")) pre = "\n"; } catch {}
      await fs.appendFile(p, pre + String(a.text ?? "") + "\n", "utf8");
      return ok(`Added to ${relOf(p)}`);
    }
    case "move_note": {
      const from = safe(a.from), to = safe(a.to);
      try { await fs.access(to); throw new Error("A note already exists at " + relOf(to)); } catch (e) { if (e.code !== "ENOENT") throw e; }
      await fs.mkdir(path.dirname(to), { recursive: true });
      await fs.rename(from, to);
      return ok(`Moved ${relOf(from)} -> ${relOf(to)}`);
    }
    default: throw new Error("Unknown tool " + name);
  }
}

function makeServer() {
  const s = new Server({ name: "viOS brain", version: "1.0.0" }, { capabilities: { tools: {} }, instructions: INSTRUCTIONS });
  s.setRequestHandler(ListToolsRequestSchema, async () => ({ tools: TOOLS }));
  s.setRequestHandler(CallToolRequestSchema, async (req) => {
    try { return await call(req.params.name, req.params.arguments); }
    catch (e) { return { content: [{ type: "text", text: "Error: " + e.message }], isError: true }; }
  });
  return s;
}

const app = express();
app.use(express.json({ limit: "20mb" }));
app.post("/mcp", async (req, res) => {
  const server = makeServer();
  const transport = new StreamableHTTPServerTransport({ sessionIdGenerator: undefined, enableJsonResponse: true });
  res.on("close", () => { transport.close(); server.close(); });
  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});
app.all("/mcp", (req, res) => res.status(405).set("Allow", "POST").json({ jsonrpc: "2.0", error: { code: -32000, message: "Method not allowed" }, id: null }));
app.get("/health", (_q, r) => r.send("ok"));
app.listen(PORT, "0.0.0.0", () => console.log(`viOS brain MCP on :${PORT}, brain=${ROOT}`));
```
## crontab
```
*/5 * * * * cd /home/ubuntu/vios/brain && git add -A && git commit -qm auto-save >/dev/null 2>&1 # vios-autosave
30 2 * * * ~/vios/vios-backup.sh >> ~/vios/backup.log 2>&1
7 * * * * python3 ~/vios/gen-dashboard.py >> ~/vios/dashboard.log 2>&1
```
## containers
```
ops-web  Up 45 hours (healthy)
vios-mcp-1  Up 3 days
vault-vaultwarden-1  Up 3 days (healthy)
vios-silverbullet-1  Up 3 days (healthy)
ops-console  Up 3 days (healthy)
core-api  Up 3 days (healthy)
core-postgres  Up 3 days (healthy)
core-minio  Up 3 days (healthy)
```
## mcp folder
```
total 20
drwxrwxr-x 2 ubuntu ubuntu 4096 Oct  2 14:49 .
drwxrwxr-x 7 ubuntu ubuntu 4096 Oct  7 02:59 ..
-rw-rw-r-- 1 ubuntu ubuntu  158 Oct  2 14:49 Dockerfile
-rw-rw-r-- 1 ubuntu ubuntu 6041 Oct  2 14:49 server.mjs
```
## compose files
### docker-compose.yml
```
name: vios
services:
  silverbullet:
    image: ghcr.io/silverbulletmd/silverbullet:latest
    restart: unless-stopped
    environment:
      SB_USER: "vishnu:${SB_PASS}"
      PUID: "${PUID}"
      PGID: "${PGID}"
    ports: ["127.0.0.1:3100:3000"]
    volumes:
      - ./brain:/space
  mcp:
    build: ./mcp
    restart: unless-stopped
    user: "${PUID}:${PGID}"
    environment:
      HOME: /tmp
    ports: ["127.0.0.1:8100:8000"]
    volumes:
      - ./brain:/brain
```
