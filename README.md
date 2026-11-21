# BEAMcal

Calibrates monitors, projectors and LED walls. It measures the display with a meter, scores it against an SDR or HDR target, and makes a 3D LUT (`.cube`) that corrects it. You load the LUT into the LED processor (Brompton Tessera, NovaStar), a LUT box, a monitor, or straight into a Blackmagic BiDi 12G over USB.

A simpler alternative to Calman: one window, four steps, shown across the top with a tick as each is done (**1 Check before → 2 Learn the display → 3 Correct it → 4 Check after**) and a short list of hardware.

## Install
- **Mac**: `dist/BEAMcal-<version>-mac-arm64.dmg` (Apple Silicon). The first time: right-click BEAMcal → **Open** → **Open**.
- **Windows**: `dist/BEAMcal-Setup-<version>.exe` installs it; `dist/BEAMcal-Portable-<version>.exe` runs without installing. SmartScreen: **More info → Run anyway**.
- **Meter software**: BEAMcal reads meters through **ArgyllCMS** 3.5.0's `spotread`, which comes inside the app (`vendor/argyll`, AGPL-3.0, unmodified, see `vendor/argyll/ABOUT.txt`). Nothing else to install. The Calibrite needs no driver on Mac or Windows. The Klein K-10 needs the FTDI driver on Windows. A different Argyll version can be used by showing BEAMcal its folder.

**Updates:** click the version number at the top of the window. Installed copies download new versions from GitHub (BEAMevents/BEAMcal) in the background and show **Install & restart**. BEAMcal never installs while it's measuring. The Mac needs the app in Applications. The Windows portable version only links to the download page.

## Using it
1. **Meter**: plug it in and press **Find**. *Simulated* is a made-up LED wall for trying BEAMcal without hardware.
   - **Measuring**: what the meter is pointed at: *LED wall*, *LCD monitor*, *Wide-gamut LCD*, *OLED*, *Projector*, *Plasma* or *CRT*. A colorimeter like the Calibrite Display Plus HL sees light through filters that don't match the eye exactly, so each kind of light needs its own spectral correction, or white and colours can read a few ΔE off (always the same way, so the LUT would aim wrong). The picker sets the meter's **Display type** (Refresh, in step with the flicker, for LED walls, projectors, plasma and CRT; Non-refresh for LCD and OLED) and the matching **Correction**.
   - The corrections are Calibrite's own. The first time one is needed, BEAMcal converts them from **Calibrite PROFILER** (or X-Rite i1Profiler) if it's installed on the computer, into BEAMcal's own folder (`<app data>/corrections`). Nothing else on the computer changes. If it isn't installed, the hint offers *Add corrections…* to pick `.edr` / `.ccss` files, or install Calibrite PROFILER (free from calibrite.com) and pick the kind again. LED wall uses Calibrite's *RGB LED* family (narrow red, green and blue LEDs).
   - *Other* leaves **Display type** and **Correction** to you (changing either by hand switches Measuring to Other). The Klein K-10 and spectrometers need no correction: the picker only sets the display type for them.
