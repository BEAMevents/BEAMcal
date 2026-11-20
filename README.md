# BEAMcal

Calibrates monitors, projectors and LED walls. It measures the display with a meter, scores it against an SDR or HDR target, and makes a 3D LUT (`.cube`) that corrects it. You load the LUT into the LED processor (Brompton Tessera, NovaStar), a LUT box, a monitor, or straight into a Blackmagic BiDi 12G over USB.

A simpler alternative to Calman: one window, four steps, shown across the top with a tick as each is done (**1 Check before → 2 Learn the display → 3 Correct it → 4 Check after**) and a short list of hardware.

## Install
- **Mac**: `dist/BEAMcal-<version>-mac-arm64.dmg` (Apple Silicon). The first time: right-click BEAMcal → **Open** → **Open**.
- **Windows**: `dist/BEAMcal-Setup-<version>.exe` installs it; `dist/BEAMcal-Portable-<version>.exe` runs without installing. SmartScreen: **More info → Run anyway**.
- **Meter software**: BEAMcal reads meters through **ArgyllCMS** 3.5.0's `spotread`, which comes inside the app (`vendor/argyll`, AGPL-3.0, unmodified, see `vendor/argyll/ABOUT.txt`). Nothing else to install. The Calibrite needs no driver on Mac or Windows. The Klein K-10 needs the FTDI driver on Windows. A different Argyll version can be used by showing BEAMcal its folder.

**Updates:** click the version number at the top of the window. Installed copies download new versions from GitHub (BEAMevents/BEAMcal) in the background and show **Install & restart**. BEAMcal never installs while it's measuring. The Mac needs the app in Applications. The Windows portable version only links to the download page.

## Using it
1. **Meter**: plug it in and press **Find**. *Simulated* is a made-up LED wall for trying BEAMcal without hardware. *Display type* and *Correction* (a `.ccss`/`.ccmx` file) are for meters that need to be told what they're looking at; leave them alone for the Klein.
2. **Patterns**: pick where the colours go:
   - **A screen** (any display connected to the computer, such as the LED processor's HDMI input). This gives an 8-bit signal. On a Mac, set that screen's colour profile to *sRGB IEC61966-2.1* (System Settings → Displays). On Windows, turn off *Automatically manage colour* for it. Otherwise the operating system changes the colours.
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
5. **2 Learn the display** (*Learn display*): reads lots of colours so BEAMcal can work out the correction. It gives no score. *Quick* (~160 colours), *Standard* (~420) or *Thorough* (~800). **Avg 2 / 3 / 5** reads every colour that many times in whole passes and uses the average, for a smoother LUT from a noisy meter or a flickering LED wall. After each pass BEAMcal puts white on the output and waits: move the meter if you like, then press **Next** (or ⌘/Ctrl+Return; Stop cancels). The log notes how much the passes disagreed. If a BiDi 12G is chosen, its LUT is turned off first so BEAMcal reads the display itself.
6. **3 Correct it** (*Make LUT*): 33 points suits most hardware (the BiDi always uses 33). Colours the display can't reach keep their brightness and hue and lose a little saturation.
   - **Save .cube…** for Brompton / NovaStar / Resolve / LUT boxes.
   - **Send** loads it into a BiDi 12G over USB (Mac) and turns it on for both outputs.
   - **This Mac's / PC's screen (ICC profile)** → **Install**: for a computer screen. BEAMcal makes an ICC display profile from the Learn readings (ArgyllCMS colprof, shaper/matrix) and makes it that screen's colour profile (ArgyllCMS dispwin, for your user only, no admin). It needs Patterns set to that screen when you Learn it. The profile describes the screen as it is, so colour-managed apps show colours correctly; it doesn't change the screen's white or brightness, and it's for SDR. **File → Save ICC Profile…** saves the .icc instead, from any Learn. Installed copies are also kept in BEAMcal's app data folder under `profiles`.
   - To learn a screen again after installing a profile, set it back to *sRGB IEC61966-2.1* first, or BEAMcal measures through the new profile.
7. **4 Check after**: a short set of about 45 colours, compared with the target, giving **Pass / Close / Fail**. It changes nothing.
   - *Check* checks the display once the LUT is loaded in the processor, monitor or BiDi. Listed as *After*.
   - *Preview LUT* checks the display as it will be with the LUT: BEAMcal applies the LUT to the patterns itself, before you load it anywhere.
   - The usual order: 1 Check → 2 Learn display → 3 Make LUT → Preview LUT → load the LUT into the Brompton / NovaStar / BiDi → 4 Check.
   - **Accuracy %**: the share of the colours the display can reach that land within ΔE 2 of the target (green from 90%, amber from 60%). Shown on each result, above the charts and in the report.
   - Errors are ΔE2000 for SDR and ΔE ITP for HDR. **Pass** = average ≤ 2 and worst ≤ 5. Colours outside what the display can physically show are listed as *out of reach* and not scored.
   - **Results** (its own card, under step 4) lists every check: Before, LUT preview and After with Pass / Close / Fail. Click one to show it in the charts. **Save report…** is there too.
   - Charts: *Colours* (CIE xy with target and measured gamut), *Greyscale* (RGB balance and error per step) and *Patches* (each swatch split Before | After, with the target as a strip underneath and the ΔE change).
8. **Save report…** makes a PDF with the result, every check, the charts and every patch.

**Send Diagnostics to BEAM…** (Help menu, or the version number) sends BEAMcal's log, settings, latest readings and a status report to info@BEAMevents.com through BEAM Diagnostics. If the computer is offline, an email opens with the zip attached instead (Outlook on Windows, Mail on the Mac; otherwise the zip goes on the Desktop).

## How the LUT is made (lib/model.js)
1. A tone curve per channel, from the red, green and blue ramps (with extra readings near black), fitted smoothly on light^(1/2.4).
2. A cube of measured colours (5³ / 7³ / 9³) placed on those curves. Light is close to a straight line there, so blending between corners is accurate.
3. Small corrections from every other reading (greyscale, ramps) near where each was taken.
4. For each of the 33³ LUT points: the colour the target wants → Newton search for the signal that makes the model show it → if it's out of reach, keep brightness and hue and reduce saturation (dimmer if it's too bright).

