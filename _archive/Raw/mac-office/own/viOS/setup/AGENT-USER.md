# Optional: separate Mac user for agents (strongest lock)

**Why:** chat rules can be forgotten (context compaction); OS permissions cannot. If scheduled agents run as a
different Mac user, macOS itself blocks them from `~/viOS-Private` and from editing `core/`.

**When:** after viOS works well for ~1–2 weeks and you turn on scheduled agents. Not needed on day one.

## Steps (about 10 minutes, needs your admin password)

1. **Create the user** — System Settings → Users & Groups → Add User → Standard, name `viosagent`.
2. **Create a shared group and give it the AI zone:**
   ```bash
   sudo dseditgroup -o create vios
   sudo dseditgroup -o edit -a "$USER" -t user vios
   sudo dseditgroup -o edit -a viosagent -t user vios
   V=~/own/viOS
   # AI zone + tools: group can read/write
   sudo chmod -R +a "group:vios allow list,search,read,write,append,add_file,add_subdirectory,readattr,writeattr,readextattr,writeextattr,file_inherit,directory_inherit" \
     "$V/ai" "$V/.git" "$V/templates"
   # whole viOS: group can read (so agents can read AGENTS.md, core/, skills)
   sudo chmod -R +a "group:vios allow list,search,read,readattr,readextattr,file_inherit,directory_inherit" "$V"
   # the path down to viOS must be traversable
   chmod +a "group:vios allow search" ~ ~/own
   ```
   Result: `viosagent` can **read** `core/` but **not write** it, and has **no access** to `~/viOS-Private`
   (it is owner-only, and encrypted).
3. **Run scheduled agents as that user** — install the plists for `viosagent` instead of yourself
   (log in once as `viosagent`, run `bash ~/../vishnuvarthanvenkatapathy/own/viOS/setup/schedules.sh on`),
   and configure its own AI keys / Ollama.
4. **Test:** `sudo -u viosagent cat ~/viOS-Private/README.md` → must say *Permission denied*.
   `sudo -u viosagent touch ~/own/viOS/core/x` → must fail.

To undo: remove the user in System Settings and run `sudo chmod -R -N ~/own/viOS` (removes the extra permissions).
