# Installing Claude Desktop on Fedora 44 KDE

**A manual installation guide for an officially unsupported platform, with integrity verification and a clean removal path.**

| | |
|---|---|
| **Target platform** | Fedora 44 Workstation, KDE Plasma 6 |
| **Architecture** | x86_64 (the arm64 route is identical, with one substitution) |
| **Upstream package** | `claude-desktop_amd64.deb`, Anthropic's official Linux beta |
| **Support status** | Unsupported by Anthropic on Fedora. This is a community procedure. |
| **Author** | NeumannTechTips |
| **Last validated** | 8 August 2026 |

---

## Why this guide exists

Anthropic shipped a Linux beta of Claude Desktop as a Debian package, distributed through its own apt repository. The official documentation is clear about the boundary: Ubuntu 22.04 or later and Debian 12 or later are supported, on x86_64 or arm64, and support for other distributions is described as coming in the future. Fedora and RHEL are named explicitly as not supported today.

That leaves Fedora users with three options. Wait for a native RPM. Use a third-party repackage. Or extract the official package and place its contents where a package manager would have put them.

This guide covers the third option, because it has one property the others lack: the bytes you install are Anthropic's own, unmodified, and you can prove it before anything touches your system. The trade-off is that `dnf` has no record of the result, so updates and removal become your responsibility rather than the package manager's.

If that trade-off does not appeal, two alternatives are worth considering first. A community project maintains RPM and AppImage repackages of the same official `.deb`, which restores update handling at the cost of trusting an intermediary. Alternatively, the Claude Code CLI has a native Fedora installer and an official RPM repository from Anthropic, and it runs the same engine as the Code tab in the desktop app. For terminal-centred work, the CLI may be all you need.

## Before you start

You will need roughly 1 GB of free space on the `/usr` filesystem, an active internet connection, and `sudo` rights. The application is Electron-based, so treat its memory appetite as comparable to a browser or a modern editor.

Everything below runs from a terminal. Konsole is fine.

---

## Step 1: Confirm your architecture

```bash
uname -m
```

`x86_64` means you want the amd64 package, which is what the rest of this guide assumes. If you see `aarch64`, substitute `arm64` wherever `amd64` appears. No other step changes.

## Step 2: Install dpkg

Fedora cannot install `.deb` packages, but it does ship the `dpkg` toolchain, which is all you need to read one:

```bash
sudo dnf install -y dpkg
```

This installs an extraction tool, not a second package manager. Nothing about your system's package management changes.

## Step 3: Create a working directory

Keep the download, the staging tree and the manifest together. This matters later, when you need to remove or update the application:

```bash
mkdir -p ~/claude-desktop-install && cd ~/claude-desktop-install
```

## Step 4: Download the package

The cleanest approach uses Anthropic's published command, which reads the repository index, finds the newest package for your architecture, and downloads it:

```bash
curl -fLO "https://downloads.claude.ai/claude-desktop/apt/stable/$(curl -s "https://downloads.claude.ai/claude-desktop/apt/stable/dists/stable/main/binary-$(dpkg --print-architecture)/Packages" | grep '^Filename: pool/main/c/claude-desktop/claude-desktop_' | sort -V | tail -n 1 | cut -d' ' -f2)"
```

If that command reports `Remote file name has no length`, the index lookup returned nothing. Confirm your network can reach `downloads.claude.ai` and that `dpkg --print-architecture` prints `amd64` or `arm64`.

You can also download through the browser from the Claude download page, though you will then need to locate the saved file yourself. Whichever route you take, confirm you have a real package before continuing:

```bash
ls -lh claude-desktop*.deb
file claude-desktop*.deb
```

Expect a file in the region of 160 MB, reported as a Debian binary package. If `file` says HTML, you have downloaded an error page rather than the application.

## Step 5: Verify what you downloaded

This step is the reason the manual route can be done responsibly, so please do not skip it.

When apt installs a package, it verifies a cryptographic chain on your behalf. The repository's release file is signed by Anthropic's key, that release file commits to the package index, and the index carries a SHA-256 digest for every package. Extracting a `.deb` by hand bypasses all of it. You are running 160 MB of code, much of it setuid-adjacent, on the strength of a URL. Reconstructing the checks by hand takes about a minute.

### Verify the signing key

```bash
curl -fsSLo claude-desktop-key.asc https://downloads.claude.ai/claude-desktop/key.asc
gpg --show-keys claude-desktop-key.asc
```

The fingerprint printed must be:

```
31DD DE24 DDFA B679 F42D  7BD2 BAA9 29FF 1A7E CACE
```

If it differs, stop. Do not extract the package. Anthropic publishes this fingerprint in its own installation documentation, which is what makes it usable as an independent reference point.

### Verify the package digest

```bash
ARCH=$(dpkg --print-architecture)
BASE="https://downloads.claude.ai/claude-desktop/apt/stable"
curl -fsSLo Packages "$BASE/dists/stable/main/binary-$ARCH/Packages"
awk '/^Package: claude-desktop$/,/^$/' Packages | grep -E '^(Version|Filename|SHA256):'
```