On the simulated wall (7500 K, wide primaries, uneven gamma, 5% droop) the test gets SDR from ΔE2000 avg 7.9 to about 0.3, and HDR PQ from ΔE ITP avg 53 to about 1.2.

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

Run the test through the Blackmagic path with `BEAMCAL_SIMULATE=1 BEAMCAL_TEST_OUTPUT=blackmagic`.

Options: `BEAMCAL_PROFILE=name` keeps separate settings, `BEAMCAL_SIMULATE=1` adds a simulated Blackmagic output, `BEAMCAL_SIM_PEAK=1500` changes the simulated wall's brightness.

Build Mac and Windows separately with the electron-builder wrapper (`--mac --arm64`, then `--win --x64`), the same as BEAMgrade.

### Which files do what
- `main.js`: window, menus, settings, pattern output, Learn / Make LUT / Check runs, saving, BiDi, diagnostics.
- `src/app.js` + `index.html` + `app.css`: the operator window, charts and report HTML.
- `src/pattern.html` / `pattern.js` + `pattern-preload.js`: the full-screen pattern window.
- `lib/colour.js`: colour spaces, SDR/PQ/HLG targets, BT.2390 tone mapping, ΔE2000, ΔE ITP. Also loaded in the window via `src/colour-pre.js`/`colour-post.js`.
- `lib/patches.js`: the Learn and Check patch lists (ColorChecker included).
- `lib/model.js`: the display model, LUT maker, `.cube` read/write.
- `lib/score.js`: Check results (pass/fail, white, gamma, contrast, out of reach).
- `lib/icc.js`: ICC display profiles: Learn readings → .ti3 → ArgyllCMS colprof; installs with dispwin.
- `lib/meters/argyll.js`: real meters via ArgyllCMS `spotread -O` (bundled in `vendor/argyll/<platform>-<arch>/bin` with colprof and dispwin; a Klein shows as a serial port).
- `lib/meters/sim.js`: the simulated wall and meter.
- `lib/v210.js` + `lib/decklink.js` (+ `decklink-sim.js`): 10-bit Blackmagic output. `native/decklink/prebuilt` is copied from BEAMgrade, where its source lives.
- `lib/bidi.js` + `native/bidi/prebuilt/darwin/bidictl`: BiDi 12G over USB, copied from BEAMgrade (protocol in BEAMgrade/native/bidi/PROTOCOL.md).
- `lib/diagnostics.js` / `diagnostics.ps1` / `logfile.js`: Send Diagnostics and the log (`<app data>/logs/beamcal.log`, 5 MB).
- `lib/updater.js`, `lib/mac-update.js`, `src/update-ui.js`, `scripts/release.js` and `build/after-pack.js` are the shared BEAM updater/release files. Keep them identical to the other apps.

### Not done yet
- Tested only with the simulated meter. The bundled spotread runs on this Mac, but the Calibrite Display Plus HL and Klein haven't been tried yet. Argyll lists the Calibrite ColorChecker Display/Pro/Plus family; if the HL isn't recognised, Argyll's I1D3_ESCAPE setting is the fallback.
- Each reading starts `spotread` again (about 1–2 s per reading with a real meter), so a Standard Learn could take 10+ minutes. A faster always-open meter connection is the next step once the meter is here.
- No HDR flag on the output. Set the processor or monitor to PQ/HLG yourself.
- LUTs are loaded into Brompton and NovaStar by hand (Save .cube). Direct upload would need their network control.
- Installing an ICC profile hasn't been run for real yet (the test makes it and finds the screen, but doesn't install). With two or more screens, the screen is matched to dispwin's list by position; check it picks the right one.
- Windows build not yet tried on a real PC (Blackmagic output, meters, ICC install, diagnostics email).
