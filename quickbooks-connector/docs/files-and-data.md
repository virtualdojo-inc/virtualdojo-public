# Files and data

Everything the connector remembers is stored **per Windows user**, in one folder. Nothing is
stored next to the `.exe` — you can move or replace the program files without losing your setup.

## The configuration folder

```
C:\Users\<you>\AppData\Local\VirtualDojo\VirtualDojoSync\
```

Also reachable as `%LOCALAPPDATA%\VirtualDojo\VirtualDojoSync`. To open it, press
**Win + R**, paste that path, and press Enter. (`AppData` is hidden in File Explorer by default;
typing the path works regardless.)

`virtualdojo_sync_console doctor` and `config --show` print the exact path in use on your machine.
If `QBEC_CONFIG_DIR` is set, that folder is used instead.

### What is in it

| File or folder | What it is |
|---|---|
| `config.json` | Your settings: VirtualDojo server, tenant and user, the QuickBooks company file path, and the per-run record cap. **No passwords or tokens.** |
| `mapping.json` | Your field mappings — the rules for which VirtualDojo field fills which QuickBooks field, the filters for which records sync, and the named constants (accounts, class, template). |
| `mapping-history\` | A dated copy of `mapping.json` saved each time a command edits it. Restore one by copying it over `mapping.json`. |
| `state-<company>-<id>.sqlite` | The local **SQLite database** — one per QuickBooks company file. See [below](#the-sqlite-database). |
| `logs\sync.log` | The run log. See [Logs](#logs). |
| `reports\` | Files produced by runs and by `dump`. See [Reports](#reports). |
| `accounts-…json`, `reflists-…json`, `dataext-…json` | Cached lists read from QuickBooks (chart of accounts; classes, templates, terms, sales reps; custom-field definitions), one set per company file. Safe to delete — they are rebuilt. |
| `vdj-schema-….json` | Cached list of VirtualDojo objects and fields, one per server. Safe to delete — it is rebuilt. |

## Saved credentials

Sign-in tokens are **not** in any of the files above. They are stored in **Windows Credential
Manager** (Control Panel → Credential Manager → Windows Credentials), in entries named for
`VirtualDojoSync`. `logout` removes them. Because Credential Manager is per Windows user, a
scheduled task must run as the same user who signed in.

## The SQLite database

File name: `state-<company file name>-<12-character code>.sqlite`, for example
`state-mycompany-1a2b3c4d5e6f.sqlite`. The code is derived from the company file's path, so each
company file gets its own database. If no company file is configured the name is
`state-unknown-e3b0c44298fc.sqlite`; set one with `config --company-file` to avoid that.

The database is what stops the connector from adding the same record to QuickBooks twice. It
remembers which VirtualDojo record became which QuickBooks record.

| Table | What it holds |
|---|---|
| `xref` | The link between each VirtualDojo record and its QuickBooks record (ListID or TxnID), its state (`pending`, `in_flight`, `synced`, `failed`), the attempt count, and the last error. |
| `line_xref` | The same link for individual invoice and bill lines. |
| `cursor` | Per object and direction, when the last successful sync finished — so later runs only read what changed. |
| `sync_run` | One row per run: when it started and finished, whether it was Test Mode, counts, and any error. |
| `meta` | Internal bookkeeping. |

### Looking inside it

You can open the file read-only with any SQLite tool (for example
[DB Browser for SQLite](https://sqlitebrowser.org/)) or the `sqlite3` command-line program:

```console
sqlite3 "%LOCALAPPDATA%\VirtualDojo\VirtualDojoSync\state-mycompany-1a2b3c4d5e6f.sqlite"
sqlite> SELECT run_id, direction, started_at, status, dry_run FROM sync_run ORDER BY started_at DESC LIMIT 10;
```

`dry_run = 1` is a Test Mode run.

> **Do not edit this database by hand.** Use `virtualdojo_sync_console state` to resolve
> interrupted writes (see [`state`](console-commands.md#state)). Closing the connector before
> you open the file avoids locking problems.

### Backing up and moving

- **Back up** the whole configuration folder. The two files that matter most are `mapping.json`
  and the `state-….sqlite` file.
- **Moving to a new computer:** copy the folder to the same location for the new Windows user,
  then run `login` again (credentials do not travel with the files). Keep the company file at the
  **same path** — the database name is derived from the path, so a different path starts a fresh
  database and the connector would no longer know what it already sent.
- **Deleting the `.sqlite` file** makes the connector forget what it has synced. Do not do this
  unless support asks you to; a following run could add records to QuickBooks that are already there.

## Logs

`logs\sync.log` records what each run did, including every write made to QuickBooks. Both the
desktop app and the console app write to it.

- The file always logs at *info* level. Adding `-v` to a console command logs more detail.
- It rotates automatically at 10 MB and keeps 10 older files (`sync.log.1` … `sync.log.10`), so
  it stays bounded without any cleanup. Older ones are deleted.
- On screen, the console shows warnings and errors only unless you pass `-v`.

Send `sync.log` to support when reporting a problem. The connector is built so that sign-in tokens
never reach a log, but the log does name your records and accounts, so review it first if that matters.

## Reports

The `reports\` folder holds:

- `failures-<run id>.csv` — written by a run that had failures: one row per failing record, with
  the reason. The run's summary prints the exact path.
- Files saved by [`dump`](console-commands.md#dump) when you do not choose `--out`.

These contain business data. Delete them when you no longer need them.

## What leaves your computer

The connector contacts only your VirtualDojo server (by default `https://app.virtualdojo.com`) over
HTTPS, and QuickBooks on the same computer. It does not send your files, logs, or reports anywhere
unless you do.
