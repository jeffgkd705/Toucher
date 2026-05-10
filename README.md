# Toucher

A macOS Finder Quick Action that adds a **Touch** item to the right-click context menu, updating a file or directory's timestamps to the current time — the equivalent of running `touch` in the terminal.

## Why

Web browsers often download files with the server's original `Last-Modified` timestamp rather than the actual download time. This causes files to sort incorrectly in date-modified views. Toucher lets you fix this with a single right-click instead of opening a terminal.

## Usage

Right-click any file or folder in Finder → **Quick Actions** → **Touch**

The item's modification and access timestamps are updated to the current time. Directory contents are not recursively touched (same behavior as standard Unix `touch`).

## Installation

**Option 1 — Double-click:**
1. Download and unzip `Toucher.workflow.zip`
2. Double-click `Toucher.workflow` — Automator will offer to install it
3. Open **System Settings → Privacy & Security → Extensions → Finder** and ensure **Touch** is checked

**Option 2 — Terminal:**
```bash
cp -r Toucher.workflow ~/Library/Services/
/System/Library/CoreServices/pbs -update
killall Finder
```

**First-run:** macOS may prompt "Automator would like to access files and folders" — allow it once.

## Requirements

- macOS Catalina (10.15) or later
- No Xcode, code signing, or Apple Developer account required

## Compatibility

Verified working on macOS Tahoe 26.3.1 (M1).
