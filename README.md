<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/NeumannTechTips_Lockup_Horizontal_Tagline_OnDark.png">
  <img alt="NeumannTechTips. Practical AI. Real results." src="assets/NeumannTechTips_Lockup_Horizontal_Tagline_OnLight.png" width="560">
</picture>

</div>

# 🐧 Claude Desktop on Fedora 45 KDE Plasma

> A manual installation guide for an officially unsupported platform. You install Anthropic's own, unmodified Linux package, prove its integrity through the full signature chain before anything touches `/usr`, and keep an exact record of every file so that updates and removal stay clean.

[![Fedora](https://img.shields.io/badge/Fedora-45%20KDE%20Plasma-5DBBD6?labelColor=0B2039&logo=fedora&logoColor=white)](#-validated-configuration)
[![KDE Plasma](https://img.shields.io/badge/KDE%20Plasma-6.8%20Wayland-5DBBD6?labelColor=0B2039&logo=kde&logoColor=white)](#-running-under-plasma-68)
[![Claude Desktop](https://img.shields.io/badge/Claude%20Desktop-5.11492.5%20validated-F2A93B?labelColor=0B2039)](#-validated-configuration)
[![Cowork](https://img.shields.io/badge/Cowork-working%20on%20KVM-F2A93B?labelColor=0B2039)](#-cowork-on-fedora-45)
[![SELinux](https://img.shields.io/badge/SELinux-enforcing-2F5F7B?labelColor=0B2039)](#step-8-copy-into-place-label-and-register)
[![Support](https://img.shields.io/badge/Anthropic%20support-Debian%20%26%20Ubuntu%20only-8FA3B8?labelColor=0B2039)](#-disclaimer)
[![Licence](https://img.shields.io/badge/licence-MIT-2F5F7B?labelColor=0B2039)](#-licence)
[![Language](https://img.shields.io/badge/language-UK%20English-2F5F7B?labelColor=0B2039)](#-licence)

Maintained by **[NeumannTechTips](https://github.com/NeumannTechTips)** · YouTube [@NeumannTechTips](https://www.youtube.com/@NeumannTechTips) · Last validated: 14 November 2026

`#Fedora45` `#FedoraKDE` `#KDEPlasma` `#Plasma6` `#Wayland` `#XWayland` `#Linux` `#ClaudeDesktop` `#Claude` `#Anthropic` `#Cowork` `#ClaudeCode` `#SELinux` `#KVM` `#PracticalAI` `#NeumannTechTips`

---

## 📖 Table of contents

- [At a glance](#-at-a-glance)
- [Why this guide exists](#-why-this-guide-exists)
- [Choosing a route](#-choosing-a-route)
- [Validated configuration](#-validated-configuration)
- [Before you start](#-before-you-start)
- [Installation](#-installation)
- [Running under Plasma 6.8](#-running-under-plasma-68)
- [Cowork on Fedora 45](#-cowork-on-fedora-45)
- [Updating](#-updating)
- [Removing](#-removing)
- [Troubleshooting](#-troubleshooting)
- [Known limitations](#-known-limitations)
- [Reference architecture](#-reference-architecture)
- [Repository contents](#-repository-contents)
- [Sources](#-sources)
- [Disclaimer](#-disclaimer)
- [Related repositories](#-related-repositories)
- [Licence](#-licence)

---

## 📋 At a glance

| | |
|---|---|
| **Target platform** | Fedora 45 KDE Plasma Desktop Edition, KDE Plasma 6.8 |
| **Architecture** | x86_64 validated. The arm64 package follows the same steps but has not been tested |
| **Upstream package** | `claude-desktop_<version>_amd64.deb` from Anthropic's signed apt repository |
| **Validated release** | Claude Desktop 5.11492.5 |
| **Support status** | Not supported by Anthropic on Fedora. This is a community procedure |
| **Earlier edition** | Fedora 44. The instructions below were not retested on Fedora 44 in this round |

---

## 🎯 Why this guide exists

Anthropic ships the Linux beta of Claude Desktop as a Debian package through its own apt repository. Its documentation supports Ubuntu 22.04 or later and Debian 12 or later, on x86_64 or arm64, and names Fedora and RHEL as distributions that are not supported today, with support for others described as coming in the future.

Fedora users therefore have three choices: wait for an official RPM, trust a community repackage, or place the contents of the official package where a package manager would have put them. This guide documents the third choice. Its defining property is provenance: the bytes you run are Anthropic's own, and you can prove that before installing anything. The cost is that `dnf` has no record of the result, so updates and removal become your responsibility.

> [!NOTE]
> Fedora 45 changed the ground under the earlier edition of this guide. Plasma 6.8 no longer offers an X11 session, GnuPG's default key store ignores the `--keyring` option the old verification step relied on, and Claude Desktop itself moved through three major versions. Every section below reflects those changes.

---

## 🔀 Choosing a route

| Route | What you install | Updates | Trust model |
|---|---|---|---|
| **This guide** | Anthropic's official `.deb`, extracted by hand | Manual, using the procedure in [Updating](#-updating) | Anthropic's signing key only |
| **Community repackage** | [`claude-desktop-unofficial`](https://github.com/aaddrick/claude-desktop-debian) RPM, AppImage or Nix build | Through `dnf` | Anthropic's package plus the community project's build and signing |
| **Claude Code CLI only** | Anthropic's `claude-code` package from its signed dnf repository | Through `dnf` | Anthropic's signing key only |

The community project rebased onto Anthropic's official Linux app in its v3.0.0 release. It now installs under `/usr/lib/claude-desktop-unofficial`, adds a diagnostic launcher, and serves a dnf repository. If you would rather have managed updates than first-hand provenance, it is the sensible alternative.

For terminal-centred work, the CLI may be all you need. Anthropic publishes signed apt, dnf and apk repositories for it, and it runs the same engine as the Code tab in the desktop app.

---

## ✅ Validated configuration

Every command in this guide was run on the machine below on 14 November 2026, unless the text says otherwise.

| Component | Observed on the test machine |
|---|---|
| Release | Fedora 45 KDE Plasma Desktop Edition, upgraded in place from Fedora 44 |
| Desktop | KDE Plasma 6.8.2, Wayland session |
| Kernel | 7.3.3-300.fc45 |
| Graphics | NVIDIA proprietary driver 620.70.06 |
| SELinux | Enforcing, with `auditd` active |
| Claude Desktop | 5.11492.5, updated from 2.42304.2 |
| Display path | XWayland, through `--ozone-platform=x11` |
| Integrity chain | Key fingerprint, `InRelease` signature, index hash and package SHA-256 all passed |
| Installed tree | Byte-for-byte identical to the package, confirmed with `diff -rq` |
| Sandbox helper | `chrome-sandbox` at mode 4755, owned by root, labelled `lib_t` |
| Sign-in | Persisted across a restart through KDE Wallet |
| Cowork | Started without any requirements message |

> [!IMPORTANT]
> **Not yet tested:** a first-time install on a freshly installed Fedora 45 system, native Wayland without the XWayland flag, arm64, the Quick Entry global hotkey, and Cowork with only the bundled `virtiofsd`. The update sequence in this guide also differs from the one run on the test machine in one deliberate respect, explained in [Updating](#-updating).

---

## 🧰 Before you start

You need about 1 GB free on the filesystem holding `/usr`, a working internet connection, and `sudo` rights. Claude Desktop is an Electron application, so expect a memory footprint similar to a browser. Cowork adds a virtual machine on top of that.

Run every step from Konsole, in **one terminal session**. Later steps reuse shell variables that earlier steps set, such as the version number and the staging path.

> [!WARNING]
> Many hardened Fedora systems set a umask of `0027`, which leaves files created as root unreadable by ordinary users. Step 3 sets a permissive umask for the installing terminal only. Close that terminal when you finish so your normal setting returns.

---

## 🔧 Installation

### Step 1: Confirm your architecture

```bash
uname -m
```

`x86_64` means the amd64 package, which the rest of this guide assumes. On `aarch64`, the commands derive the right architecture automatically, but that path is untested.

### Step 2: Install the tools

Fedora cannot install `.deb` packages, but it does ship the `dpkg` toolchain, which can read one. The other packages provide the cache and registration tools used in Step 8. On a standard Fedora KDE system most of them are already present.

```bash
sudo dnf install dpkg curl gnupg2 shared-mime-info desktop-file-utils gtk-update-icon-cache
```

This adds an extraction tool, not a second package manager. Nothing about how `dnf` manages your system changes.

### Step 3: Prepare a working folder and a permanent record folder

```bash
umask 022
mkdir -p ~/claude-desktop-install ~/.local/share/claude-desktop-fedora
cd ~/claude-desktop-install
```

The working folder holds downloads and can be emptied at any time. The record folder holds the install manifests and must survive. The earlier edition kept its manifest beside the downloads, and on the test machine that file was both misplaced and out of date by the time it was needed.

### Step 4: Verify the signing key and the repository index

Verification comes before the download, so that the version you choose is taken from an index you have already proven.

When apt installs a package, it checks a chain of trust on your behalf: Anthropic's key signs the release file, the release file records the hash of the package index, and the index records the SHA-256 digest of each package. Extracting a `.deb` by hand bypasses all of that, so this step rebuilds the chain link by link.

**Check the key.**

```bash
BASE="https://downloads.claude.ai/claude-desktop/apt/stable"
ARCH=$(dpkg --print-architecture)
curl -fsSLo claude-desktop-key.asc https://downloads.claude.ai/claude-desktop/key.asc
gpg --show-keys claude-desktop-key.asc
```

The fingerprint must read:

```
31DD DE24 DDFA B679 F42D  7BD2 BAA9 29FF 1A7E CACE
```

If it differs, stop. Anthropic publishes this fingerprint in its own installation documentation, which makes it an independent reference point.

**Check the release file's signature in a throwaway key store.**

```bash
export GNUPGHOME=$(mktemp -d)
gpg --import claude-desktop-key.asc
curl -fsSLo InRelease "$BASE/dists/stable/InRelease"
gpg --verify InRelease
gpgconf --kill all
rm -rf "$GNUPGHOME"; unset GNUPGHOME
```

Expect `Good signature from "Anthropic Claude Code Release Signing"`. The accompanying warning that the key "is not certified with a trusted signature" is normal: it only means nobody has signed this key into your personal web of trust, and the fingerprint check above covers that gap.

> [!TIP]
> The temporary `GNUPGHOME` matters on Fedora 45. GnuPG now uses the `keyboxd` key store by default, and in that mode it ignores `--keyring` and `--no-default-keyring`. The earlier edition's "throwaway keyring" command therefore imported Anthropic's key into your personal store. A separate home directory is the isolation method GnuPG's own maintainers recommend.

**Tie the package index to the signed release file.**

```bash
curl -fsSLo Packages "$BASE/dists/stable/main/binary-$ARCH/Packages"
grep -c "$(sha256sum Packages | cut -d' ' -f1)" InRelease
```

A count of `1` or more proves that the index you downloaded is the one Anthropic signed.

### Step 5: Choose the newest version and download it

```bash
VER=$(awk '/^Package: claude-desktop$/,/^$/' Packages | awk '/^Version:/{print $2}' | sort -V | tail -n 1)
read -r FILE SHA < <(awk -v v="$VER" '
  /^Package: /  {p=$2}
  /^Version: /  {ver=$2}
  /^Filename: / {f=$2}
  /^SHA256: /   {s=$2}
  /^$/ { if (p=="claude-desktop" && ver==v) print f, s; p=ver=f=s="" }
  END  { if (p=="claude-desktop" && ver==v) print f, s }' Packages)
echo "Newest: $VER"

curl -fLO "$BASE/$FILE"
DEB=$(basename "$FILE")
echo "$SHA  $DEB" | sha256sum -c -
```

The last line must print `OK`. A mismatch means the file was corrupted in transit or is not the package Anthropic published, and either way it must not be installed. Version 5.11492.5 is 183,335,272 bytes.

Optionally, confirm that the package writes nothing outside `/usr`. This command should print only `./`:

```bash
dpkg-deb -c "$DEB" | awk '{print $6}' | grep -v '^\./usr/'
```

### Step 6: Extract into a fresh staging folder

```bash
STAGE=$(sudo mktemp -d /var/tmp/claude-desktop-stage.XXXXXX)
sudo dpkg-deb -x "$DEB" "$STAGE"
sudo rm -rf "$STAGE/usr/share/lintian"
sudo ls -l "$STAGE/usr/lib/claude-desktop/chrome-sandbox"
```

Extracting as root preserves the ownership and permission bits recorded in the archive, above all the setuid bit on Chromium's sandbox helper, which must read `-rwsr-xr-x`. A **new, empty** staging folder matters as much: extracting a new version over an old one merges the two trees and carries obsolete files forward. The `lintian` file is Debian packaging metadata with no use on Fedora.

### Step 7: Record the install manifest

```bash
MANIFEST=~/.local/share/claude-desktop-fedora/installed-files-$VER.txt
sudo find "$STAGE/usr" \( -type f -o -type l \) | sed "s|^$STAGE||" | sort > "$MANIFEST"
echo /usr/share/mime/packages/com.anthropic.Claude.xml >> "$MANIFEST"
wc -l "$MANIFEST"
```

Version 5.11492.5 produces 100 entries. The file is owned by you and named after the version, so each update leaves an exact record of what it installed. The `-type l` clause captures the launcher at `/usr/bin/claude-desktop`, which is a symbolic link. The manifest from the earlier edition on the test machine had lost that entry, which would have left a dangling link behind after removal.

### Step 8: Copy into place, label and register

```bash
sudo tar -C "$STAGE" -cf - usr | sudo tar -C / -xpf - --no-overwrite-dir

sudo install -Dm644 /usr/lib/claude-desktop/resources/linux-mime/com.anthropic.Claude.xml \
  /usr/share/mime/packages/com.anthropic.Claude.xml
sudo update-mime-database /usr/share/mime
sudo update-desktop-database /usr/share/applications
sudo gtk-update-icon-cache -qf /usr/share/icons/hicolor

sudo restorecon -RF /usr/lib/claude-desktop
xargs -a "$MANIFEST" sudo restorecon -F
sudo rm -rf "$STAGE"
```

Each block does a specific job:

| Command | Why it is needed |
|---|---|
| `tar -p` with `--no-overwrite-dir` | Preserves ownership, permission bits, the setuid bit and symbolic links exactly as extracted, and leaves the metadata of existing directories such as `/usr/share` untouched |
| `install` and `update-mime-database` | Registers the `.mcpb`, `.dxt` and `.skill` file types, which the package's Debian install script would otherwise have done |
| `update-desktop-database` and `gtk-update-icon-cache` | Refreshes the system's desktop entry and icon caches |
| `restorecon` | Gives the new files the SELinux labels Fedora's policy expects for their location |

> [!NOTE]
> The earlier edition used `cp -a`. That flag preserves every attribute it can, SELinux labels included, so it carries the staging folder's labels into `/usr`. It can also re-apply the staging folder's own permissions and timestamps to existing system directories. The `tar` pipeline avoids both, and it is the method used on the test machine.

Confirm the result:

```bash
ls -l /usr/bin/claude-desktop
ls -lZ /usr/lib/claude-desktop/chrome-sandbox
```

The first must be a symbolic link into `../lib/claude-desktop/`. The second must read `-rwsr-xr-x. 1 root root system_u:object_r:lib_t:s0`.

### Step 9: Refresh the Plasma menu

```bash
kbuildsycoca6
```

KDE caches its list of applications, so a desktop entry placed directly on disk stays invisible until the cache is rebuilt. There is no output on success.

### Step 10: Launch

```bash
claude-desktop --ozone-platform=x11
```

Run it in the foreground the first time so that any start-up error reaches your terminal. The flag is explained in the next section, which also shows how to make it permanent.

Sign in with your Anthropic account. Chat is available on every plan, including Free. Claude Code and Cowork require a paid plan: Pro, Max, Team or Enterprise.

---

## 💻 Running under Plasma 6.8

Plasma 6.8 is Wayland-only. The X11 session has been removed from the login screen, and applications written for X11 now run through the XWayland compatibility layer. On Fedora 45, `echo $XDG_SESSION_TYPE` therefore prints `wayland` for every Plasma login.

The `--ozone-platform=x11` flag tells Claude Desktop's Chromium engine to run as an XWayland client instead of a native Wayland one. That is the configuration validated on the test machine. The earlier edition adopted it to avoid rendering problems under native Wayland on Fedora 44, and native Wayland has not been retested on Fedora 45.

To apply the flag from the application launcher and the desktop actions as well, create a per-user copy of the desktop entry. Never edit Anthropic's own file: every update overwrites it.

```bash
mkdir -p ~/.local/share/applications
sed -E 's/^Exec=claude-desktop /Exec=claude-desktop --ozone-platform=x11 /' \
  /usr/share/applications/com.anthropic.Claude.desktop \
  > ~/.local/share/applications/com.anthropic.Claude.desktop
update-desktop-database ~/.local/share/applications
kbuildsycoca6
grep '^Exec=' ~/.local/share/applications/com.anthropic.Claude.desktop
```

Version 5.11492.5 has three `Exec=` lines, the main entry and two desktop actions, and every one should carry the flag.

> [!CAUTION]
> Your copy takes precedence over the system entry, so it also hides anything a later version adds to that entry, such as new file-type handlers or menu actions. Regenerate it after every update with the same commands. [Updating](#-updating) includes this step.

| Desktop integration | State on the test machine |
|---|---|
| System tray icon | Working, with `libayatana-appindicator-gtk3` installed |
| KDE portal (`xdg-desktop-portal-kde`) | Installed by default on Fedora 45 KDE |
| Quick Entry global hotkey | Not tested. Anthropic documents it as working on X11, and as depending on the desktop's GlobalShortcuts portal under native Wayland |

---

## 🧪 Cowork on Fedora 45

Cowork runs its longer agentic tasks inside a virtual machine that Claude Desktop hosts with QEMU and KVM. The package bundles some of what that needs, and the host must supply the rest.

| Requirement | On Fedora 45 | State on the test machine |
|---|---|---|
| Hardware virtualisation | Enable VT-x or AMD-V in firmware | `vmx` flag present, `kvm_intel` loaded |
| QEMU | `qemu-system-x86-core` | Installed |
| UEFI firmware | `edk2-ovmf` | Installed. The package itself provides Debian-style links under `/usr/share/OVMF` |
| `virtiofsd` | Bundled at `/usr/lib/claude-desktop/resources/virtiofsd`. Fedora's `virtiofsd` package is optional | Both present during the test |
| `/dev/kvm` | Membership of the `kvm` group | Group member. Device was `root:kvm` with mode `0666` |
| `/dev/vhost-vsock` | `vhost_vsock` kernel module | Module present in the kernel tree, device node present at boot |

Check your system in one pass. These commands only read:

```bash
grep -m1 -oE 'vmx|svm' /proc/cpuinfo || echo "no virtualisation flag: enable it in firmware"
rpm -q qemu-system-x86-core edk2-ovmf
ls -l /dev/kvm /dev/vhost-vsock
id -nG | tr ' ' '\n' | grep -x kvm || echo "not in the kvm group"
modinfo -n vhost_vsock
ls -l /usr/share/OVMF/
```

Fix anything missing:

```bash
sudo dnf install qemu-system-x86-core edk2-ovmf
sudo usermod -aG kvm "$USER"
```

Log out and back in after joining the group. Anthropic's documentation asks for group membership even where `/dev/kvm` already opens, because `/dev/vhost-vsock` is normally restricted to the `kvm` group. If Cowork reports that it requires the `vhost_vsock` module, load it now and on every boot:

```bash
sudo modprobe vhost_vsock
echo vhost_vsock | sudo tee /etc/modules-load.d/vhost_vsock.conf
```

Claude Desktop checks these requirements once, at launch, so restart it after any change.

> [!NOTE]
> Cowork's firmware lookup expects Debian's `/usr/share/OVMF` path. On Fedora 45, `edk2-ovmf` (version `20260812-10.fc45` on the test machine) already ships that path as links into `/usr/share/edk2/ovmf`, so no manual symlink is needed. The community repackage adds one of its own for distributions that lack it.

---

## 🔄 Updating

The desktop app does not update itself on Linux, and `dnf` does not know it exists. Check the index for a newer version whenever you like by repeating Steps 3 to 5. The newest version is printed after `Newest:`, and you can compare it with your latest manifest:

```bash
ls ~/.local/share/claude-desktop-fedora/
```

To update:

1. **Quit Claude Desktop completely**, including from the system tray, then confirm nothing is left running:

   ```bash
   pgrep -f '^/usr/lib/claude-desktop/claude-desktop' && echo "still running" || echo "stopped"
   ```

2. **Decide about your settings.** A major version may migrate `~/.config/Claude` into a form older builds cannot read, so back it up now if you want a true rollback point. Check its size first, because Cowork data makes it large: it was 1.5 GB on the test machine.

   ```bash
   du -sh ~/.config/Claude
   cp -a ~/.config/Claude ~/.config/Claude.bak-before-update
   ```

3. **Verify, download, stage and record** by repeating Steps 3 to 7. Keep the previous manifest's path for step 6 of this list:

   ```bash
   OLD=$(ls ~/.local/share/claude-desktop-fedora/installed-files-*.txt | sort -V | tail -n 2 | head -n 1)
   echo "$OLD"
   ```

4. **Move the current version aside**, so that a failed copy can be rolled back:

   ```bash
   sudo mv /usr/lib/claude-desktop /usr/lib/claude-desktop.prev
   ```

5. **Install** by repeating Steps 8 and 9.

6. **List files the old version placed outside `/usr/lib/claude-desktop` that the new version no longer ships:**

   ```bash
   comm -23 <(grep -v '^/usr/lib/claude-desktop/' "$OLD" | sort) \
            <(grep -v '^/usr/lib/claude-desktop/' "$MANIFEST" | sort)
   ```

   Review the list. If it is not empty, remove those files by adding `| xargs -r sudo rm -v` to the same command.

7. **Regenerate your per-user desktop entry** with the commands in [Running under Plasma 6.8](#-running-under-plasma-68).

8. **Launch and confirm the version** from the running process:

   ```bash
   pgrep -af 'type=renderer' | grep -o '"appVersion":"[^"]*"' | sort -u
   ```

9. **Delete the previous version** once you are satisfied:

   ```bash
   sudo rm -rf /usr/lib/claude-desktop.prev
   ```

   Do not leave it in place. It contains an outdated copy of `chrome-sandbox` that still carries the setuid-root bit.

To roll back before step 9, quit the app, then run `sudo rm -rf /usr/lib/claude-desktop && sudo mv /usr/lib/claude-desktop.prev /usr/lib/claude-desktop`. After step 9, reinstall the older package by following the installation steps with its version number, which the signed index continues to list.

> [!NOTE]
> On the test machine, the update from 2.42304.2 to 5.11492.5 deleted the old directory before copying the new one. That worked, but it left no rollback copy if the copy had failed. Moving the old directory aside first is the safer order, and it is the only step here that was not run verbatim.

---

## 🧹 Removing

Your most recent manifest turns removal into an exact operation. Read it before using it, because it drives `rm` with `sudo`.

```bash
MANIFEST=$(ls ~/.local/share/claude-desktop-fedora/installed-files-*.txt | sort -V | tail -n 1)
less "$MANIFEST"

xargs -a "$MANIFEST" sudo rm -fv
sudo rm -rf /usr/lib/claude-desktop /usr/lib/claude-desktop.prev /usr/share/doc/claude-desktop
sudo update-mime-database /usr/share/mime
sudo update-desktop-database /usr/share/applications
sudo gtk-update-icon-cache -qf /usr/share/icons/hicolor
rm -f ~/.local/share/applications/com.anthropic.Claude.desktop*
kbuildsycoca6
```

Your settings and conversation data in `~/.config/Claude` are left alone. Delete that folder separately if you want a complete removal. The record folder can go last: `rm -rf ~/.local/share/claude-desktop-fedora`.

---

## 🩺 Troubleshooting

### Migrating from the Fedora 44 edition of this guide

The earlier edition wrote one manifest, `installed-files.txt`, and its update procedure never regenerated it. After any update that file no longer matches what is installed: on the test machine, 1,816 of its entries no longer existed and 3,939 installed files were missing from it. Do not remove anything with it.

Find which package you actually have by comparing each `.deb` you still hold against the installed tree:

```bash
for d in ~/Downloads/claude-desktop*.deb ~/claude-desktop-install/claude-desktop*.deb; do
  [ -f "$d" ] || continue
  t=$(mktemp -d ~/cmp.XXXXXX)
  dpkg-deb -x "$d" "$t"
  echo "$(dpkg-deb -f "$d" Version): $(diff -rq "$t/usr/lib/claude-desktop" /usr/lib/claude-desktop | wc -l) differences"
  rm -rf "$t"
done
```

The package with `0` differences is the one installed. Because the earlier edition copied with `cp -a`, also check that the system directories it wrote into still carry their expected SELinux labels. The `-n` flag makes this a dry run that only reports:

```bash
sudo restorecon -nv /usr /usr/bin /usr/lib /usr/share /usr/share/applications /usr/share/icons/hicolor /usr/share/doc
```

No output means nothing is mislabelled. If paths are listed, run the same command without `-n` to correct them. From there, the cleanest path is a normal update to the newest version, which writes a correct manifest in the new record folder. Look also for a leftover rollback directory such as `/usr/lib/claude-desktop.<version>.bak` from an earlier manual update. Each one holds an old setuid `chrome-sandbox`, so remove it once the current version works.

### `gpg: Note: Specified keyrings are ignored due to option "use-keyboxd"`

You ran a verification command written for the earlier edition. GnuPG ignored the `--keyring` option and placed Anthropic's key in your personal key store. Use the temporary `GNUPGHOME` method from Step 4 instead. To remove the key that was imported by mistake:

```bash
gpg --delete-keys 31DDDE24DDFAB679F42D7BD2BAA929FF1A7ECACE
```

### Claude Desktop asks you to sign in every time

The app stores its sign-in in the desktop keyring, which on Fedora KDE is KDE Wallet. Anthropic's documentation describes a conflict when GNOME Keyring is also installed on a Plasma desktop, and a failure to save when the keyring is locked. Check what you have before changing anything:

```bash
rpm -q kwalletmanager5 kf6-kwallet gnome-keyring
dnf repoquery --installed --whatrequires gnome-keyring
```

If `gnome-keyring` is installed and nothing you need depends on it, removing it and restarting resolves the conflict Anthropic describes. On the test machine, sign-in persisted without any change.

### The launcher opens the app without new features, or new menu actions are missing

Your per-user desktop entry in `~/.local/share/applications` still reflects an older version and is hiding the current system entry. Regenerate it as shown in [Running under Plasma 6.8](#-running-under-plasma-68).

### `.mcpb`, `.dxt` or `.skill` files do not open in Claude Desktop

The MIME registration in Step 8 was skipped or failed. Re-run the `install` and `update-mime-database` commands from that step, then check the association:

```bash
xdg-mime query filetype some-extension.mcpb
```

### `Permission denied` when reading files under `/usr/lib/claude-desktop`

A restrictive umask stripped the "other" permission bits from directories created during the copy. Reading a file requires execute permission on every directory above it, so one directory at `drwxr-x---` is enough to lock your own account out. Find the offending component with:

```bash
umask
namei -l /usr/lib/claude-desktop/chrome-sandbox
```

A umask of `0027` removes only the "other" bucket, so mirroring "group" onto "other" restores exactly what was lost:

```bash
sudo chmod -R o=g /usr/lib/claude-desktop /usr/share/doc/claude-desktop
sudo chmod o=g /usr/share/applications/com.anthropic.Claude.desktop
sudo find /usr/share/icons/hicolor -iname 'claude-desktop.png' -exec chmod o=g {} \;
```

The same umask can make other root-written files unreadable, including `.repo` files under `/etc/yum.repos.d`, which then break `dnf repolist` for ordinary users. Step 3's `umask 022` prevents this during installation.

### The app exits immediately on launch

Check the sandbox helper:

```bash
ls -l /usr/lib/claude-desktop/chrome-sandbox
```

If the owner-execute position shows `x` instead of `s`, the setuid bit was lost. Restore it:

```bash
sudo chown root:root /usr/lib/claude-desktop/chrome-sandbox
sudo chmod 4755 /usr/lib/claude-desktop/chrome-sandbox
sudo restorecon -F /usr/lib/claude-desktop/chrome-sandbox
```

Do not work around this with `--no-sandbox`. That flag disables a genuine security boundary in a browser engine that renders remote content.

### `dri_gbm.so: Permission denied` on launch

<details>
<summary><strong>Diagnosis, recorded on Fedora 44 and not re-observed on Fedora 45</strong></summary>

```
MESA-LOADER: failed to open dri: /usr/lib64/gbm/dri_gbm.so: cannot open shared object file: Permission denied
```

"Permission denied" usually points to file permissions or SELinux, so rule both out properly before drawing conclusions.

**1. Check the file.** It is a stock Fedora library that this install never touches, and it should read `-rwxr-xr-x root root`:

```bash
ls -l /usr/lib64/gbm/dri_gbm.so
```

**2. Check SELinux by looking for a logged denial.** `getenforce` alone proves nothing, and a search only counts if `auditd` is running:

```bash
systemctl is-active auditd
sudo ausearch -m avc -ts recent
```

If `auditd` is inactive, search the kernel log instead with `sudo journalctl -k -g denied`.

**3. Likely cause: Chromium's own GPU sandbox.** The package is built for Debian's multiarch library layout (`/usr/lib/x86_64-linux-gnu`), while Fedora uses `/usr/lib64`. The sandboxed GPU process appears to refuse a path it does not recognise and reports the refusal as a permission error. As a one-off diagnostic only:

```bash
claude-desktop --ozone-platform=x11 --disable-gpu-sandbox
```

If the message disappears, that confirms the cause. Do not keep the flag. Without it, the app falls back to software rendering, and Chat, Code and Cowork all work normally.

</details>

### Minor messages

| Message | Meaning |
|---|---|
| `vaInitialize failed: unknown libva error` | Hardware video decoding is unavailable, so Chromium decodes embedded video in software. Nothing else is affected |
| `Broken pipe` from `dpkg-deb` | You piped its output into `head`, which closed the pipe early. Nothing failed |
| `Remote file name has no length` from `curl` | The index lookup returned nothing. Check that `downloads.claude.ai` is reachable and that `dpkg --print-architecture` prints `amd64` or `arm64` |

---

## 🚧 Known limitations

**From the Linux beta itself**, regardless of distribution:

| Feature | Status on Linux |
|---|---|
| Chat, Claude Code and Cowork | Available |
| Computer Use (application and screen control) | Not available |
| Dictation in the desktop app | Not available. The CLI supports voice dictation |
| Quick Entry global hotkey | Works on X11. Under native Wayland it depends on the desktop's GlobalShortcuts portal |
| Self-updating | Not available. Updates arrive only through the package manager on supported systems |

**From this installation method:**

- `dnf` neither updates nor tracks the application. A future release that needs a newer system library will fail at run time instead of being caught at install time.
- The package's Debian install script never runs. The table below shows what that skips.

| Action in the package's install script | Needed on Fedora 45 KDE? |
|---|---|
| AppArmor profile for Ubuntu 24.04 and later | No. Fedora confines applications with SELinux |
| Apt signing key, repository entry and unattended-upgrades rule | No. Registering a Debian repository on an RPM system would be wrong |
| GNOME Shell search provider | No. Plasma does not use it |
| `/usr/bin/ccd` terminal launcher link | Only when a build ships `resources/bin/ccd`. Version 5.11492.5 does not |
| MIME types for `.mcpb`, `.dxt` and `.skill` | Yes. Step 8 replicates it |

---

## 📐 Reference architecture

![Reference architecture for the manual installation of Claude Desktop on Fedora 45 KDE Plasma, showing seven layers from upstream distribution through to Cowork virtualisation](claude-desktop-fedora-install-architecture.png)

A board-pack view of the installation, built for print as much as for the screen. It runs across seven layers: upstream distribution, acquisition and integrity verification, staging and extraction, the target filesystem, hardening and desktop integration, the lifecycle control plane, and Cowork virtualisation. Each layer is a titled, colour-tinted group rather than a swimlane, so the structure survives being exported flat into a slide or a PDF.

The integrity chain runs as a band across the top, because it applies to every install and every update. Below it the install reads left to right, from you at the terminal through Anthropic's repository, staging, `/usr`, desktop integration, the update lifecycle and the optional Cowork virtual machine. Every object carries a numbered badge, and the badge and border colour show its class. Data stores are drawn as cylinders. A dashed outline marks something temporary, such as the staging folder or the `.prev` rollback copy, or something outside your control, such as Anthropic's repository. Line style carries the interaction. A solid line is runtime data flow, and a dashed line is design-time governance, meaning the integrity chain, or the control loop that re-verifies on every update. Each connector takes the colour of the box it leaves, and the legend along the bottom names every numbered object.

| File | Use |
|---|---|
| `claude-desktop-fedora-install-architecture.png` | Viewing, and the version embedded above |
| `claude-desktop-fedora-install-architecture.svg` | Printing and scaling without quality loss |
| `claude-desktop-fedora-install-architecture.drawio` | Editing in [diagrams.net](https://app.diagrams.net), the draw.io desktop app or the VS Code extension |

---

## 📂 Repository contents

<details>
<summary><strong>Folder tree</strong></summary>

```
claude-desktop-fedora-kde/
├── README.md                                          this guide
├── LICENSE                                            MIT
├── assets/
│   ├── NeumannTechTips_Lockup_Horizontal_Tagline_OnDark.png
│   └── NeumannTechTips_Lockup_Horizontal_Tagline_OnLight.png
├── claude-desktop-fedora-install-architecture.drawio  editable diagram source
├── claude-desktop-fedora-install-architecture.png     rendered diagram
└── claude-desktop-fedora-install-architecture.svg     scalable diagram
```

</details>

**Files this guide creates on your system:**

| Location | Purpose |
|---|---|
| `/usr/lib/claude-desktop/` | The application |
| `/usr/bin/claude-desktop` | Launcher symlink |
| `/usr/share/applications/com.anthropic.Claude.desktop` | System desktop entry |
| `/usr/share/icons/hicolor/*/apps/claude-desktop.png` | Icons in five sizes |
| `/usr/share/mime/packages/com.anthropic.Claude.xml` | File-type registration |
| `/usr/share/doc/claude-desktop/copyright` | Licence text |
| `~/.local/share/applications/com.anthropic.Claude.desktop` | Your per-user entry with the XWayland flag |
| `~/.local/share/claude-desktop-fedora/` | Install manifests, one per version |

---

## 📚 Sources

| Source | What it supports |
|---|---|
| [Claude Desktop on Linux (beta)](https://code.claude.com/docs/en/desktop-linux), Anthropic | Supported distributions, signing key fingerprint, package pool lookup, Cowork requirements, keyring troubleshooting and the Linux feature gaps |
| [Claude Code setup: Linux package managers](https://code.claude.com/docs/en/setup#install-with-linux-package-managers), Anthropic | The CLI's signed apt, dnf and apk repositories |
| [Download Claude](https://claude.ai/download), Anthropic | Platform availability and plan requirements |
| [What's new in Fedora KDE Plasma Desktop 45](https://fedoramagazine.org/whats-new-in-fedora-kde-plasma-desktop-45/), Fedora Magazine | Plasma 6.8 as the desktop of Fedora 45 KDE |
| [Fedora Linux 45 officially released](https://linuxiac.com/?p=222316), Linuxiac | Plasma 6.8.2 on Fedora 45 and the retirement of the X11 session |
| [Fedora 45 now available](https://www.linux-magazine.com/Online/News/Fedora-45-Now-Available), Linux Magazine | Release contents, including the 7.2 kernel series |
| [EX-11: Prepping for Plasma's last X11-supported release](https://blog.davidedmundson.co.uk/blog/596/), David Edmundson, KDE | Removal of the Plasma X11 session in 6.8, with X11 applications continuing through XWayland |
| [T7265: `--keyring` ignored with `use-keyboxd`](https://dev.gnupg.org/T7265), GnuPG | Why `--keyring` no longer isolates keys, and the separate home directory alternative |
| [claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian), community project | RPM, AppImage and Nix repackaging of the official Linux app |
| [claude-desktop-debian v3.0.0 release notes](https://newreleases.io/project/github/aaddrick/claude-desktop-debian/release/v3.0.0%2Bclaude1.18286.0) | The rebase onto Anthropic's Linux app, the `claude-desktop-unofficial` name and the firmware compatibility link |

Observations attributed to the test machine come from commands run on it on 14 November 2026 and are recorded in [Validated configuration](#-validated-configuration).

---

## 📜 Disclaimer

Fedora is not a supported platform for Claude Desktop. This procedure installs an official, unmodified Anthropic package by hand and is offered as a community contribution. It carries no warranty and is not endorsed by Anthropic.

Anthropic's documentation states that support for further distributions is planned. When an official RPM arrives, migrate to it and remove this installation using the steps above.

This is general information, not professional advice, and the views here are the author's own and not those of any employer or client. The procedure runs commands with elevated privileges, installs a setuid binary and writes into system paths. Read every command before you run it, and do not try it first on a machine you cannot afford to rebuild.

Full version: [DISCLAIMER.md](https://github.com/NeumannTechTips/neumanntechtips-resources/blob/main/DISCLAIMER.md)

---

## 🔗 Related repositories

| Repository | Purpose |
|---|---|
| [`neumanntechtips-resources`](https://github.com/NeumannTechTips/neumanntechtips-resources) | Free prompt packs, checklists and reference material from the channel, licensed CC BY 4.0 |
| [`ollama-podman-openwebui-fedora-kde-guide`](https://github.com/NeumannTechTips/ollama-podman-openwebui-fedora-kde-guide) | Ollama and Open WebUI under Podman on Fedora KDE Plasma, with NVIDIA GPU acceleration |
| [`markitdown-mcp-guide`](https://github.com/NeumannTechTips/markitdown-mcp-guide) | Connecting Microsoft MarkItDown-MCP to any MCP-compatible AI client |
| [`hermes-signal-hardening`](https://github.com/NeumannTechTips/hermes-signal-hardening) | Securing a self-hosted Hermes agent's Signal messaging gateway |

This guide comes out of the work behind **[NeumannTechTips](https://www.youtube.com/@NeumannTechTips)**, a channel on practical AI for people who have to make it work inside a real organisation. A new video goes out every other Thursday.

---

## 📄 Licence

The text and commands in this guide are released under the [MIT Licence](LICENSE). Claude Desktop itself is Anthropic's software and remains subject to Anthropic's own terms. The NeumannTechTips logo files in `assets/` are brand assets and are not covered by the MIT Licence.

---

<div align="center">

**Community installation guide · [NeumannTechTips](https://github.com/NeumannTechTips)**

</div>
