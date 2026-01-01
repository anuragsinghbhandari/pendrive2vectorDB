# What a pendrive REALLY is (end to end)?  

- What a pendrive is not : - storage + OS + filesystem  

- It's a tiny embedded system pretending to be simple.

## Components  

- NAND Flash memory.
- Flash controller (microcontroller)
- Crystal oscillator
- USB connector
- Tiny PCB
There is no CPU, but there is a controller that runs firmware.

### NAND Flash memory (core storage)  
```
BLOCK (e.g. 1 MB)
 ├── Page 0 (4 KB)
 ├── Page 1 (4 KB)
 ├── Page 2 (4 KB)
 └── ...
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
1 byte.  

Key limitations:

- Before write: 11111111  (if data is stored than value is 0 which means electron
is trapped, else 1 means no electron trapped no data stored).
- After write:  10110001

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

Flash memory can’t selectively remove electrons, so it erases data in
large blocks to stay reliable.
