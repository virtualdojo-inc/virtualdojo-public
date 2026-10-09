# VirtualDojo Browser Extension

A companion extension for **Google Chrome** and **Microsoft Edge** that brings government
procurement opportunities straight into your VirtualDojo account.

- **Import** — on GSA eBuy, NASA SEWP, Army CHESS, SAM.gov, and Unison Marketplace, adds
  one-click **Import** (and bulk **Import All**) so RFQs and RFIs, including attachments, due
  dates, and points of contact, become VirtualDojo records without manual re-entry.
- **Side panel** — search and look up VirtualDojo records from the toolbar, or right-click
  selected text and choose **Search VirtualDojo**.
- **Field Inspector** (VirtualDojo admins) — record highlights, a full field viewer and editor,
  data export, and data import on VirtualDojo record pages.

You stay signed in through VirtualDojo as usual. The extension acts only on your behalf and
never collects usernames or passwords. It runs only on VirtualDojo and the supported
procurement portals.

## Download

Download the newest **Browser Extension** release from the
[releases page](https://github.com/virtualdojo-inc/virtualdojo-public/releases?q=browser-extension&expanded=true).
Each release has these files under **Assets**:

| File | Use it for |
|---|---|
| `virtualdojo-extension-<version>-chrome.zip` | Google Chrome |
| `virtualdojo-extension-<version>-edge.zip` | Microsoft Edge |
| `SHA256SUMS.txt` | Checksums |

No sign-in is needed. The current version, with direct links to its files, is listed in
[`latest.json`](latest.json).

To confirm your download is intact, compare its hash with the line for that file in
`SHA256SUMS.txt`:

```powershell
Get-FileHash .\virtualdojo-extension-0.39.0-chrome.zip -Algorithm SHA256
```

```console
shasum -a 256 virtualdojo-extension-0.39.0-chrome.zip
```

## Install

1. **Unzip** the download into a folder you will keep, for example
   `Documents\VirtualDojo Extension`. The browser loads the extension from this folder, so do not
   delete or move it afterwards.
2. Open the extensions page: `chrome://extensions` in Chrome, or `edge://extensions` in Edge.
3. Turn on **Developer mode** (top-right in Chrome, left sidebar in Edge).
4. Click **Load unpacked** and select the folder you unzipped (the one that contains
   `manifest.json`).
5. Optional: click the puzzle-piece icon in the toolbar and pin **VirtualDojo**.

Then sign in to VirtualDojo at [app.virtualdojo.com](https://app.virtualdojo.com) in the same
browser. The extension picks up your session from there.

> **"Disable developer mode extensions" prompt?** Chrome and Edge show this for any extension
> not installed from a store. Click **Cancel** (or close the prompt) to keep VirtualDojo
> enabled.

## Update

An extension installed this way does **not** update itself. To move to a new version:

1. Download the new zip and unzip it **over the same folder**, replacing the files.
2. On the extensions page, click the reload icon on the **VirtualDojo** card.

Your sign-in and settings are kept.

## Getting help

Contact your VirtualDojo representative or open a support case at
[support.virtualdojo.com](https://support.virtualdojo.com).