That prints the published digest for each version in the index. Now compute the digest of your download:

```bash
sha256sum claude-desktop*.deb
```

The two values must match. A mismatch means the file was corrupted in transit or is not the package the repository published, and either way it should not be installed.

### Verify the index itself, if you want the full chain

The digest check above is only as trustworthy as the index it came from. To close that gap, verify the repository's signed release file against the key you already checked:

```bash
gpg --no-default-keyring --keyring ./claude-trust.gpg --import claude-desktop-key.asc
curl -fsSLo InRelease "$BASE/dists/stable/InRelease"
gpg --no-default-keyring --keyring ./claude-trust.gpg --verify InRelease
```

This uses a local, throwaway keyring rather than importing an unrelated key into your personal one. A good signature from the Anthropic key completes the chain: signature to release file, release file to index, index to package.

One honest caveat. The `InRelease` path follows the standard Debian repository layout rather than a path published in Anthropic's documentation. If it returns a 404, try `dists/stable/Release` together with `dists/stable/Release.gpg` instead, which is the older equivalent. The digest check in the previous section remains valid either way, since its path is documented.

## Step 6: Extract the package

Extract as root. This matters more than it appears to:

```bash
sudo dpkg-deb -x claude-desktop*.deb extracted/
```

Running the extraction under `sudo` preserves the root ownership and permission bits recorded inside the archive, including the setuid bit on the Chromium sandbox helper. Extract as a normal user and every file becomes yours, which breaks the sandbox and forces you to repair permissions afterwards. Getting this right here saves two troubleshooting steps later.

## Step 7: Record an install manifest

`dnf` will not know these files exist, so write down what you are about to install before you install it:

```bash
sudo find extracted/usr \( -type f -o -type l \) | sed 's|^extracted||' | sudo tee installed-files.txt > /dev/null
wc -l installed-files.txt
```

Note the `-type l` clause. The application's launcher at `/usr/bin/claude-desktop` is a symbolic link, not a regular file, and a manifest built with `-type f` alone would silently omit it. That omission only surfaces months later, when an incomplete uninstall leaves a broken link behind.

## Step 8: Copy into place

```bash
sudo cp -a extracted/usr/. /usr/
```

The `-a` flag is doing real work here. It preserves ownership, permission bits and symbolic links exactly as extracted. A plain `cp` does neither of two things you need: it drops the setuid bit as a deliberate safety measure, and it applies your shell's umask to everything it creates. On a hardened Fedora install with a restrictive umask, that combination produces an application that either cannot start or cannot be read by your own user account. The troubleshooting section covers the symptoms if you have already hit this.

Confirm the result:

```bash
ls -la /usr/bin/claude-desktop
ls -la /usr/lib/claude-desktop/chrome-sandbox
```

The first should be a symbolic link pointing into `../lib/claude-desktop/`. The second must read `-rwsr-xr-x`, with an `s` in the owner-execute position. That `s` is the setuid bit, and Chromium's sandbox will not initialise without it.

## Step 9: Refresh the Plasma menu

KDE caches its list of installed applications, so a desktop entry placed directly on disk stays invisible until the cache is rebuilt:

```bash
kbuildsycoca6
```

On Plasma 5, use `kbuildsycoca5` instead. There is no output on success.

## Step 10: Launch

```bash
claude-desktop
```

Run it in the foreground the first time rather than backgrounding it with `&`, so that any startup error reaches your terminal instead of disappearing into job control.

`/usr/bin` is already on the `PATH` of every user on a standard Fedora system, so there is nothing to add to your shell profile and nothing that resets on reboot. The command works from any directory, in any new terminal, permanently.

You should now also find Claude in the Plasma application launcher, with its icon. Right-click it there to pin it to your task manager or favourites if you would rather not use a terminal at all.

Sign in with your Anthropic account. Chat is available on every plan including Free. Claude Code and Cowork require Pro, Max, Team or Enterprise.

---

## Desktop integration extras

Two optional packages fix issues that only appear on some setups.

If the system tray icon does not appear:

```bash
sudo dnf install -y libayatana-appindicator-gtk3
```

If you run a Wayland session and the Quick Entry global hotkey does nothing, it needs KDE's global shortcuts portal. Fedora's KDE spin normally ships this already, so check before installing:

```bash
sudo dnf install -y xdg-desktop-portal-kde
```

The hotkey works without extra configuration on X11.

---

## Troubleshooting

### `command not found` after a successful copy

The copy did not complete. It is easy to miss, because `cp` can fail partway with very little to say for itself. Re-run the copy verbosely and read the output:

```bash
cd ~/claude-desktop-install
sudo cp -av extracted/usr/. /usr/
```

Then confirm the symlink exists at `/usr/bin/claude-desktop`.

### `Permission denied` when listing files under `/usr/lib/claude-desktop`

Your umask stripped the world-execute bit from the directory when it was created. Reading a file requires execute permission on every directory above it, so a directory at `drwxr-x---` blocks your own user account from reaching anything inside.

