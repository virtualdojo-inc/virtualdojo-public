# VirtualDojo Public Downloads

Connectors, browser extensions, desktop apps, and guides for VirtualDojo customers.

Each product lives in its own folder with its own README and documentation. Start with the
one you need.

## Products

| Product | What it does | Start here |
|---|---|---|
| **QuickBooks Connector** | Syncs VirtualDojo with QuickBooks Desktop Enterprise. Includes a desktop app and a console (command-line) app. | [quickbooks-connector/](quickbooks-connector/README.md) |
| **Browser extensions** (Chrome, Edge, Safari) | Companion extension for VirtualDojo. | *Coming soon* |
| **Desktop AI client** | VirtualDojo AI assistant for your desktop. | *Coming soon* |

## QuickBooks Connector at a glance

- **Download:** [desktop app](https://github.com/virtualdojo-inc/virtualdojo-public/releases/download/quickbooks-connector-v0.0.1/virtualdojo_sync.exe) · [console app](https://github.com/virtualdojo-inc/virtualdojo-public/releases/download/quickbooks-connector-v0.0.1/virtualdojo_sync_console.exe) · [checksums](https://github.com/virtualdojo-inc/virtualdojo-public/releases/download/quickbooks-connector-v0.0.1/SHA256SUMS.txt)
- [Overview and installation](quickbooks-connector/README.md)
- [Console commands reference](quickbooks-connector/docs/console-commands.md) — every command and option
- [Where your files are stored](quickbooks-connector/docs/files-and-data.md) — configuration, field mappings, the local SQLite database, logs, reports, and saved credentials
- [Troubleshooting](quickbooks-connector/docs/troubleshooting.md) — exit codes and common problems

## Folder layout

```
virtualdojo-public/
├── README.md                  ← you are here
└── quickbooks-connector/
    ├── README.md              ← overview, requirements, install
    └── docs/
        ├── console-commands.md
        ├── files-and-data.md
        └── troubleshooting.md
```

Each new product gets its own top-level folder with the same shape: a `README.md` that explains
what it is and how to install it, and a `docs/` folder for the detailed guides.

## Getting help

Contact your VirtualDojo representative or open a support case at
[support.virtualdojo.com](https://support.virtualdojo.com).
