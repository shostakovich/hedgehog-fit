Status: DONE

Task: Image preprocessing — downscale + grayscale

Work completed:
- Created `src/assets/igel-1.png` through `src/assets/igel-6.png` from the six source PNGs in `tmp/`
- Applied `-colorspace Gray`, `-resize 400x400`, and `-strip` with ImageMagick
- Kept pose mapping aligned to the spec order after visual inspection:
  - `igel-1` = Planke
  - `igel-2` = Liegestütze
  - `igel-3` = Bein-Dehnung
  - `igel-4` = Aufwärmen
  - `igel-5` = Sitz-Dehnung
  - `igel-6` = Bizeps

Verification:
- `ls -la src/assets`
  - exactly six output files present: `igel-1.png` … `igel-6.png`
- `magick identify src/assets/igel-*.png`
  - all files are `PNG 400x400`
  - all files report `8-bit Grayscale Gray`
  - file sizes:
    - `igel-1.png` = `49362B`
    - `igel-2.png` = `51107B`
    - `igel-3.png` = `48692B`
    - `igel-4.png` = `51066B`
    - `igel-5.png` = `51642B`
    - `igel-6.png` = `53514B`
- Visual inspection:
  - reviewed all six converted outputs
  - confirmed the final filename mapping above is the intended working set for later embedding/rendering

Notes:
- No Liquid, YAML, docs, or bin files were modified for this task.
