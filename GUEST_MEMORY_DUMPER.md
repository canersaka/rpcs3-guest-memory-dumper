# Guest memory dumper

This fork adds one tool to stock RPCS3: **Utilities > Dump Guest Memory**. It writes the allocated PS3 guest pages from a loaded game to disk for offline analysis. It does not dump RPCS3 host-process memory, host registers, or GPU-only state.

## Output

You pick a folder and get two core files, plus one file for each live SPU thread:

- `guest_memory.bin` is a 4 GB logical address image. Its file offset equals the PS3 effective address, so a value at guest address `0x40401100` is also at file offset `0x40401100`.
- `manifest.json` lists every allocated region with its start, end, permissions, and storage mode, plus the SPU local-store table.
- `spu_XXXXXXXX_ls.bin` contains a live SPU thread's 256 KB local store. Its identifiers, program counter, type, and guest-memory offset are recorded in the manifest.

On a sparse-capable file system, only allocated pages consume disk space. If Windows cannot mark the image sparse, RPCS3 warns before creating a full 4 GB file. The manifest's `sparse_image` field records which storage mode was used.

## Usage

Build like stock RPCS3; see `BUILDING.md`. Run or pause your game at the point you care about, select **Utilities > Dump Guest Memory**, and choose a destination folder. Emulation pauses, the tool waits for all PPU, SPU, and RSX threads to settle, writes the snapshot, and then restores the previous running state. If the threads do not settle within five seconds, no dump is kept.

Free space is checked before writing. A failed or cancelled dump attempts to remove its output folder, and any cleanup failure is reported in the log.

## Notes

The image always reports a logical size of 4 GB. Copying a sparse image with a tool that does not preserve sparse files may expand it to the full size. The dump is a read-only snapshot; nothing is written back into the emulator.

Memory dumps may contain decrypted game content and personal data held by the game. Treat them as sensitive and do not share them casually.