Diagnose it precisely rather than guessing which directory is at fault:

```bash
umask
namei -l /usr/lib/claude-desktop/chrome-sandbox
```

`namei` prints the permissions of every component in the path, so the offending directory is obvious. A umask of `0027` is the usual culprit: it removes all "other" permissions from anything the process creates, and `sudo` inherits the umask of your interactive shell.

Since the umask only ever stripped the "other" bucket and left "group" alone, mirroring group onto other restores exactly what was lost:

```bash
sudo chmod -R o=g /usr/lib/claude-desktop
sudo chmod -R o=g /usr/share/doc/claude-desktop
sudo chmod o=g /usr/share/applications/com.anthropic.Claude.desktop
sudo find /usr/share/icons/hicolor -iname 'claude-desktop.png' -exec chmod o=g {} \;
```

To avoid this on future manual installs without changing your session's umask, scope a permissive one to the single command that needs it:

```bash
(umask 022 && sudo cp -a extracted/usr/. /usr/)
```

### The application crashes immediately on launch

Check the sandbox helper's permission bits:

```bash
ls -la /usr/lib/claude-desktop/chrome-sandbox
```

If the owner-execute position shows `x` rather than `s`, the setuid bit was stripped during the copy. Restore it:

```bash
sudo chown root:root /usr/lib/claude-desktop/chrome-sandbox
sudo chmod 4755 /usr/lib/claude-desktop/chrome-sandbox
```

Resist the temptation to work around this with `--no-sandbox`. That flag disables a genuine security boundary in a browser engine that renders remote content, which is a poor trade for saving one `chmod`.

### `Broken pipe` from `dpkg-deb`

Harmless. If you piped `dpkg-deb -c` into `head`, the pipe closed once `head` had its lines while `dpkg-deb` was still writing. Nothing failed.

### Nothing in `~/Downloads`

The browser saved it elsewhere. Search rather than guess:

```bash
find ~ -maxdepth 3 -iname "*claude-desktop*.deb" 2>/dev/null
```

---

## Updating

Nothing updates automatically on this path. The desktop app has no built-in updater on Linux, and `dnf` does not know the package exists.

When a new version is released, repeat steps 4 through 9 with the fresh package. The copy overwrites the previous files in place. Re-run the integrity verification each time, because a signing key and a digest that were correct last month tell you nothing about this month's download.

## Removing

The manifest you wrote in step 7 turns removal from guesswork into an exact operation:

```bash
cd ~/claude-desktop-install
sudo xargs -a installed-files.txt rm -v
sudo rm -rf /usr/lib/claude-desktop /usr/share/doc/claude-desktop
kbuildsycoca6
```

Review `installed-files.txt` before running this. It drives `rm` with `sudo`, and a manifest generated against the wrong directory would do real damage.

---

## Known limitations of the Linux beta

Several features present on macOS and Windows are absent on Linux, independently of anything Fedora-specific:

- Computer Use, meaning application and screen control, is not available.
- Voice dictation is not available in the desktop app. The CLI supports it.
- The Quick Entry global hotkey works on X11. Under native Wayland it depends on the desktop environment's GlobalShortcuts portal.

Chat, Cowork and Claude Code are all present in the Linux beta.

On top of those, this installation method adds two of its own. You lose automatic updates, and you lose the package manager's dependency resolution, which means a future release depending on a newer system library will fail at runtime rather than being caught at install time.

---

## Reference architecture

`claude-desktop-fedora-install-architecture.drawio` accompanies this guide. It is a Type 1 static architecture view of the installation, structured across six layers: upstream distribution, acquisition and integrity verification, staging and extraction, target filesystem, hardening and desktop integration, and the lifecycle control plane.

Connector line style carries the interaction pattern. Solid lines are synchronous, dotted lines are bulk or file transfer, and grey dashed lines are the control plane. Each connector carries a reference from F1 to F15, resolved in the flow-reference table on the diagram itself.

Open it in [diagrams.net](https://app.diagrams.net), the draw.io desktop application, or the VS Code extension. Export to PNG or PDF for printing.

---

## Sources

- [Claude Desktop on Linux (beta)](https://code.claude.com/docs/en/desktop-linux), Anthropic. Supported distributions, repository configuration, signing key fingerprint, package pool lookup, and the feature gaps in the Linux beta.
- [Download Claude](https://claude.ai/download), Anthropic. Platform availability and plan requirements.
- [claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian). Community repackaging of the official Linux `.deb` into RPM, AppImage and Nix formats, for readers who would prefer a managed package over this manual route.

---

## Disclaimer

Fedora is not a supported platform for Claude Desktop. This procedure installs an official, unmodified Anthropic package by hand and is offered as a community contribution. It carries no warranty, and it is not endorsed by Anthropic. Verify the package before installing it, and keep the manifest so you can reverse the installation cleanly.

Anthropic's documentation states that support for additional distributions is planned. When a native RPM arrives, migrate to it and remove this installation using the steps above.
