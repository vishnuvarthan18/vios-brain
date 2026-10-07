---
tags: chat
date: 2026-06-21
source: Claude personal account
uuid: e5efb18a-daa8-4bf7-93c8-99e6ba161b51
---
# Resolving npm permission denied errors

## Summary
**Conversation Overview**

The person encountered npm EACCES permission errors when attempting to globally install `@anthropic-ai/claude-code` on their Mac. The errors stemmed from npm's global prefix pointing to `/usr/local/lib/node_modules`, a root-owned directory not writable by the current user. A second error occurred when attempting to self-update npm for the same reason.

Claude diagnosed the root cause as a misconfigured npm global prefix rather than a one-off permissions issue, and explicitly advised against using `sudo` as a fix due to the recurring problems it causes with root-owned files. The recommended solution was to reconfigure npm's prefix to a user-owned directory (`~/.npm-global`), add it to the PATH via `~/.zshrc`, and then retry the original install. Claude also identified that the likely underlying cause was Node.js having been installed via the macOS `.pkg` installer, and suggested switching to Homebrew Node or `nvm` as a more permanent fix, with `nvm` noted as preferable for multi-version workflows.

## Chat

**Vishnu:** Last login: Tue Jun 16 16:25:27 on ttys000
vishnuvarthanvenkatapathy@Mac ~ % npm install -g @anthropic-ai/claude-code
npm error code EACCES
npm error syscall mkdir
npm error path /usr/local/lib/node_modules/@anthropic-ai
npm error errno -13
npm error Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules/@anthropic-ai'
npm error     at async mkdir (node:internal/fs/promises:1507:10)
npm error     at async /usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:590:20
npm error     at async Promise.allSettled (index 0)
npm error     at async #reifyPackages (/usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:322:11)
npm error     at async Arborist.reify (/usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:128:5)
npm error     at async Install.exec (/usr/local/lib/node_modules/npm/lib/commands/install.js:157:5)
npm error     at async Npm.exec (/usr/local/lib/node_modules/npm/lib/npm.js:193:9)
npm error     at async module.exports (/usr/local/lib/node_modules/npm/lib/cli/entry.js:67:5) {
npm error   errno: -13,
npm error   code: 'EACCES',
npm error   syscall: 'mkdir',
npm error   path: '/usr/local/lib/node_modules/@anthropic-ai'
npm error }
npm error
npm error The operation was rejected by your operating system.
npm error It is likely you do not have the permissions to access this file as the current user
npm error
npm error If you believe this might be a permissions issue, please double-check the permissions of the file and its containing directories, or try running the command again as root/Administrator.
npm notice
npm notice New minor version of npm available! 11.16.0 -> 11.17.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v11.17.0
npm notice To update run: npm install -g npm@11.17.0
npm notice
npm error A complete log of this run can be found in: /Users/vishnuvarthanvenkatapathy/.npm/_logs/2026-06-21T07_08_39_657Z-debug-0.log
vishnuvarthanvenkatapathy@Mac ~ %  npm install -g npm@11.17.0
npm error code EACCES
npm error syscall rename
npm error path /usr/local/lib/node_modules/npm
npm error dest /usr/local/lib/node_modules/.npm-i9nnxROI
npm error errno -13
npm error Error: EACCES: permission denied, rename '/usr/local/lib/node_modules/npm' -> '/usr/local/lib/node_modules/.npm-i9nnxROI'
npm error     at async Object.rename (node:internal/fs/promises:1432:10)
npm error     at async moveFile (/usr/local/lib/node_modules/npm/node_modules/@npmcli/fs/lib/move-file.js:30:5)
npm error     at async Promise.allSettled (index 0)
npm error     at async #reifyPackages (/usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:322:11)
npm error     at async Arborist.reify (/usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:128:5)
npm error     at async Install.exec (/usr/local/lib/node_modules/npm/lib/commands/install.js:157:5)
npm error     at async Npm.exec (/usr/local/lib/node_modules/npm/lib/npm.js:193:9)
npm error     at async module.exports (/usr/local/lib/node_modules/npm/lib/cli/entry.js:67:5) {
npm error   errno: -13,
npm error   code: 'EACCES',
npm error   syscall: 'rename',
npm error   path: '/usr/local/lib/node_modules/npm',
npm error   dest: '/usr/local/lib/node_modules/.npm-i9nnxROI'
npm error }
npm error
npm error The operation was rejected by your operating system.
npm error It is likely you do not have the permissions to access this file as the current user
npm error
npm error If you believe this might be a permissions issue, please double-check the permissions of the file and its containing directories, or try running the command again as root/Administrator.
npm error A complete log of this run can be found in: /Users/vishnuvarthanvenkatapathy/.npm/_logs/2026-06-21T07_09_10_026Z-debug-0.log
vishnuvarthanvenkatapathy@Mac ~ %

**Claude:** Your global npm prefix points to `/usr/local`, which your user can't write to. Don't fix this with `sudo` — it leaves root-owned files and recurs. Reconfigure npm to a user-owned prefix.

```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
```

Add to PATH (you're on zsh):

```bash
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

Then install without `-g` permission issues:

```bash
npm install -g @anthropic-ai/claude-code
```

Skip the npm self-update — same EACCES cause, not needed for Claude Code.

**Cleaner alternative:** if you installed Node via the macOS `.pkg` installer, that's why `/usr/local` is root-owned. Switching to Homebrew Node or `nvm` makes the global prefix user-writable and avoids this permanently:

```bash
brew install node
```

(nvm is better if you need multiple Node versions.) Either path works; the prefix reconfig above is the fastest fix right now.
