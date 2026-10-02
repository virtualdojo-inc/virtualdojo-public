# Console commands

Reference for `virtualdojo_sync_console.exe`. Open Command Prompt or PowerShell in the folder
where you installed it and run:

```console
virtualdojo_sync_console <command> [options]
```

(You can type the name with or without `.exe`.) Add `-h` or `--help` after any command to see its
options, for example `virtualdojo_sync_console run --help`.

**Global options** go before the command:

| Option | Meaning |
|---|---|
| `--version` | Print the version and exit |
| `-v`, `--verbose` | Debug logging, on screen and in the log file |
| `--config-dir PATH` | Use this folder for the configuration, mapping, SQLite database, logs and reports instead of the default. Lets you run separate configurations side by side. Sign-in is still shared per Windows user. See [Running two configurations](files-and-data.md#running-two-configurations) |
| `-h`, `--help` | Show help |

## Commands at a glance

| Command | What it does | Changes anything? |
|---|---|---|
| [`doctor`](#doctor) | Checks your setup and sign-in | No |
| [`login`](#login) | Signs in to VirtualDojo | Saves credentials |
| [`logout`](#logout) | Removes saved credentials | Removes credentials |
| [`config`](#config) | Shows or changes settings | Settings only |
| [`run`](#run) | Runs a sync | **Only with `--commit`** |
| [`discover`](#discover) | Reads your QuickBooks file and suggests settings | Only with `--apply` |
| [`dump`](#dump) | Saves raw QuickBooks data to a file for support | No |
| [`mapping`](#mapping) | Shows and edits field mappings | Mapping file only |
| [`state`](#state) | Shows and resolves interrupted writes | Only with `--mark-*` |
| [`customfield`](#customfield) | Lists QuickBooks custom fields, or creates one | **`add` writes one custom-field definition** |
| [`update`](#update) | Checks whether a newer version is available | No |
| [`gui`](#gui) | Opens the desktop window | — |

---

## doctor

Checks that the connector is set up and can reach what it needs: credentials, the mapping, lists
cached from QuickBooks, and QuickBooks itself.

```console
virtualdojo_sync_console doctor
virtualdojo_sync_console doctor --deep
```

| Option | Meaning |
|---|---|
| `--deep` | Actually open a QuickBooks session. Without it, QuickBooks is not contacted. |

Exit code `0` means everything checked out; `1` means at least one problem was reported.
`doctor` also prints the path to your configuration folder.

## login

Signs in to VirtualDojo. By default it opens your web browser for single sign-on.

```console
virtualdojo_sync_console login
virtualdojo_sync_console login --provider microsoft
virtualdojo_sync_console login --server https://app.virtualdojo.com
virtualdojo_sync_console login --api-key sk-xxxxxxxx
```

| Option | Meaning |
|---|---|
| `--server SERVER` | Which VirtualDojo to use, e.g. `https://app.virtualdojo.com` |
| `--provider PROVIDER` | `microsoft` or `google` |
| `--api-key API_KEY` | Use an `sk-` API key instead of the browser. Best for scheduled, unattended runs. |

Credentials are saved in Windows Credential Manager, not in a file.

## logout

Removes the saved credentials.

```console
virtualdojo_sync_console logout
```

## config

Shows or changes settings. With no options it prints the current settings.

```console
virtualdojo_sync_console config --init
virtualdojo_sync_console config --show
virtualdojo_sync_console config --company-file "C:\Path\To\YourCompany.QBW"
virtualdojo_sync_console config --max-records 250
virtualdojo_sync_console config --set income_account="Sales:Product Sales"
virtualdojo_sync_console config --server https://dev.virtualdojo.com
```

| Option | Meaning |
|---|---|
| `--init` | Write a starter mapping file if you do not have one. Never overwrites an existing mapping. |
| `--show` | List the mapping's constants (their kinds and current values) and each object's filters. |
| `--company-file PATH` | Choose the QuickBooks company file (`.QBW`). **Each company file has its own local database**, so changing this changes which saved records and in-progress writes `state` reports. Credentials are untouched. |
| `--max-records N` | Most records written per object per run. Default `100`. `0` means no limit. |
| `--set NAME=VALUE` | Set a mapping constant, such as an account or class name. Repeat the option to set several. Use `config --show` to see the available names. |
| `--server URL` | Point at a different VirtualDojo. **Clears your saved credentials**, so sign in again afterward. |

Set the company file explicitly. If it is blank, QuickBooks uses whichever file is currently open,
and the local database is shared under the name `unknown` — two different company files would then
share one record of what was synced.

## run

Runs a sync in one direction.

```console
virtualdojo_sync_console run to-quickbooks
virtualdojo_sync_console run to-quickbooks --commit
virtualdojo_sync_console run to-virtualdojo --commit
```

| Argument / option | Meaning |
|---|---|
| `to-quickbooks` | VirtualDojo → QuickBooks. Sends the objects your mapping marks as going to QuickBooks. |
| `to-virtualdojo` | QuickBooks → VirtualDojo. Brings mapped fields back, such as invoice payment status. |
| `--commit` | **Write for real.** Without it the run is **Test Mode** and changes nothing. |
| `--scoped` | Sync only the *root* records (normally invoices) and whatever they reference, instead of every mapping's whole filtered set. Much faster on a large account. The record cap then counts roots. See [`mapping scope`](#mapping). |
| `--allow-literal-refs` | Permit a real post through a hardcoded QuickBooks reference. In that case every record posts to the **same** QuickBooks record, so this is refused by default. |

At the end of every run the connector prints a summary and, if anything failed, the path to a
`failures-<run id>.csv` file listing every failing record (see
[Files and data](files-and-data.md#reports)).

**Always run once without `--commit` first** and read what it reports.

`run` exit codes: `0` finished cleanly, `1` the run was aborted, `2` finished but something needs
your attention. Other codes are in the [exit codes table](troubleshooting.md#exit-codes).

## discover

Reads your QuickBooks company file (read-only) and proposes configuration values from what it
finds.

```console
virtualdojo_sync_console discover
virtualdojo_sync_console discover --apply
```

| Option | Meaning |
|---|---|
| `--apply` | Write the values that can be derived unambiguously. Without it, `discover` only prints proposals. |

## dump

Saves the raw XML QuickBooks returns for a list or transactions to a file. Read-only. Mainly used
when working with support on field mappings.

```console
virtualdojo_sync_console dump Customer
virtualdojo_sync_console dump Invoice --ref-number INV-1001
virtualdojo_sync_console dump Customer --name "Acme:Job1" --out acme.xml
virtualdojo_sync_console dump Item --all
```

Choose one of: `Customer`, `Item`, `Account`, `Vendor`, `Class`, `Template`, `Terms`, `SalesRep`,
`Invoice`, `Bill`, `DataExtDef`.

| Option | Meaning |
|---|---|
| `--name FULLNAME` | Limit to one list record, such as `Acme:Job1`. Repeatable. Not valid for `Invoice` or `Bill`. |
| `--ref-number REFNUMBER` | Limit to one transaction, such as `INV-1`. Repeatable. For `Invoice` or `Bill`. |
| `--limit LIMIT` | Maximum records. Default `200`. |
| `--all` | No limit — reads the entire list. |
| `--no-custom-fields` | Leave out custom fields. |
| `--out OUT` | Where to save the file. Default: the `reports` folder. |

> A dump contains your real QuickBooks data. Treat the file accordingly before sending it to
> anyone.

## mapping

Shows and edits the field mappings (the rules for which VirtualDojo field fills which QuickBooks
field). Changes are saved to `mapping.json` in your [configuration folder](files-and-data.md),
and each edit keeps a dated backup in `mapping-history`.

```console
virtualdojo_sync_console mapping <verb> [options]
```

**Look at mappings**

| Verb | What it does |
|---|---|
| `fields [ID]` | List what is mapped. Give a mapping ID (such as `invoices_to_invoice`) for one, or none for all. `config --show` lists the IDs. |
| `targets ENTITY` | List what *can* be mapped on a QuickBooks entity, e.g. `Invoice`, `Customer`, `ItemNonInventory`, `InvoiceLineAdd`. Add `--unmapped ID` to hide ones already mapped. |
| `sources OBJECT` | List the VirtualDojo field names for an object (e.g. `invoices`) — the values `--source` accepts. `--mapped-by ID` marks the ones already used; `--refresh` re-fetches from VirtualDojo instead of using the cached copy. |
| `diff` | Show what this version ships that you do not have. |

**Turn mappings on or off**

| Verb | What it does |
|---|---|
| `enable ID` | Include this mapping in runs. |
| `disable ID` | Skip this mapping entirely. |

**Add and remove objects**

| Verb | What it does |
|---|---|
| `add-object --vdj-object OBJ --qb-entity ENTITY [--direction {to-quickbooks,to-virtualdojo}] [--id ID] [--with-lines]` | Create a new, empty mapping. The default ID is `<object>_to_<entity>`. `--with-lines` adds a line-item block for entities that have one. |
| `rm-object ID` | Remove a mapping and its fields. |

**Filter which records sync**

| Verb | What it does |
|---|---|
| `set-filter ID KEY=VALUE` | Add or replace a filter term, e.g. `status_in=Issued,Sent,Paid` or `issued_date_gte=2026-01-01`. Suffix the field name with an operator for anything other than an exact match. |
| `rm-filter ID KEY` | Remove one filter term. |

**Edit fields**

| Verb | What it does |
|---|---|
| `add-field ID --target PATH [options]` | Map a new field. |
| `set-field ID --target PATH [options]` | Change a mapped field; settings you do not name are kept. |
| `rm-field ID --target PATH [--lines]` | Unmap a field. |

`add-field` and `set-field` take the same options:

| Option | Meaning |
|---|---|
| `--target PATH` | The QuickBooks field, such as `RefNumber` or `SalesAndPurchase.PrefVendorRef.FullName`. Required. |
| `--lines` | The field belongs to the line items, not the document header. |
| `--source SOURCE` | Take the value from this VirtualDojo field. |
| `--const CONST` | Take the value from a named constant (see `config --set`). |
| `--literal LITERAL` | Use this fixed value for every record. |
| `--related ALIAS.FIELD` | Take the value from a related record. |
| `--resolve NAME` | Look the value up in QuickBooks and send its ID instead of its name. Without this, references are matched **by name** and break if someone renames the QuickBooks record. |
| `--coerce COERCE` | Override the automatic type conversion. |
| `--date-format STRFTIME` | Write a date using this pattern (e.g. `%m/%d/%Y`) instead of `YYYY-MM-DD`. Only for custom fields declared as dates. |
| `--max-len MAX_LEN` | Override the length limit. |
| `--on-overflow {error,truncate,truncate_hash}` | What to do when a value is too long. References and document numbers default to `error`, because shortening them can make two different records collide. |
| `--on-null {omit,error,empty}` | What to do when the source value is blank. |
| `--note NOTE` | A note stored with the mapping. |
| `--required` / `--not-required` | Whether a blank value should stop the record. |
| `--allow-pinned-reference` | Allow a hardcoded QuickBooks reference. |

**Control scoped runs**

| Verb | What it does |
|---|---|
| `scope` | Show which mappings a `--scoped` run starts from. |
| `scope --add ID` | Make this mapping a scope root. |
| `scope --remove ID` | Stop treating it as a root. |
| `scope --clear` | Remove every root. `run --scoped` then refuses to run rather than syncing nothing. |

**Copy from what the program ships**

| Verb | What it does |
|---|---|
| `adopt ID` | Copy a whole object from the shipped defaults. |
| `adopt ID --target TARGET [--lines]` | Copy just one field. |
| `adopt ID --ordering` / `--requires` / `--relations` | Also copy the run order, the "records this object requires" links used by `--scoped`, or the related-record lookups an object needs. |

## state

Lists writes whose outcome is unknown — for example because the program or computer stopped in
the middle of sending a record to QuickBooks — and lets you resolve them.

```console
virtualdojo_sync_console state
virtualdojo_sync_console state --in-flight
virtualdojo_sync_console state --mark-synced invoices/<record-id> --qb-id <TxnID>
virtualdojo_sync_console state --mark-failed invoices/<record-id>
```

| Option | Meaning |
|---|---|
| `--in-flight` | List writes whose outcome is unknown. This is the default view. |
| `--mark-synced OBJECT/ID` | You checked QuickBooks and the record **did** arrive. Requires `--qb-id`. |
| `--mark-failed OBJECT/ID` | You checked QuickBooks and the record did **not** arrive. The next run sends it again. |
| `--qb-id ID` | The TxnID or ListID you found in QuickBooks. Required with `--mark-synced`, so the record can always be linked back up. |

Normally you do not need this: the next run reconciles interrupted writes itself and stops if it
cannot be sure. Use `state` when it stops and tells you a record needs a human decision.

## customfield

A mapping can only write to a QuickBooks custom field that already exists in the company file.
This command lists them, and can create one.

```console
virtualdojo_sync_console customfield list
virtualdojo_sync_console customfield add "Contract Type" --assign Invoice
```

| Verb / option | Meaning |
|---|---|
| `list` | List the custom fields in the company file and what each is assigned to. Read-only. |
| `add NAME --assign OBJECT` | Create a custom field called NAME for OBJECT (for example `Invoice`). Repeat `--assign` for several objects. Does nothing if the field already exists; if it exists but is not assigned to every object you gave, it stops and changes nothing. |
| `--type TYPE` | QuickBooks data type. Default `STR255TYPE` (text up to 255 characters). |

`add` is the only thing this command writes to your company file. The name is matched exactly when
values are written, so type it exactly as it should read in QuickBooks. QuickBooks must be able to
open the company file configured with `config --company-file`.

## update

Reports whether a newer release has been published. With `--install` it also downloads, verifies and installs it; without it, nothing is downloaded or changed.

```console
virtualdojo_sync_console update
virtualdojo_sync_console update --install
```

Exit code `0`: you are up to date. `2`: a newer version exists (the download page is printed).
`1`: the check could not be completed, for example because the computer is offline. With `--install`, `0` means it installed (or you were already current) and `1` means it could not; nothing is changed if a check fails. Run the program again afterwards. `doctor` also
shows an `updates` line. Set `QBEC_NO_UPDATE_CHECK=1` to switch off the daily check the desktop app
does (it runs hourly); the `update` command itself always checks when you run it.

## gui

Opens the desktop window (the same as running `virtualdojo_sync.exe`).

```console
virtualdojo_sync_console gui
```

---

## Advanced: environment variables

Set these in the console window before running a command (`set NAME=value` in Command Prompt,
`$env:NAME = "value"` in PowerShell).

| Variable | Meaning |
|---|---|
| `QBEC_CONFIG_DIR` | Use this folder instead of the default [configuration folder](files-and-data.md). Handy for keeping separate setups. |
| `QBEC_TRACE_XML` | Path to a file. When set, every request sent to and reply from QuickBooks is written to it. Useful when support asks for one. The file contains your data. |
| `QBEC_NO_UPDATE_CHECK` | Set to `1` to turn off the desktop app's daily update check. |
| `QBEC_NO_DIALOGS` | Set to `1` to suppress pop-up dialogs, e.g. for scheduled runs. |

## Scheduling a run

Use Windows Task Scheduler to run the console app. Sign in once with an API key so there is no
browser step:

```console
virtualdojo_sync_console login --api-key sk-xxxxxxxx
```

Then schedule, for example:

```console
C:\VirtualDojoSync\virtualdojo_sync_console.exe run to-quickbooks --commit
```

The task must run as the **same Windows user** that signed in (credentials are stored per user),
and QuickBooks must be able to open the company file for that user at that time.
Check the exit code ([table](troubleshooting.md#exit-codes)) to see whether a run needs attention.
