# Troubleshooting and bug reports

## How to report a problem

1. **In game, press F12.** A small status line appears in F1 > Diagnostics while the report is written (one or two seconds).
   The port creates `UserData/Reports/report_<date>_v<version>.zip` and opens the folder in Explorer with the zip selected.
   The version in the name tells the developer exactly which build you ran; check **F1 > Info** for a newer one first.
2. Open a new post in [Discussions](../../../discussions), category *Bug reports*. Drag the zip into the text box.
3. Say **where you were**: planet, room or objective name, and what you were doing (fighting, using a lever, opening the map...).
   With the area name the developer can go there directly and reproduce it. A short description of what you saw is enough;
   the screenshot in the zip shows the rest.

What the zip contains, and nothing else:

| File | Content |
| --- | --- |
| `screenshot.png` | The frame on screen when you pressed F12 |
| `console.log` | The port's log for that session (loading, shaders, warnings) |
| `crash_*.txt` | Only if the game crashed earlier in that session: registers and stack |
| `Config.toml` | Your settings and key bindings |
| `system.txt` | Port version and build, Windows build, CPU, RAM, GPU, display, settings in use |
| `save/` | Your Metroid Prime 3 save (`save.bin`, `banner.bin`), so the developer can load the spot you report |
| `HOW_TO_REPORT.txt` | These instructions |

No game assets and no personal files. Open the files before sending if you want to check, and remove `save/` if you
prefer not to share your save.

### If the game closes with an error dialog

The dialog names the folder of that run, `UserData/Logs/base_<number>_pid<number>`. Zip that whole folder and attach it
to the post (it holds `console.log`, `crash_*.txt` and a memory snapshot that lets the crash be analysed offline).
Logs older than four days are deleted automatically.

### Performance problems (stutter, low FPS)

Create a report with **F12** and mention the area. Performance issues
are reproduced from the area name alone, so no further files are needed.

## Common issues

**The game asks for the disc image at every start, or says it was not found**
The chosen path is stored as `iso_path` under `[paths]` in `UserData/Config.toml`, and only once the image has opened.
If the file was moved, pick it again in the dialog or in **F1 > Diagnostics** (*Select disc image*, or paste the full
path into the box and press *Use this path*). Supported: `.iso` and `.rvz` only; any other file is refused with a message. Dolphin converts a dump to either
(right-click the game > *Convert File* > RVZ or ISO).

Ways to never see the dialog:
- **`game` folder:** put the dump in a folder named `game` next to the executable. The first disc image found there
  (alphabetically) is used whenever `iso_path` is empty, or names a file that no longer exists.
- **Command line:** `"Corruption_3 PC.exe" --iso "D:\Games\Metroid Prime 3 - Corruption (USA).rvz"` (also
  `--iso=<path>`). It applies to that start only and is not saved.
- **Environment variable:** `CORRUPTION3_ISO=<path>`, read once at start. Same rules as `--iso`.

`--iso` and `CORRUPTION3_ISO` win over `iso_path`. Set `CORRUPTION3_NO_DISC_DIALOG=1` to stop with the error message
instead of opening the file dialog (for unattended runs).

The error says which check failed: no image configured, a path that cannot be used (a moved file, a folder, a shell
location), a file that is not a readable Wii disc image, or the wrong game.

**Running through Proton or Wine (Linux): the disc image is asked at every start even after picking it**
Older builds (v0.1.6 and before) could save a "shell location" from the file dialog instead of a real path when the
dump was reached through *This PC* or its shortcuts, for example
`::{20d04fe0-3aea-1069-a2d8-08002b30309d}\Metroid Prime 3`. That value has no drive letter and no extension, so it
never opens. Newer builds use a dialog that only returns real files, refuse such values, and fall back to the `game`
folder. On any version, you can fix it by writing the real path into `UserData/Config.toml`. Under Proton or Wine, `Z:`
is the Linux root, and either `/` or `\\` works as separator; include the extension:

```toml
[paths]
iso_path = "Z:/home/you/Dumps/Metroid Prime 3 - Corruption (USA).rvz"
```

Alternatives: the `game` folder next to the executable (a symbolic link to the dump works under Proton), or `--iso` /
`CORRUPTION3_ISO` in the launcher's command line or environment. Thanks to cheurteen for the diagnosis and the
original version of this entry.

**"This disc image is not one the port runs"**
Two USA discs are supported: *Metroid Prime 3: Corruption*, game ID `RM3E01`, revision 0, and *Metroid Prime
Trilogy*, game ID `R3ME01` (Metroid Prime 3 only). The message names the game ID it found, for example
`Found RM3P01 - Metroid Prime 3: Corruption (Europe)`. PAL and Japanese discs, including the European Trilogy
(`R3MP01`), run different game code and will not work. With the Trilogy disc, `Corruption_3 PC.exe` starts
`Corruption_3 Trilogy Menu.exe` and `Corruption_3 Trilogy MP3.exe`, from the Trilogy add-on, which must be in the same folder. The USA disc includes English, Spanish, French, German and Italian (F1 > Language).

**"This disc is Metroid Prime Trilogy (USA). Playing its Metroid Prime 3 needs the Trilogy add-on"**
The Trilogy executables are a separate download. Get `Corruption_3-PC-vX.Y.Z-Trilogy-addon-win64.zip` of the same
version as your `Corruption_3 PC.exe` (F1 > Info shows it) from the release page and unzip it into the game folder.
The message names the file that is missing. To play Corruption instead, pick its disc image in the same dialog.

**"The Metroid Prime Trilogy add-on in this folder is vX, but Corruption_3 PC is vY"**
The two archives come from different releases. Download the add-on of the version the message names and unzip it
over the folder (or update Corruption_3 PC to the add-on's version). Both always have to be the same version.

**"Unsupported Processor"**
The port is built for x86-64-v3 (AVX2). Intel Core 4th generation (Haswell, 2013) or newer, AMD Excavator (2015) or newer.

**Black screen or crash at startup on the graphics side**
Try `graphics_api = "vulkan"` (or `"d3d12"`) in F1 > Video, and update your GPU drivers. Attach the run's log folder.

**Keys do nothing**
The game window must have focus. Check F1 > Controls that *Keyboard and mouse mode* is on (it needs a restart when changed).

**Low-battery warning from the game**
F1 > Controls > *Wii Remote battery always full* (on by default) answers the game's battery query with a full battery.

**Dark rooms or missing geometry for a moment**
Shaders for that material are still compiling. It resolves by itself within a second or two; the shared shader cache
grows with every release.

## Where things are

Everything lives next to the executable (portable layout):

| Folder | Content |
| --- | --- |
| `UserData/Config.toml` | Settings, key bindings and the disc image path (created from `Config.default.toml` on first run) |
| `UserData/NAND/` | Save files (Wii NAND layout) |
| `UserData/Logs/` | One folder per run with the console log and crash files |
| `UserData/Reports/` | F12 bug reports |
| `UserData/Cache/` | Shader and pipeline caches; safe to delete |
| `UserData/texture_replacements/` | Custom textures (Dolphin naming) and the generated keyboard icons |

## Updating

Unzip a new release over the same folder and overwrite. `UserData` is never inside the zip, so saves (`UserData/NAND`),
settings, key bindings, the disc image path and your bug reports survive. If a release note says a setting changed its
meaning, the note tells you what to edit; otherwise nothing is needed.
