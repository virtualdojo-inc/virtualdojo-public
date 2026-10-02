# VirtualDojo QuickBooks Connector

Syncs data between VirtualDojo and **QuickBooks Desktop Enterprise** (built for version 24.0,
qbXML 16.0).

- **Send to QuickBooks** — pushes customers, items, and invoices from VirtualDojo into
  QuickBooks. A mapping for AP invoices (as Bills) is also supported.
- **Bring back from QuickBooks** — reads payment status from QuickBooks and updates the matching
  invoice in VirtualDojo (for example, marking it **Paid**).

QuickBooks Desktop has no network API, so the connector runs on a Windows computer that can open
your company file. It talks to QuickBooks locally and to VirtualDojo over HTTPS. Nothing is
installed beyond the program files themselves.

## The two programs

| Program | File | Use it for |
|---|---|---|
| **Desktop app** | `virtualdojo_sync.exe` | Day-to-day use. Double-click to open a window where you sign in, edit field mappings, and run a sync. |
| **Console app** | `virtualdojo_sync_console.exe` | Scheduled or unattended runs, diagnostics, and scripting. Run it from Command Prompt or PowerShell. Every command is listed in the [console commands reference](docs/console-commands.md). |

Both programs share the same sign-in, configuration, mappings, and local database. Set something
up in one and the other sees it.

## Requirements

- Windows, with QuickBooks Desktop Enterprise installed on the same computer
- Access to the QuickBooks company file (`.QBW`) you want to sync
- A VirtualDojo account (browser sign-in with Microsoft or Google), or a VirtualDojo API key
  (starts with `sk-`) for scheduled runs
- An administrator-level QuickBooks user for the first-time connection (see below)

You do **not** need Python or any other software.

## Download

Get the latest `virtualdojo_sync.exe` and `virtualdojo_sync_console.exe` from the
[Releases page](https://github.com/virtualdojo-inc/virtualdojo-public/releases).
Each release includes a `SHA256SUMS.txt` file so you can confirm your download is intact:

```powershell
Get-FileHash .\virtualdojo_sync.exe -Algorithm SHA256
```

The result should match the line for that file in `SHA256SUMS.txt`.

## Install

1. **Choose a permanent folder** and put both `.exe` files there, for example
   `C:\VirtualDojoSync\`.
2. **Do not move the files after the first connection.** QuickBooks remembers which program you
   approved, and it appears to key that approval on the program's location. Replacing a file *in
   the same folder* with a newer version does not ask again. Putting a new version in a *different*
   folder triggers the permission prompt again.
3. **First-time QuickBooks approval.** Open QuickBooks, switch the company file to
   **single-user mode**, and sign in as an **administrator**. Then start the connector and run
   a connection check with `virtualdojo_sync_console doctor --deep`, which opens a QuickBooks
   session. QuickBooks asks whether to let the application access the company file. Review the
   options it offers and grant access.

> **Windows or antivirus warning?** The programs are currently **not code-signed**, so Windows
> SmartScreen or QuickBooks may describe the app as coming from an unknown publisher or "without
> a certificate". Verify the SHA-256 hash against `SHA256SUMS.txt`, then choose to continue.

## Quick start

From Command Prompt in the folder that holds the programs:

```console
virtualdojo_sync_console config --init
virtualdojo_sync_console login
virtualdojo_sync_console config --company-file "C:\Path\To\YourCompany.QBW"
virtualdojo_sync_console doctor --deep
virtualdojo_sync_console run to-quickbooks
```

1. `config --init` writes a starter mapping file if you do not have one yet.
2. `login` opens your browser to sign in to VirtualDojo.
3. `config --company-file` tells the connector which QuickBooks company file to use.
4. `doctor --deep` checks that sign-in works and that it can open a QuickBooks session.
5. `run to-quickbooks` performs a **Test Mode** run. It reads and validates everything but
   **writes nothing**. Read the results, fix anything it reports, and only then add `--commit`
   to write for real:

```console
virtualdojo_sync_console run to-quickbooks --commit
virtualdojo_sync_console run to-virtualdojo --commit
```

**Test Mode is the default.** A run changes nothing unless you pass `--commit`.

> **Review the starter mapping before your first real run.** The mapping that `config --init`
> writes contains example account names and settings (income, expense, tax, and A/R accounts).
> They must match accounts that exist in *your* chart of accounts. `doctor` lists any it cannot
> verify. Change them with `config --set NAME=VALUE`, or in the desktop app's mapping screen.

## Safety features

- **Test Mode by default.** Nothing is written without `--commit`.
- **Record cap.** A run writes at most 100 records per object by default, so a mistyped filter
  cannot flood your ledger. Change it with `config --max-records N` (`0` removes the limit).
- **Duplicate protection.** Before any record is sent to QuickBooks, the connector saves a
  note about it in its local database. If the program or computer stops mid-write, the next run
  checks QuickBooks for that record instead of adding it a second time. If it cannot tell, it
  stops and asks you rather than guess. See `state` in the
  [console commands reference](docs/console-commands.md#state).
- **Credentials stay out of files.** Sign-in tokens are stored in Windows Credential Manager,
  never in a configuration file or log.

## Documentation

| Guide | What is in it |
|---|---|
| [Console commands](docs/console-commands.md) | Every command and option, with examples |
| [Files and data](docs/files-and-data.md) | Where the configuration, field mappings, SQLite database, logs, and reports live, and how to back them up |
| [Troubleshooting](docs/troubleshooting.md) | Exit codes, the diagnostic `doctor` command, and common problems |

## Support

Contact your VirtualDojo representative or open a case at
[support.virtualdojo.com](https://support.virtualdojo.com). When you do, include the output of
`virtualdojo_sync_console doctor` and the latest `sync.log` (see
[Files and data](docs/files-and-data.md#logs)).
