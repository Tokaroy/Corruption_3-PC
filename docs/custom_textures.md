# Custom textures and HD texture packs

The port can replace the game's textures with your own images, using the same file names as
[Dolphin](https://dolphin-emu.org/)'s custom textures. That means **HD texture packs made for Dolphin work as they are**:
copy their files into one folder, start the game, and every texture the pack covers is drawn from the pack.

A player confirmed this in [issue #14](https://github.com/bonllavier/Corruption_3-PC/issues/14) with an AI-upscaled Metroid Prime 3 / Trilogy pack from the
Dolphin forums. This project does not make, host or link texture packs; get them from their authors.

Windows and Linux work the same way.

## Install a pack

1. **Close the game.** The folder is read once, at start-up.
2. Open the folder where the executable is and go to `UserData/texture_replacements/`. It already exists after the
   first start, and the port keeps its own files there (`strap_screen/`, plus `keyboard_icons/` and `variants/` when key or
   gamepad icons are on): **leave those as they are**.
3. Dolphin packs usually come as `Load/Textures/<game ID>/...`. Copy the **contents of the game ID folder** (or the
   whole game ID folder; subfolders are fine, they are searched) into `UserData/texture_replacements/`. For example:

   ```
   Corruption_3 PC.exe
   UserData/
     texture_replacements/
       strap_screen/          <- the port's, keep it
       keyboard_icons/        <- the port's (if key icons are on), keep it
       RM3E01/                <- the pack you copied
         tex1_128x128_1a2b3c4d5e6f7a8b_14.png
         ...
   ```

   Do not name your own folder `variants`: the port reserves that name and skips it.
4. Start the game. The first start with a large pack takes longer (see [Load times](#load-times-and-stutter)).

To play without the pack again, **move its folder out** of `texture_replacements` (see the note on F1 below).

### Which disc the pack must be for

A replacement is chosen by the **exact contents** of the original texture (a hash in the file name), not by its name in
the game. Packs are made for one disc:

- *Metroid Prime 3: Corruption* (USA) is `RM3E01`.
- *Metroid Prime Trilogy* (USA) is `R3ME01`; only its Metroid Prime 3 runs in this port (Trilogy add-on).

A texture that is byte-for-byte the same on both discs has the same hash and is replaced with either disc. A texture
that differs is only replaced with the disc the pack was made for; it then simply stays original. Nothing breaks
either way. Packs for Metroid Prime 1 or 2 have no effect, since those games do not run.

## Supported files

- **Name**: `tex1_<width>x<height>_<texture hash>_<format>` for normal textures and
  `tex1_<width>x<height>_<texture hash>_<palette hash>_<format>` for palette textures, the Dolphin scheme. The `_m`
  marker, the `_arb` suffix and the `$` wildcard in place of a hash are understood too. Width and height are those of the
  original texture; the image itself can be any size (2x, 4x, 8x...).
- **Formats**: `.png`, and `.dds` in DXT1/BC1, DXT5/BC3, ATI2/BC5, BC7 (DX10 header) or uncompressed 32-bit RGBA/BGRA.
  DXT3/BC2, BC4 and BC6H are not supported; those files are skipped and the original texture is used.
- **Mipmaps**: extra levels as separate files `<name>_mip1.png`, `<name>_mip2.png`... next to the main file, each half
  the size of the previous one, in the same format.
- Dolphin's newer *graphics mods* (JSON material and shader packs) are not supported; only image replacements.

Replaced textures are not touched by *Texture upscaling* (Super-xBR): the pack's image is used as it is.

## Load times and stutter

The port does not preload packs (there is no equivalent of Dolphin's *Prefetch Custom Textures* yet):

- At start-up it lists every file in `texture_replacements` to know which textures have a replacement. With tens of
  thousands of files, especially on a hard disk, this is noticeable.
- Each replacement image is read and decoded **the first time the game shows that texture**. The title screen and
  every new room load many textures at once, so the first start, the first visit to each area and the first time you
  open some menus take longer or hitch. After that the texture stays in memory and costs nothing.
- **PNG is the slowest format to load.** A pack converted to DDS (BC7 for quality, BC1/BC3 for size) loads much faster and
  takes less video memory. Dolphin's texture tools and many pack authors offer DDS versions.
- Replacement textures are kept in video memory up to 4 GB; past that the least recently used ones are dropped and
  read again from disk when needed. On graphics cards with little memory, prefer DDS or a 2x pack over a 4x one.

There is no progress indicator while a pack loads; the window simply takes longer to show the game. A loading screen was
requested in [#14](https://github.com/bonllavier/Corruption_3-PC/issues/14).

## The F1 option and dumping textures

*F1 > Video > Custom textures (texture_replacements folder)* and *Dump game textures to Cache/texture_dumps* both apply
at the next start.

- **Custom textures**: in v0.1.7 replacements are **always active**, ticked or not, because the port's own start-up
  screens (`strap_screen`) live in the same folder and switch the feature on. To turn a pack off, move its folder out of
  `texture_replacements`.
- **Dump game textures** (needs *Custom textures* ticked too): every texture the game shows that has no replacement is
  written to `UserData/Cache/texture_dumps/` as a `.dds`, already named the way its replacement must be named. This is
  how you check whether a pack matches your disc, or make your own replacements: edit the image, save it as `.png` or
  `.dds` with the **same name**, and put it in `texture_replacements`. Dumping costs disk space and some speed; turn it
  off when you are done.

## Problems

| What you see | What to do |
| --- | --- |
| Nothing changes | The pack is for another disc or game, or its files are not under `UserData/texture_replacements/`. Make sure the file names start with `tex1_`. Dump a few textures and compare names. |
| Some textures change, others do not | Normal if the pack does not cover everything, or was made for the other disc (see above). |
| A texture shows wrong colours or a wrong image | Hash collision or a wrongly named file in the pack. Find the file by its name in the dump folder and remove it from the pack. |
| Very long first start, hitches in new rooms | See [Load times](#load-times-and-stutter); convert the pack to DDS. |
| Out of memory / crash with a pack | Try a smaller (2x) or DDS version of the pack and send an F12 report saying which pack you use. |

When you report a visual problem, say whether a texture pack is installed (the F12 report does not include the pack).
