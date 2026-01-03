# What a pendrive REALLY is (end to end)?  

* What a pendrive is not : - storage + OS + filesystem  

* It's a tiny embedded system pretending to be simple.

## Components  

* NAND Flash memory.
* Flash controller (microcontroller)
* Crystal oscillator
* USB connector
* Tiny PCB
There is no CPU, but there is a controller that runs firmware.

### NAND Flash memory (core storage)  
```
BLOCK (e.g. 1 MB)
 ├── Page 0 (4 KB)
 ├── Page 1 (4 KB)
 ├── Page 2 (4 KB)
 └── ...
<<<<<<< HEAD
```
- bits live here.  
- Data is stored in pages (e.g. 4KB - 16KB)
- Pages are grouped into erase blocks (e.g. 256 pages)
- can do  
-- Read a page  
-- Write a page (only if empty).
- can't do
-- overwrite a page.
- you must have to erase the entire block first even if you want to change
=======

* bits live here.  
* Data is stored in pages (e.g. 4KB - 16KB)
* Pages are grouped into erase blocks (e.g. 256 pages)
* can do  
    Read a page  
    Write a page (only if empty).
* can't do
    overwrite a page.
* you must have to erase the entire block first even if you want to change
>>>>>>> 7fc5215 (updated other components:)
1 byte.  

Key limitations:

* Before write: 11111111  (if data is stored than value is 0 which means electron
is trapped, else 1 means no electron trapped no data stored).
* After write:  10110001

you can flip 1 -> 0,
but you cannot flip 0 -> 1 unless erase the entire block, erase resets all
bits in the block back to 1. (because electrons are trapped in the flash cell's
structure, you can only reset them in large groups, not individually. )

Why writing is easy but erasing is hard:-  
writing(1->0) , electrons are pushed in , can be done per page, precise enough.
erasing(0->1) , electrons must be pulled out requires high voltage,
  causes electrical stess, cannot be precisely targeted to one cell)

so the consequences:-  
 to change 1 small value, you have to update 4 KB, but that page lives
 in a 1 MB block, so the conrtroller reads entire 1MB block into RAM ,
 modifies
 4 Kb part, erase the whole 1 MB block and write back the new 1 MB block.  

> Flash memory can’t selectively remove electrons, so it erases data in large
> blocks to stay reliable.
> Data is stored in pages because flash hardware can only reliably read/write
> groups of cells, and erased in blocks because removing electrons requires high
> voltage applied to a large region.

### Flash controller (microcontroller)  

> If NAND flash is body, the flash controller is the brain.

A flash controller is a tiny embedded system that sits between:

```powershell
USB / SATA / PCIe
↓
Flash Controller
↓
Raw NAND Flash

```

Inside the controller you typically have:
  A microcontroller / processor core
  SRAM (small but fast)
  DMA engines
  ECC engines
  USB/SATA/NVMe interface logic
  Firmware (this is key)

FTL is firmware inside the flash controller that makes flash look like a normal
disk by translating logical addresses into physical flash pages.

Why FTL is needed (recap in one breath)

Flash can:

* Read → page
* Write → empty page only
* Erase → entire block

But the OS expects:

* Overwrite anywhere
* Random writes
* Fixed block addresses

> FTL bridges this mismatch.
FTL constantly remaps blocks, so block 0 today might not be the block 0 tomorrow.

#### Core responsibilities of FTL

* Logical -> physical mapping
    FTL keeps a table:
      LBA -> Physical Page (e.g. LBA 1024 -> Block 87, Page 12)
      This table changes all the time.
* Copy-on-write (fake overwrites)
    When OS says write (WRITE LBA 1024), ftl does not overwrite(as discussed above),
    it write to a new empty page and update mapping and marks old page as stale.
* Garbage Collection
    Block contains a mix of valid and stale pages.
    So FTL peroidically:
      Copies valid pages elsewhere
      Erases the whole block
      Reuses it
    Erase is delayed not immediate
* Wear leveling (prevent early death)
    Flash blocks wear out after limited erases.
    FTL ensures:
      Writes are spread evenly
      No block is overused
    Two types:
      Dynamic (new writes go to least-used blocks)
      Static (even rarely-changed data gets moved)

FTL firmware maintains:
    Mapping tables
    Erase counters
    Free page lists
    Garbage collection state
    Error correction info (ECC)

why pendrive FTL is weak:
  Tiny controller
  Little or no DRAM
  Simplistic mapping
  Minimal GC logical

FTL lives inside Flash controller firmware

Why databases & random writes suffer

  Because:
    Small overwrite → new page
    Many overwrites → many stale pages
    GC triggers → massive internal copying
  This causes write amplification:
    Write 4 KB → flash moves 1 MB internally

## USB interface Layer

The controller speaks USB protocol

Most pendrives expose: USB Mass Storage Class (MSC), this tells OS , treat
me like a disk, send read / write commands.

## File system (Not part of Pendrive)

File system lives on the pendrive, but is created by the OS.
  FAT32/ exFAT/ NTFS
  Allocation tables
  Directories
  Filenames

Pendrives don't understand files, it only understands read write commands
