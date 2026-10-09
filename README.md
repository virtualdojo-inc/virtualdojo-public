# VirtualDojo Public Downloads

Connectors, browser extensions, desktop apps, and guides for VirtualDojo customers.

Each product lives in its own folder with its own README and documentation. Start with the
one you need.

## Products

| Product | What it does | Start here |
|---|---|---|
| **QuickBooks Connector** | Syncs VirtualDojo with QuickBooks Desktop Enterprise. Includes a desktop app and a console (command-line) app. | [quickbooks-connector/](quickbooks-connector/README.md) |
| **Browser Extension** (Chrome, Edge) | Imports opportunities from eBuy, SEWP, CHESS, SAM.gov and Unison into VirtualDojo, plus a search side panel. | [browser-extension/](browser-extension/README.md) |
| **Desktop AI client** | VirtualDojo AI assistant for your desktop. | *Coming soon* |

## QuickBooks Connector at a glance

- **Download:** [QuickBooks Connector releases](https://github.com/virtualdojo-inc/virtualdojo-public/releases?q=quickbooks-connector&expanded=true) — the desktop app, the console app and `SHA256SUMS.txt` are under each release's **Assets**
- [Overview and installation](quickbooks-connector/README.md)
- [Console commands reference](quickbooks-connector/docs/console-commands.md) — every command and option
- [Where your files are stored](quickbooks-connector/docs/files-and-data.md) — configuration, field mappings, the local SQLite database, logs, reports, and saved credentials
- [Troubleshooting](quickbooks-connector/docs/troubleshooting.md) — exit codes and common problems

## Browser Extension at a glance

- **Download:** [Browser Extension releases](https://github.com/virtualdojo-inc/virtualdojo-public/releases?q=browser-extension&expanded=true) — the Chrome and Edge zips and `SHA256SUMS.txt` are under each release's **Assets**
- [Overview, installation and updating](browser-extension/README.md)

## Folder layout

```
virtualdojo-public/
├── README.md                  ← you are here
├── browser-extension/
│   ├── README.md              ← overview, download, install, update
│   └── latest.json            ← the current version and its download links
└── quickbooks-connector/
    ├── README.md              ← overview, requirements, install
    ├── latest.json            ← the current version (the apps' update check reads it)
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
