*EXPERIMENTAL*

# wg-tool

WARNING: This tool was lightly tested and works well for my needs.
There may be surprising edge cases I've not tested yet.
It might work, it might do nothing or it could brick your router!

Use the backup feature before yiu do any thing else...just in case.

You have been warned!

`wg-tool` is a utility for **ASUSWRT-Merlin** routers that simplifies the management of WireGuard VPN client slots (`wgc1`–`wgc5`).  
It safely handles swapping, copying, erasing, backing up, and restoring client configurations directly from NVRAM, while always keeping timestamped backups so nothing is lost.

---

## Features

- **Swap** two slots while preserving all NVRAM keys.
- **Copy** a slot into another without touching the source.
- **Zero (erase)** a slot by clearing its keys.
- **Backup** all five slots at once to a timestamped directory.
- **Make-config**:  
  - With backup, also generate `.conf` files for each slot.  
  - Standalone: convert any saved `.nvram` dump into a ready-to-use `.conf`.
- **Dry-run** mode to preview changes without committing them.
- **Restart** flag to re-enable and restart WireGuard after an operation (default is to leave clients disabled).
- **Safety-first**:  
  - Every run creates a timestamped backup directory under `/jffs/backups/wg_tool/`.  
  - Both `.nvram` and optional `.conf` files are stored.  
  - Destructive actions require explicit confirmation.  

---

## Installation

1. Copy `wg-tool` into your router under `/jffs/scripts/`.
2. Make it executable:

   ```sh
   chmod +x /jffs/scripts/wg-tool
   ```

3. Run directly from SSH to verify installation:

   ```sh
   /jffs/scripts/wg-tool --help
   ```

---

## Usage

```text
wg-tool [-d] [-r] wgcA wgcB
    Swap slot A <-> slot B  (A and B in wgc1..wgc5)

wg-tool [-d] [-r] -c|--copy wgcA wgcB
    Copy slot A -> slot B   (A and B in wgc1..wgc5)

wg-tool [-d] -z|--zero wgcA
    Zero (erase) slot A     (A in wgc1..wgc5)

wg-tool -b|--backup [-m|--make-config]
    Backup all five slots (wgc1..wgc5) to a stamped directory.
    With -m, also generate matching wgcN.conf files.

wg-tool -m|--make-config /path/to/wgcX.nvram
    Convert a saved .nvram file into a .conf with the same basename.
```

### Flags

- `-b, --backup`     Backup mode (no changes to NVRAM)  
- `-c, --copy`       Copy mode (default is swap)  
- `-d, --dry-run`    Plan only; no prompt, no changes (ignored in backup / standalone make-config)  
- `-m, --make-config`  
  With backup: also emit `.conf` per slot  
  Standalone: convert a `.nvram` file into `.conf`  
- `-r, --restart`    Re-enable affected clients and restart WG (not for zero target)  
- `-z, --zero`       Erase mode (one argument)  
- `-h, --help`       Show this help  

---

## Examples

Backup all slots (nvram dumps only):
```sh
/jffs/scripts/wg-tool --backup
```

Backup all slots, with `.conf` output too:
```sh
/jffs/scripts/wg-tool --backup --make-config
```

Convert a saved `.nvram` file into a `.conf`:
```sh
/jffs/scripts/wg-tool --make-config /jffs/backups/wg_tool/20250824_094221/wgc3.nvram
```

Swap `wgc1` and `wgc3`, leave both disabled afterwards:
```sh
/jffs/scripts/wg-tool wgc1 wgc3
```

Copy Proton in `wgc3` into slot `wgc1`, then re-enable and restart:
```sh
/jffs/scripts/wg-tool --copy --restart wgc3 wgc1
```

Zero out slot `wgc2` (erase keys), preview only:
```sh
/jffs/scripts/wg-tool --dry-run --zero wgc2
```

Zero out slot `wgc2` for real (slot remains disabled afterwards):
```sh
/jffs/scripts/wg-tool --zero wgc2
```

---

## Backup Directory Layout

Every run creates a directory under `/jffs/backups/wg_tool/` named with a timestamp:

```
/jffs/backups/wg_tool/20250824_094221/
    wgc1.nvram
    wgc1.conf   (if --make-config was used)
    wgc2.nvram
    wgc2.conf
    wgc3.nvram
    wgc3.conf
    wgc4.nvram
    wgc4.conf
    wgc5.nvram
    wgc5.conf
```

- `.nvram` files are exact key/value dumps from the router.  
- `.conf` files are ready-to-use WireGuard configs reconstructed from NVRAM.  

---

## Safety Notes

- By default, all involved clients are **disabled** after changes. Use `-r/--restart` to bring them back online.  
- Destructive operations (swap, copy, zero) always print a plan and ask for confirmation before applying.  
- Dry-run mode (`-d`) is strongly recommended before running changes on production routers.  
