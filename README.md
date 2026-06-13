# OnlyOffice aarch64 Termux/Proot Patched

OnlyOffice Desktop Editors patched to run on **aarch64 (ARM64)** Android devices using **Termux proot-distro** or any proot/chroot-based Linux environment (e.g. [Tiny Computer](https://github.com/search?q=tiny+computer+android)).

The original AppImage fails with this error on proot environments:

```
./DesktopEditors: error while loading shared libraries: libQt5Gui.so.5:
cannot enable executable stack as shared object requires: Permission denied
```

This is because `libQt5Gui.so.5` has an executable stack flag set, which is blocked by Android's kernel security policy. This patched build clears that flag.

---

## Download

Grab the latest `.AppImage` from the [Releases](../../releases) page.

---

## How to Run (Termux / Proot)

Since FUSE is not available in proot environments, you need to extract and run it manually:

```bash
# Extract the AppImage
./OnlyOffice-patched.AppImage --appimage-extract

# Run it
cd squashfs-root
./AppRun
```

---

## How We Patched It

### 1. Extract the original AppImage

```bash
./OnlyOffice.AppImage --appimage-extract
cd squashfs-root
```

### 2. Find the problematic library

```bash
find ~/squashfs-root -name "libQt5Gui.so*"
# Output: /home/tiny/squashfs-root/usr/bin/libQt5Gui.so.5
```

### 3. Clear the executable stack flag with Python

Since `execstack` is not available in proot/Termux environments, we patch the ELF binary directly using Python:

```bash
python3 - <<'EOF'
import sys

path = "./usr/bin/libQt5Gui.so.5"

with open(path, "r+b") as f:
    data = bytearray(f.read())

found = False
i = 0
while i < len(data) - 8:
    if data[i:i+4] == b'\x51\xe5\x74\x64':  # PT_GNU_STACK segment
        print(f"Found GNU_STACK at offset {i}")
        flags_offset = i + 4
        flags = int.from_bytes(data[flags_offset:flags_offset+4], 'little')
        print(f"Current flags: {flags:#010x}")
        flags &= ~0x1  # clear PF_X (executable bit)
        data[flags_offset:flags_offset+4] = flags.to_bytes(4, 'little')
        print(f"New flags:     {flags:#010x}")
        found = True
        break
    i += 1

if not found:
    print("GNU_STACK segment not found")
    sys.exit(1)

with open(path, "wb") as f:
    f.write(data)
print("Done!")
EOF
```

### 4. Repackage into AppImage

```bash
cd ~
mv squashfs-root onlyoffice-patched

# Download appimagetool for aarch64
wget https://github.com/AppImage/appimagetool/releases/download/continuous/appimagetool-aarch64.AppImage
chmod +x appimagetool-aarch64.AppImage

# Extract appimagetool (again, no FUSE)
./appimagetool-aarch64.AppImage --appimage-extract

# Repackage
./squashfs-root/AppRun ~/onlyoffice-patched OnlyOffice-patched.AppImage
```

---

## Tested On

- Tiny Computer (Debian 12, aarch64) on Android
- Should work on any Termux proot-distro (Debian/Ubuntu) on aarch64

---

## Credits

- [OnlyOffice](https://www.onlyoffice.com/) — the original app
- Patch method inspired by the `execstack` tool, reimplemented in Python for environments where it is unavailable