2. **Patterns**: pick where the colours go:
   - **A screen** (any display connected to the computer, such as the LED processor's HDMI input). This gives an 8-bit signal. On a Mac, set that screen's colour profile to *sRGB IEC61966-2.1* (System Settings → Displays). On Windows, turn off *Automatically manage colour* for it. Otherwise the operating system changes the colours. On a Mac, BEAMcal shows **not sRGB ⚠** on the screen and in the Patterns card when it isn't, and notes it in the log when you Learn.
   - **A Blackmagic UltraStudio / DeckLink**, which gives 10-bit SDI/HDMI (up to 4K on 12G models). Pick the video format. BEAMcal notices one plugged in while it's open within a few seconds.
   - **Patch**: *10%* is usual for LED walls and HDR, *Full* for projectors. **Settle**: how long to wait after each colour change (LED walls and slow monitors need longer).
3. **Target**:
   - *SDR*: Rec.709 / P3 / Rec.2020 colours; gamma 2.2 / 2.4 / BT.1886 / sRGB.
   - *HDR PQ*: tone-mapped from the content's peak (1000 / 4000 / 10000 nits) to what the display can really do.
   - *HDR HLG*.
   - **White**: *D65*, *Native* (keep the display's own white and fix everything else) or *Match…* (read another display's white with the meter and aim for that).
   - **Peak**: *Max* (as bright as the display can go at that white, minus 2%) or *Set* a number of nits.
   - For HDR, set the display or processor's input to PQ or HLG yourself. BEAMcal sends the signal values, not the HDR flag.
4. **1 Check before** (*Check*): checks the display as it is, before you change anything (see step 4 for what a check is). Listed as *Before*.
5. **2 Learn the display** (*Learn display*): reads lots of colours so BEAMcal can work out the correction. It gives no score. *Quick* (~160 colours), *Standard* (~420) or *Thorough* (~800). While it reads, the time left shows to the right of the progress bar (also for Check and Refine). It's worked out from how long the readings so far took, by how bright each colour is, because meters are much slower on dark colours; it carries over from one run to the next with the same meter set-up. Waiting for Next isn't counted. **Avg 2 / 3 / 5** reads every colour that many times in whole passes and uses the average, for a smoother LUT from a noisy meter or a flickering LED wall. After each pass BEAMcal puts white on the output and waits: move the meter if you like, then press **Next** (or ⌘/Ctrl+Return; Stop cancels). The log notes how much the passes disagreed. If a BiDi 12G is chosen, its LUT is turned off first so BEAMcal reads the display itself.
6. **3 Correct it** (*Make LUT*): 33 points suits most hardware (the BiDi always uses 33). Colours the display can't reach keep their brightness and hue and lose a little saturation.
   - **Save .cube…** for Brompton / NovaStar / Resolve / LUT boxes.
   - **Load into** picks where **Send** puts the correction: a BiDi 12G found on USB, or *This Mac's / PC's screen*.
   - **Send** loads it into a BiDi 12G over USB (Mac) and turns it on for both outputs.
   - **Clear** (shown when a BiDi is chosen) takes the LUT off it: no LUT selected, off on SDI and HDMI. Use it to start from the display as it is. Learn also turns the BiDi's LUT off by itself while it reads.
   - **Refine** (⌘R), once the LUT is loaded where it will be used: reads the display again through the LUT (the Learn colours, the Check colours and, for HDR, extra greys) and corrects the LUT for what's still off. It's for when the After check is worse than the LUT preview, usually because something changes the signal on its way to the LUT (the computer's colour management, a player, a switcher, a range setting). With a BiDi chosen in **Load into**, BEAMcal loads each refined LUT itself; otherwise it asks you to confirm the LUT is loaded, and you Save .cube and reload it after. Refine again if the next Check still isn't a Pass; each one corrects what the last left over. The LUT info shows *refined ×N*.
   - **This Mac's / PC's screen (ICC profile)** → **Install**: for a computer screen. BEAMcal makes an ICC display profile from the Learn readings (ArgyllCMS colprof, shaper/matrix) and makes it that screen's colour profile (ArgyllCMS dispwin, for your user only, no admin). It needs Patterns set to that screen when you Learn it. The profile describes the screen as it is, so colour-managed apps show colours correctly; it doesn't change the screen's white or brightness, and it's for SDR. Install asks for the profile's **name** (a dated one is suggested): it's what *Colour profile* shows in Displays, and the file name; if one with that name is already installed, it asks before replacing it. **Folder** (next to Install, or File → Show Colour Profiles Folder) opens where installed profiles live, to delete old ones: `~/Library/ColorSync/Profiles` on a Mac, `C:\\Windows\\System32\\spool\\drivers\\color` on Windows (deleting there may need an administrator). BEAMcal keeps no other copy. **File → Save ICC Profile…** saves the .icc instead, from any Learn.
   - To learn a screen again after installing a profile, set it back to *sRGB IEC61966-2.1* first, or BEAMcal measures through the new profile.
7. **4 Check after**: a short set of about 45 colours, compared with the target, giving **Pass / Close / Fail**. It changes nothing.
   - *Check* checks the display once the LUT is loaded in the processor, monitor or BiDi. Listed as *After*.
   - *Preview LUT* checks the display as it will be with the LUT: BEAMcal applies the LUT to the patterns itself, before you load it anywhere.
   - The usual order: 1 Check → 2 Learn display → 3 Make LUT → Preview LUT → load the LUT into the Brompton / NovaStar / BiDi → 4 Check.
   - **Chip chart** (⌘J) puts the DSC Labs chip chart (the same one BEAMgrade uses) full screen on the pattern output, to look at the display once it's calibrated. **Chart + LUT** (⇧⌘J) shows it with BEAMcal applying the LUT itself. The chart is made for the target on the left: unchanged for SDR Rec.709, the same colours in the wider container for P3 / Rec.2020, and white at 203 nits (HDR reference white) for PQ, or 75% for HLG. **Hide pattern**, Esc or the next reading takes it off.
   - **Accuracy %**: the share of the colours the display can reach that land within ΔE 2 of the target (green from 90%, amber from 60%). Shown on each result, above the charts and in the report.
   - Errors are ΔE2000 for SDR and ΔE ITP for HDR. **Pass** = average ≤ 2 and worst ≤ 5. Colours outside what the display can physically show are listed as *out of reach* and not scored.
   - **Results** (its own card, under step 4) lists every check: Before, LUT preview and After with Pass / Close / Fail. Click one to show it in the charts. **Save report…** is there too.
   - Charts: *Colours* (CIE xy with target and measured gamut), *Greyscale* (RGB balance and error per step) and *Patches* (each swatch split Before | After, with the target as a strip underneath and the ΔE change).
8. **Save report…** makes a PDF with the result, every check, the charts and every patch.
9. **↺ Start over** (top of the window, or Calibrate → Start Over…, ⇧⌘N) starts a new calibration: it clears the checks, the learnt display and the LUT, and the step bar goes back to 1. It asks first. Meter, pattern and target settings stay, and a LUT already loaded into a BiDi, processor or monitor stays there.

**Send Diagnostics to BEAM…** (Help menu, or the version number) sends BEAMcal's log, settings, latest readings and a status report to info@BEAMevents.com through BEAM Diagnostics. If the computer is offline, an email opens with the zip attached instead (Outlook on Windows, Mail on the Mac; otherwise the zip goes on the Desktop).

## How the LUT is made (lib/model.js)
1. A tone curve per channel, from the red, green and blue ramps (with extra readings near black), fitted smoothly on light^(1/2.4).
2. A cube of measured colours (5³ / 7³ / 9³) placed on those curves. Light is close to a straight line there, so blending between corners is accurate.
3. Small corrections from every other reading (greyscale, ramps) near where each was taken.
4. For each of the 33³ LUT points: the colour the target wants → Newton search for the signal that makes the model show it → if it's out of reach, keep brightness and hue and reduce saturation (dimmer if it's too bright).

5. **Refine** (lib/refine.js): with the LUT loaded, read again. For each colour, the model says what the LUT should have given; the difference, through the local slope, says how far the signal was moved before it reached the LUT. The new LUT is the original with its input shifted back by that (spread smoothly between the colours read, black kept black, 85% of each step so repeats settle). Out-of-reach colours barely move because the slope there is flat.

On the simulated wall (7500 K, wide primaries, uneven gamma, 5% droop) the test gets SDR from ΔE2000 avg 7.9 to about 0.3, and HDR PQ from ΔE ITP avg 53 to about 1.2. With `BEAMCAL_SIM_PATH=1` the signal is also changed before the converter (a slight gamma and saturation change, like colour management): the loaded LUT then checks at PQ ΔE ITP 4.7 (Fail) and one Refine brings it to about 1.5 (Pass); SDR 0.54 → 0.25 after two.

## Development
```
npm start            # run the app
npm test             # drives the real window with the simulated meter: Check → Learn → LUT → Check (SDR and PQ), saves .cube + report, checks the layout fits at default and minimum size, Send Diagnostics to a stand-in
npm run test:maths   # quick maths check without a window: test/lut-test.js [quick|standard|thorough] [sdr|pq|hlg]
npm run icon         # redraw build/icon.png
npm run dist:mac     # Mac .dmg + .zip
npm run dist:win     # Windows installer + portable
npm run release      # publish to GitHub (BEAMevents/BEAMcal); see scripts/release.js
```
This Mac has no Node.js, so run through Electron:
- App: `node_modules/electron/dist/Electron.app/Contents/MacOS/Electron .`
- Test: `… Electron test/app-test.js` (screenshots and files in `test/out`)
- Maths: `ELECTRON_RUN_AS_NODE=1 … Electron test/lut-test.js standard pq`

Run the test through the Blackmagic path with `BEAMCAL_SIMULATE=1 BEAMCAL_TEST_OUTPUT=blackmagic`, or on a screen with `BEAMCAL_TEST_OUTPUT=screen` (BEAMcal picks a plugged-in Blackmagic card otherwise, and the test then sends patterns to it).

Options: `BEAMCAL_PROFILE=name` keeps separate settings, `BEAMCAL_SIMULATE=1` adds a simulated Blackmagic output, `BEAMCAL_SIM_PEAK=1500` changes the simulated wall's brightness.

Build Mac and Windows separately with the electron-builder wrapper (`--mac --arm64`, then `--win --x64`), the same as BEAMgrade.

### Which files do what
- `main.js`: window, menus, settings, pattern output, Learn / Make LUT / Check runs, saving, BiDi, diagnostics.
- `src/app.js` + `index.html` + `app.css`: the operator window, charts and report HTML.
- `src/pattern.html` / `pattern.js` + `pattern-preload.js`: the full-screen pattern window.
- `lib/colour.js`: colour spaces, SDR/PQ/HLG targets, BT.2390 tone mapping, ΔE2000, ΔE ITP. Also loaded in the window via `src/colour-pre.js`/`colour-post.js`.
- `lib/patches.js`: the Learn and Check patch lists (ColorChecker included).
- `lib/model.js`: the display model, LUT maker, `.cube` read/write.
- `lib/refine.js`: Refine (corrects the LUT's input from readings taken through it).
- `lib/eta.js`: the time-left estimate.
- `lib/corrections.js`: the Measuring picker: kinds of display, finding Calibrite's .edr corrections and converting them with ArgyllCMS oeminst (bundled with spotread since 2026-11-21).
- `lib/score.js`: Check results (pass/fail, white, gamma, contrast, out of reach).
- `lib/chart.js` + `src/assets/dsc-labs-chart.png`: the chip chart, converted to the target (copied from BEAMgrade/src/assets).
- `lib/icc.js`: ICC display profiles: Learn readings → .ti3 → ArgyllCMS colprof; installs with dispwin.
- `lib/meters/argyll.js`: real meters via ArgyllCMS `spotread -O` (bundled in `vendor/argyll/<platform>-<arch>/bin` with colprof, dispwin and oeminst; a Klein shows as a serial port).
- `lib/meters/sim.js`: the simulated wall and meter.
- `lib/v210.js` + `lib/decklink.js` (+ `decklink-sim.js`): 10-bit Blackmagic output. `native/decklink/prebuilt` is copied from BEAMgrade, where its source lives.
- `lib/bidi.js` + `native/bidi/prebuilt/darwin/bidictl`: BiDi 12G over USB, copied from BEAMgrade (protocol in BEAMgrade/native/bidi/PROTOCOL.md).
- `lib/diagnostics.js` / `diagnostics.ps1` / `logfile.js`: Send Diagnostics and the log (`<app data>/logs/beamcal.log`, 5 MB).
- `lib/updater.js`, `lib/mac-update.js`, `src/update-ui.js`, `scripts/release.js` and `build/after-pack.js` are the shared BEAM updater/release files. Keep them identical to the other apps.

### Not done yet
- The Calibrite Display Plus HL is found (Argyll lists it as "X-Rite i1 DisplayPro, ColorMunki Display") and reads through BEAMcal's meter code (2026-11-21); a full Learn → LUT → Check with it hasn't been run yet. The Klein hasn't been tried.
- spotread stays open between readings (since 2026-11-21): bright colours take about 0.5 s with the Calibrite, dark ones several seconds (the meter reads longer in low light). A 46-colour Check took 73 s. If the open connection doesn't answer, BEAMcal goes back to one spotread per reading (~2.4 s more each); not yet tried on Windows.
- Measuring has been tried with the real Calibrite (one reading with LED wall: the correction moved white by about 0.007 in xy) but not through a whole calibration. Windows: the Calibrite PROFILER folder is a guess (`Program Files\\calibrite PROFILER\\resources\\…`); if it isn't found, *Add corrections…* asks for the files.
- Refine has only been run on the simulated wall. On the real display it needs the LUT loaded where it's used (BiDi, or by hand), and a Mac screen output should still be set to sRGB first.
- No HDR flag on the output. Set the processor or monitor to PQ/HLG yourself.
- LUTs are loaded into Brompton and NovaStar by hand (Save .cube). Direct upload would need their network control.
- Installing an ICC profile hasn't been run for real yet (the test makes it and finds the screen, but doesn't install). With two or more screens, the screen is matched to dispwin's list by position; check it picks the right one.
- Windows build not yet tried on a real PC (Blackmagic output, meters, ICC install, diagnostics email).
