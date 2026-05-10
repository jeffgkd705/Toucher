# CLAUDE.md — Toucher

A macOS Finder Quick Action that adds a "Touch" item to the right-click context menu, updating a file or directory's timestamps via the equivalent of the Unix `touch` command.

## Approach: Automator Quick Action (.workflow bundle)

Chosen because:
- Can be built entirely on Linux (no Xcode, no macOS toolchain)
- No code signing or Apple Developer account required
- No sandboxing complications — shell script runs as the user with full file permissions
- Simple distribution: zip the `.workflow` bundle, user drops it in `~/Library/Services/`

Trade-off: The "Touch" item appears under the "Quick Actions" submenu in Finder, not at the top level of the context menu. Top-level placement requires a FinderSync extension (requires Xcode, macOS-only build).

## Project Structure

```
Toucher/
├── CLAUDE.md
└── Toucher.workflow/
    └── Contents/
        ├── Info.plist        # Registers action as a macOS Service scoped to Finder
        └── document.wflow    # Automator workflow document (embeds shell script)
```

## Key Files

### `Contents/Info.plist`
Registers the workflow with macOS Services. Critical fields:
- `NSMenuItem.default` → `"Touch"` — the displayed menu label
- `NSSendFileTypes` → `["public.item"]` — matches both files and directories
- `NSApplicationIdentifier` → `"com.apple.finder"` — scopes to Finder only (prevents it appearing in every app's Services menu)

### `Contents/document.wflow`
Automator document (XML plist). Contains a single "Run Shell Script" action:
```bash
for f in "$@"
do
    /usr/bin/touch "$f"
done
```
- `inputMethod: 1` — paths passed as `$@` arguments (not stdin)
- Shell: `/bin/bash`
- Paths are POSIX paths; quoting handles spaces in filenames

**Critical `workflowMetaData` fields** — these must all be present or macOS throws "There was a problem with the input to the Service" on invocation:
```xml
<key>serviceApplicationBundleID</key>
<string>com.apple.finder</string>
<key>serviceApplicationPath</key>
<string>/System/Library/CoreServices/Finder.app</string>
<key>serviceInputTypeIdentifier</key>
<string>com.apple.Automator.fileSystemObject</string>
<key>serviceOutputTypeIdentifier</key>
<string>com.apple.Automator.nothing</string>
<key>serviceProcessesInput</key>
<integer>0</integer>
<key>workflowTypeIdentifier</key>
<string>com.apple.Automator.servicesMenu</string>
```
Without `serviceInputTypeIdentifier` in particular, macOS cannot map Finder's file URL selection to the workflow's expected input type.

## Building and Packaging

On Linux:
```bash
zip -r Toucher.workflow.zip Toucher.workflow/
```

No compilation step. The `.workflow` is a plain directory of XML files.

## Installation on macOS

**Via Automator (GUI):**
1. Double-click `Toucher.workflow` — Automator offers to install it
2. Open System Settings > Privacy & Security > Extensions > Finder — ensure "Touch" is checked

**Via Terminal:**
```bash
cp -r Toucher.workflow ~/Library/Services/
/System/Library/CoreServices/pbs -update
killall Finder
```

**First-run behavior:** macOS may show a privacy prompt "Automator would like to access files and folders." — allow it once. If prompts recur, grant Full Disk Access to Automator in System Settings > Privacy & Security.

## Usage

Right-click any file or directory in Finder → Quick Actions → Touch

Updates the item's modification and access timestamps to the current time. Does NOT recursively touch directory contents (same as standard Unix `touch` behavior).

## Error Handling (Optional Enhancement)

To show a notification on failure (e.g., read-only file):
```bash
for f in "$@"; do
    /usr/bin/touch "$f" || osascript -e "display notification \"Could not touch: $f\" with title \"Toucher\""
done
```

## Compatibility

- macOS Catalina (10.15) and later; verified working on macOS Tahoe 26.3.1 (M1)
- No additional software required on target machine

## What Is NOT Needed

- Xcode
- Apple Developer account
- Code signing
- Notarization (for personal/local install)
