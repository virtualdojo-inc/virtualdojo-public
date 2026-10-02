# Troubleshooting

Start with:

```console
virtualdojo_sync_console doctor --deep
```

It reports your configuration folder, sign-in, mapping, and whether it can open a QuickBooks
session, and tells you what is wrong in plain language.

## Exit codes

The console app returns a number when it finishes. `0` always means success. Use it in scripts
and Task Scheduler to decide whether a run needs a look.

| Code | Meaning | What to do |
|---|---|---|
| `0` | Finished cleanly | Nothing |
| `1` | A run was aborted, or `doctor` found a problem | Read the message printed; check `logs\sync.log` |
| `2` | A run finished, but something needs attention | Read the report printed at the end and the `failures-….csv` it names |
| `10` | Configuration or mapping problem (for example no mapping yet) | Run `config --init`, or fix what the message names |
| `11` | Not signed in, or sign-in failed | Run `login` |
| `12` | VirtualDojo returned an error | Check the message; try again; contact support if it persists |
| `13` | VirtualDojo's daily request limit for your account was reached | Wait for the limit to reset; do not retry repeatedly |
| `14` | Mapping problems found — **nothing was written** | Fix the listed problems, then run again |
| `15` | The local database needs attention | Run `state` |
| `16` | The connector cannot tell whether an interrupted write reached QuickBooks, so it stopped | Check QuickBooks, then resolve with `state --mark-synced` or `--mark-failed` |
| `20` | A QuickBooks error | Read the message |
| `21` | Could not open QuickBooks or start a session | See [QuickBooks will not connect](#quickbooks-will-not-connect) |
| `22` | QuickBooks rejected a request | Read the message; QuickBooks' own reason is included |
| `23` | A request was malformed, or invoice lines did not add up to the total | Send `logs\sync.log` to support |
| `64` | The command line was not understood | Check the spelling and options; use `--help` |

## QuickBooks will not connect

- **QuickBooks and the connector must be on the same computer**, and Windows must be able to open
  the company file.
- **Use a backslash path.** QuickBooks rejects company file paths written with forward slashes
  (`C:/Data/Co.QBW`). Use `C:\Data\Co.QBW`. The connector converts slashes for you, but
  the error QuickBooks gives for this mistake ("Could not start QuickBooks") does not mention the path.
- **First-time approval needs an administrator in single-user mode.** If you never saw the
  QuickBooks permission prompt, or someone clicked No, sign in to QuickBooks as an administrator
  in single-user mode and run `doctor --deep` again.
- **The prompt came back after an update.** QuickBooks appears to remember its approval by the
  program's location. Keep the `.exe` files in one permanent folder and replace them in place
  when you update.
- **"Without a certificate" or an unknown-publisher warning.** The programs are not yet
  code-signed. Check the SHA-256 hash against `SHA256SUMS.txt` from the release you downloaded.

## "Not signed in" (exit code 11)

Run `virtualdojo_sync_console login`. Scheduled tasks must run as the same Windows user that
signed in, because the saved credentials are per user. Using a different server with
`config --server` clears credentials on purpose, so sign in again afterward.

## `doctor` says it cannot verify an account

The mapping names accounts (income, expense, tax, A/R) that the connector cannot find in a cached
copy of your chart of accounts. Either the chart has not been loaded yet, or the name is not an
exact match for an account in your company file. Run `doctor --deep` with QuickBooks reachable,
then set the right name with `config --set NAME="Parent:Child Account"`. Use QuickBooks'
full account name, including parent accounts and colons.

## A run stopped and mentions an "in-flight" record

The connector writes a note before sending each record to QuickBooks. If it was interrupted,
the next run looks in QuickBooks to see whether the record arrived. If it cannot tell, it stops
(exit code `16`) rather than risk a duplicate.

1. Run `virtualdojo_sync_console state` to list the records in question.
2. Look for each one in QuickBooks.
3. If it is there: `state --mark-synced OBJECT/ID --qb-id <TxnID or ListID>`.
4. If it is not: `state --mark-failed OBJECT/ID`, and the next run sends it again.

## Records you expected did not sync

- Check the mapping's **filter**: `config --show` lists them (for example invoices with status
  `Issued`, `Sent`, or `Paid`).
- Check the **record cap**: a run writes at most 100 records per object by default. Raise it with
  `config --max-records N`.
- Make sure you ran with `--commit`. Without it a run is Test Mode and writes nothing.
- Check that the mapping is **on**: `config --show` marks each as `[on ]` or `[off]`. Turn one on
  with `mapping enable ID`.

## Collecting information for support

1. `virtualdojo_sync_console --version`
2. `virtualdojo_sync_console doctor`
3. The newest `logs\sync.log` (see [Files and data](files-and-data.md#logs))
4. The `failures-….csv` from the affected run, if there is one

Review these first: they contain names and amounts from your books. Do not send the `.sqlite`
database, `QBEC_TRACE_XML` traces, or `dump` files unless support asks for them.

Support: [support.virtualdojo.com](https://support.virtualdojo.com)
