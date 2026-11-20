# DiskCheck

A small disk checker written in ibyt. Boot a PC from a USB stick, choose a hard disk, SSD or USB stick, and test it for bad sectors. Typical uses: a used drive you are not sure about, a disk that makes noises, or a USB stick that may be fake (it claims more capacity than it really has).

DiskCheck runs on bare metal (BIOS boot, no operating system). It reads and writes the drives through the BIOS.

## Tests

| Choice | Name | What it does | Safe? |
|---|---|---|---|
| 1 | Quick read test | Reads every 32nd block of 32 KB plus the end of the disk (about 3 %). Seconds to minutes. | yes |
| 2 | Full read test | Reads every sector. | yes, no data is changed |
| 3 | Write test | Writes a pattern into every sector, then reads everything back and compares. | **no, destroys all data** |

The pattern of the write test depends on the sector number, so every sector gets different data. This is what finds **fake USB sticks**: they have less real memory than they claim, and the rest wraps around and overwrites the beginning. A read test cannot find this, the write test can. It also finds sectors that do not hold what was written.

## What it reports

- **Read errors**: sectors the drive could not read.
- **Write errors**: sectors the drive refused to write.
- **Data errors**: sectors that were read but hold wrong data (write test only).
- **Slow blocks**: blocks that took longer than 0.5 s. A drive that reads some areas only with effort is often close to failing.
- The first 32 bad sectors by number (`d` = wrong data, `w` = write error).
- A verdict: **OK** (nothing found), or **PROBLEMS FOUND** (copy your data to another disk now).

If a block fails, DiskCheck tries every sector of that block alone (twice), so the report names the exact sectors.

While a test runs, one progress line shows the state (reading, writing, verifying), percent, speed in MB/s, the error count and the slow blocks. Speed is measured with the CPU time stamp counter, calibrated against the PIT timer.

## Safety

- The disk the computer was started from is marked `boot disk (no write test)` and is **never written**.
- The write test only starts after you type the word `YES` in capital letters. Anything else cancels and nothing is written.
- Esc stops a running test. The report then says that the result is partial.
- Tests 1 and 2 never write.

## Using it

1. Put DiskCheck on a USB stick and boot from it (BIOS/Legacy/CSM mode, Secure Boot off).
2. The list shows all drives the BIOS knows, with BIOS code, size and sectors. The stick you booted from is marked.
3. Type the disk number and press Enter. Enter without a number quits.
4. Choose the test (1, 2 or 3). Enter without a number goes back to the list.
5. For the write test, type `YES` and press Enter.
6. Watch the progress line. Esc stops.
7. Read the result. The drive list comes back, so you can test another disk.

## Building

You need the ibyt compiler and the ibyt `library` folder next to the source. No other tools are needed.

```
cd diskcheck
<ibyt-compiler>          # produces MyProgram.img
```

Write the image to a stick with **ibytWriter**, or by hand (`/dev/sdX` is the whole stick, **not your hard disk**):

```
sudo dd if=MyProgram.img of=/dev/sdX bs=1M conv=fsync status=progress
```

To try it in an emulator, give it a second drive that you do not mind losing:

```
qemu-system-x86_64 -drive format=raw,file=MyProgram.img -drive format=raw,file=test.img
```

## Tested

In QEMU with an 8 MiB test disk:

- Quick read test, full read test and write test with verify: no errors on a healthy disk.
- Read errors injected at sectors 100 and 5000: both are found by the single-sector retry and listed.
- Wrong disk number, a wrong confirmation word (nothing is written).

**Not tested yet:** fake sticks, Esc during a test, drives larger than 2 TB.

## Limits (version 1)

- Only drives that the BIOS knows, only 512-byte sectors.
- The read test shows only what the BIOS reports. A drive can hide read errors; the write test then helps.
- The self-test functions of the drives (SMART) are not read.
- The BIOS reads and writes slowly. Large disks can take hours; use the quick test first.
- The report is on the screen only, there is no log file.

## No warranty

DiskCheck is provided as it is, without any warranty. The write test destroys all data on the chosen disk. You are responsible for the disk you choose.

## License

Released under the **MIT License**. You may use, change and share it freely, also commercially, as long as the copyright notice and the license text are kept.

Copyright (c) 2026 Alexander Zima, https://www.ibyt.tech
