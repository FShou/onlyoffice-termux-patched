# OnlyOffice Desktop Editors aarch64 Patched

Automatically patches the official ARM64 build of ONLYOFFICE Desktop Editors to run on Android aarch64 environments such as Termux Proot, Debian Proot, Ubuntu Proot, and Tiny Computer.

## Why This Exists

Recent ARM64 builds of ONLYOFFICE Desktop Editors may fail to start inside Android-based Linux environments with an error similar to:

```text
libQt5Gui.so.5: cannot enable executable stack as shared object requires: Permission denied
```

This repository automatically:

1. Downloads the latest ARM64 release of ONLYOFFICE Desktop Editors.
2. Extracts the Debian package.
3. Patches `libQt5Gui.so.5` by clearing the executable stack flag.
4. Rebuilds the package.
5. Publishes a patched release on GitHub.

## Features

* Automatic weekly update checks
* Manual workflow trigger support
* Downloads directly from official ONLYOFFICE GitHub releases
* Creates versioned GitHub releases
* Rebuilds patched `.deb` packages automatically

## Installation

Download the latest patched release from the Releases page.

Install:

```bash
dpkg -i onlyoffice-desktopeditors-<version>-aarch64-patched.deb
```

If dependencies are missing:

```bash
apt --fix-broken install
```

## Running

```bash
onlyoffice-desktopeditors
```

## Supported Environments

* Termux + proot-distro
* Debian Proot
* Ubuntu Proot
* Tiny Computer
* Other Android ARM64 Linux environments

## How the Patch Works

The workflow locates:

```text
libQt5Gui.so.5
```

inside the package and clears the executable stack flag (`PF_X`) from the GNU_STACK segment.

This fixes startup failures caused by Android's restrictions on executable stacks.

## GitHub Actions

The workflow runs:

* Every Monday via cron
* Manually through GitHub Actions

When a new ONLYOFFICE release is detected:

1. The package is downloaded.
2. The patch is applied.
3. A new GitHub Release is created.

Release tags follow the format:

```text
v<version>-aarch64-patched
```

Example:

```text
v9.4.0-aarch64-patched
```

## Disclaimer

This project is not affiliated with ONLYOFFICE.

ONLYOFFICE Desktop Editors and all related trademarks belong to their respective owners.

This repository only automates patching of the official ARM64 package and redistributes the resulting package under the terms of the original software license.
