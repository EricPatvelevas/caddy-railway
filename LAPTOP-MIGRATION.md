# Moving this development setup to another laptop

Inventory checked on 2026-10-28 across caddy-railway, tlr-gl, tlr-green,
tlr-blue, tlr-yellow, tlr-purple, tlr-haven, tlr-gem, and tlr-white.

## Save before moving

1. Enable VS Code Settings Sync for settings, keybindings, snippets, extensions,
   and profiles, using the account you will use on the new laptop.
2. Commit and push the changes you want to keep in every repository. Uncommitted
   changes and untracked files will not appear in a fresh clone.
3. Back up private `.env` and `.env.railway*` files, OAuth credential JSON files,
   SSH keys, database dumps, and needed Docker volume data through a secure
   backup. Git deliberately ignores these files. Keep the backup outside Git.
4. Record the versions of VS Code, Git, Docker Desktop/Compose, Node.js, pnpm,
   uv, and any WSL distributions you use. These system tools are not installed
   by Settings Sync or Python lockfiles.

Keep `uv.lock`, `package-lock.json`, NuGet `packages.lock.json`, runtime version
files, `.vscode/`, Dockerfiles, Compose files, and environment templates in Git.
Do not copy `.venv`, `node_modules`, build output, or caches to recreate a setup.

## Windows and WSL are separate environments

Green, Blue, Yellow, Purple, and White now use Windows CPython 3.14.6 for local
development. White was migrated from Linux to Windows after the initial audit;
its previous environment is retained in its ignored
`.local/venv-linux-backup-20261028` directory.
Recreate these local environments from Windows PowerShell on the new laptop,
and open the projects in a local Windows VS Code window. Docker builds use
their own Linux environments. A Python virtual environment cannot be shared
between Windows and Linux.

The checked-in VS Code tasks are also part of the setup. In particular,
tlr-gl's packaging tasks invoke pnpm, PowerShell, and WinRAR at
`C:\Program Files\WinRAR\Rar.exe`. Install those tools to use those tasks as
written, or update the task paths for the new machine.

## Restore VS Code

Install VS Code, sign into the same Settings Sync account, and enable Sync.
Restore project `.vscode/` files by cloning the repositories.

`development/vscode-extensions.windows.txt` contains the extension IDs and
versions found in this laptop's Windows user installation on the inventory date.
As an alternative to Sync, install the recorded versions from PowerShell in
caddy-railway:

```powershell
Get-Content .\development\vscode-extensions.windows.txt | ForEach-Object {
    code --install-extension $_
}
```

Some recorded extension versions may no longer be available. The list does not
capture enablement, profile membership, settings, extension logins, or WSL
extensions. Use Settings Sync for editor preferences, sign into extensions as
needed, and install any required extensions in the WSL extension host separately.
Refresh the snapshot from Windows PowerShell before the actual move:

```powershell
code --list-extensions --show-versions |
    Set-Content .\development\vscode-extensions.windows.txt
```

## Restore projects

Clone these repositories into sibling directories, matching the current layout.
Some tasks use sibling paths, including Purple's use of Green's MongoDB service.
Install Docker Desktop and enable the integration needed by your chosen terminal.

| Projects | Restore dependencies |
| --- | --- |
| tlr-green, tlr-blue, tlr-yellow, tlr-purple, tlr-white | Install Python 3.14.6 and uv, then run `uv sync --locked --python 3.14.6` inside each repository. Select the recreated interpreter in VS Code. |
| tlr-gl | Install Node.js and run `npm ci` using its committed `package-lock.json`. Existing VS Code packaging tasks additionally require pnpm. |
| tlr-haven | Use Node.js 24 as declared by `.nvmrc`, then run `npm ci`. Record the exact Node patch version if you need to reproduce it. |
| tlr-gem | Use Windows and the .NET 10 SDK selected by `global.json`; run `dotnet restore --locked-mode`, then follow its README to download models and publish. |
| caddy-railway | Follow its README and Dockerfile for the gateway build. |

For exact runtime matching, also preserve the versions of uv, Node.js, pnpm,
Docker, and .NET actually used. A broad version selector such as Node.js `24`,
Python `3.14`, or .NET SDK roll-forward permits updates. Docker image tags can
also change; use image digests when an immutable image is required.

Restore private environment files before starting services. Templates explain
which settings are needed; they do not contain your production credentials.
Green now has `.env.example`; Green and Blue also have service-specific
`.env.railway-*.example` files. Copy a template to the corresponding filename
and configure it, or restore the original from your secure backup. Follow each
project's README for how it loads configuration: Purple does not automatically
load `.env` during a normal local Python start.

Restore required databases and Docker volumes separately. A Compose file
recreates services and empty volumes; it does not restore their existing data.
Gem's ignored model files can be downloaded with its existing Get-Model script.

## Git cleanup included with this audit

Local environment files in tlr-gl, tlr-green, and tlr-blue were removed from Git's
index and kept on disk. The same applies to Purple's `scripts/backup.sql` data
dump and tlr-gl's generated Vite timestamp files. Git shows staged deletions for
these paths because tracking was removed; the local files were preserved.
The ignore rules, templates, and this guide still need to be committed along
with those removals.

Removing a file from tracking does not remove its contents from earlier commits.
Replace any real credentials that were committed, especially if the repository
has been shared. History cleanup, if needed, is a separate operation.

A production MongoDB credential was also removed from a Green source comment
and four maintenance scripts. Those scripts now require `MONGODB_URI` in the
process environment or an explicit `--mongo-uri`; no database was contacted
during this audit. Restore that configuration securely before using the scripts.

VS Code references: [Settings Sync](https://code.visualstudio.com/docs/configure/settings-sync)
and [extension CLI options](https://code.visualstudio.com/docs/configure/command-line).
