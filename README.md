# HP Victus 15 FA2082WM Complete Optimization Guide

### For people stuck with 50W TGP RTX 4050

-----

> **DISCLAIMER:** Everything in this guide is done at your own risk.
> I am not responsible for any damage to your device.
> This guide is based on real testing on FA2082WM with F.07 BIOS.

-----

## Device Specs (Test Machine)

- HP Victus 15 FA2082WM (120W)
- Intel i5-13420H (45W PL1 / 90W PL2 stock)
- 16GB Dual Channel DDR4
- RTX 4050 45+5W TGP (stock)
- 280W Omen grey tip charger (laptop pulls max 210W)

> **Minimum charger requirement: 150W. Recommended: 200W+**

-----

## Software Used

- OmenCore: https://omencore.pages.dev
- ThrottleStop
- HWiNFO64 (with EC polling disabled)
- MSI Afterburner
- UEFITool NE: https://github.com/LongSoft/UEFITool/releases
- IFR Extractor (search “IFR Extractor download”)
- setup_var.efi: https://github.com/datasone/setup_var.efi/releases
- UEFI Shell: https://github.com/tianocore/edk2/releases

-----

## Games Tested

- Rainbow Six Siege X — Esports settings @1080p, Reflex + Boost (CPU Intensive)
- Where Winds Meet — Max settings, No DLSS, No FG @1080p (GPU Intensive)

> All tests performed on fresh Windows install with manufacturer drivers.
> No pre-loaded bloatware. Tests done in same map areas for consistency.
> Tested for 1+ hours per configuration.

-----

## BEFORE YOU DO ANYTHING — REPASTE YOUR LAPTOP

**This is mandatory. Everything else is pointless without this.**

- Remove stock thermal paste
- Apply **PTM7950** or **Thermal Grizzly Phasesheet** on CPU and GPU dies
- Replace white putty under heatsink with **Upsiren UX Pro**
- Tested with: ID Cooling Frost PTM-2 + Upsiren UX Pro

This alone drops temps 10-15C and is the foundation of all performance gains.

-----

-----

# SECTION 1 — GPU VBIOS FLASH

-----

## Performance Results (No OC/UV Applied)

### Stock 50W VBIOS (Baseline)

|Game            |Avg FPS|GPU Temp|Hotspot|CPU Temp|Notes            |
|----------------|-------|--------|-------|--------|-----------------|
|R6S Esports     |130-150|65C     |73C    |90C     |Bad stutters     |
|Where Winds Meet|45-50  |68C     |77C    |79-80C  |Dips and stutters|

-----

### 75W Victus VBIOS (60W + 15W Dynamic Boost)

Download: https://www.techpowerup.com/vgabios/278726/278726

|Game            |Avg FPS|GPU Temp    |Hotspot|CPU Temp|Notes |
|----------------|-------|------------|-------|--------|------|
|R6S Esports     |160-180|70C @55W avg|78C    |90C     |Smooth|
|Where Winds Meet|60-65  |75C @70W max|86C    |79C     |Smooth|

-----

### 105W MSI VBIOS

Download: https://www.techpowerup.com/vgabios/260894/260894

|Game            |Avg FPS|GPU Temp    |Hotspot|CPU Temp|Notes               |
|----------------|-------|------------|-------|--------|--------------------|
|R6S Esports     |120-140|74C @70W avg|83C    |79C     |Stable but worse FPS|
|Where Winds Meet|63-68  |84C @95W    |96C    |79-80C  |Very smooth         |


> 105W tested with OC+UV: 2600MHz @ 0.925V + 1000MHz mem OC
> Result: Same FPS but cooler temps similar to 75W VBIOS

-----

## Which VBIOS Should You Choose?

**75W VBIOS — Recommended for most people**

- Best for mixed esports + AAA gaming
- Safe thermals without cooling pad
- Works great with DLSS and Frame Generation
- Consistent performance across all game types

**105W VBIOS + UV — For AAA gaming only**

- 5-10% better GPU performance in GPU intensive titles
- Worse performance in CPU intensive games (EC starves CPU)
- Needs cooling pad at 30C+ ambient temperature
- With ImonSlope (Section 2) this becomes more viable

**50W Stock VBIOS — Don’t bother**

-----

## How to Flash VBIOS

Guide: https://m.youtube.com/watch?v=fPgsv_unksk

-----

-----

# SECTION 2 — CPU OPTIMIZATION VIA IMON SLOPE

-----

## Why The CPU Throttles

The Embedded Controller (EC) on this laptop enforces a joule budget timer.
After approximately 1 minute of sustained load it hard caps the CPU to 25-45W
regardless of thermal headroom. This happens because:

- EC tracks total energy delivered over a rolling time window
- Once it hits its internal limit it throttles regardless of temperature
- ThrottleStop PL1/PL2 writes get overwritten by EC continuously
- CFG Lock blocks MSR writes entirely

**ImonSlope is the solution.** It makes the CPU underreport its power draw
to the EC — so the EC’s joule budget timer never triggers. CPU runs at full
performance while EC thinks it’s drawing much less power.

-----

## IMPORTANT — Do Not Use My Offsets Blindly

The BIOS variable offsets in this guide are specific to FA2082WM with F.07 BIOS.
Different models, different BIOS versions, or even different regional variants
may have different offsets. Writing wrong values to wrong offsets can brick
your laptop. Always verify your own offsets first using the steps below.

-----

## Step 0 — Extract Your BIOS File

### Create HP BIOS Recovery USB (Safety Net First)

Before doing anything create a recovery USB. This is your safety net if
anything goes wrong.

1. Download your BIOS update exe from HP Support for your exact model number
1. Insert a USB drive (8GB+ recommended, will be formatted)
1. Run the HP BIOS update exe
1. Select “Create Recovery USB” or “Save to USB” when prompted
1. Let it complete fully
1. Label this USB “BIOS RECOVERY” and keep it safe — do not use it for anything else

### Extract Raw BIOS File for Analysis

1. Right click the HP BIOS update exe
1. Open with 7-Zip → Extract here
1. Look inside extracted folder for a file ending in .bin, .rom, or .fd
1. This is your raw BIOS file — copy it to your desktop

-----

## Step 1 — Find Your Variable Offsets

**Never skip this step. Always verify your own offsets.**

### Using UEFITool NE

1. Download UEFITool NE from https://github.com/LongSoft/UEFITool/releases
   (Must be NE — New Engine version, not the old version)
1. Open UEFITool NE
1. File → Open → select your .bin/.rom/.fd BIOS file
1. Press Ctrl+F to open search dialog
1. Click the Text tab
1. Search for: CFG Lock
1. Double click the result in the bottom panel
1. It highlights a module in the tree above
1. Right click the highlighted module → Extract body
1. Save the extracted file (call it setup.bin)

### Using IFR Extractor

1. Download IFR Extractor (search “IFR Extractor download” — free tool)
1. Open IFR Extractor
1. Load your setup.bin file
1. It generates a large text file — save it somewhere
1. Open the text file in Notepad++ or any text editor

### Finding Your Offsets

Search the text file for each of these terms and note the VarOffset and VarStore:

**Search for each term one by one:**

```
CFG Lock
Overclocking Lock
UnderVolt Protection
Overclocking Feature
BIOS Lock
RTC Memory Lock
ImonSlope
```

**Each entry looks like this:**

```
One Of: CFG Lock, VarStoreInfo (VarOffset/VarName): 0x43,
VarStore: 0x1, QuestionId: 0x2C3
  One Of Option: Disabled, Value (8 bit): 0x0
  One Of Option: Enabled, Value (8 bit): 0x1 (default)
```

From this example:

- VarOffset = 0x43 (this is your offset to write)
- VarStore = 0x1 (this is which variable store it belongs to)
- Default value = 0x1 (Enabled) — you want to change this to 0x0

**Write down every offset and varstore on paper.**

### Finding VarStore Names

For each unique VarStore ID you found, search the text file for:

```
VarStoreId: 0x1
VarStoreId: 0x6
```

(Replace number with your actual IDs)

You will see something like:

```
VarStore: Name: CpuSetup, VarStoreId: 0x1
VarStore: Name: PchSetup, VarStoreId: 0x6
```

Write down every VarStore name and its corresponding ID number.
You need both the name and ID for the commands.

### For ImonSlope Specifically

There will be 3-4 ImonSlope entries. Search for all of them and note:

- Each VarOffset
- Which VarStore they belong to
- Their current default value (read it after booting — see Step 5)

The VCCIN ImonSlope entry is the most important one.

-----

## Step 2 — Prepare USB Drives

You need TWO separate USB drives:

- USB 1 — HP BIOS Recovery (created in Step 0 — do not touch)
- USB 2 — setup_var.efi boot drive

### Setting Up USB 2

1. Insert second USB drive
1. Format as FAT32
1. Create this exact folder structure on the USB:

```
USB:\EFI\BOOT\
```

1. Download UEFI Shell from https://github.com/tianocore/edk2/releases
1. Find the Shell.efi or UEFI_Shell.efi file
1. Rename it to BOOTX64.EFI
1. Place it at USB:\EFI\BOOT\BOOTX64.EFI
1. Download setup_var.efi from https://github.com/datasone/setup_var.efi/releases
1. Place setup_var.efi in the USB root (not inside any folder)
1. Create a new text file in USB root called startup.nsh
1. Inside startup.nsh write exactly this one line:

```
fs0:\setup_var.efi
```

1. Save and close

Final USB structure should look like:

```
USB:\EFI\BOOT\BOOTX64.EFI
USB:\setup_var.efi
USB:\startup.nsh
```

-----

## Step 3 — Disable Secure Boot

1. Restart laptop
1. Press F10 repeatedly to enter BIOS
1. Navigate to Security → Secure Boot
1. Set to Disabled
1. Save and Exit (F10 to save)

-----

## Step 4 — Boot USB and Unlock BIOS Variables

1. Insert USB 2
1. Restart laptop
1. Press F9 repeatedly for boot menu
1. Select your USB drive
1. UEFI Shell loads and automatically launches setup_var.efi
1. You now have a command prompt

### Verify Offsets Before Writing (Read First)

Replace the example values with YOUR offsets from Step 1:

```
setup_var.efi YourCpuSetupName(0xYourVarStoreID):0xYourCFGLockOffset
```

Example (FA2082WM F.07 only):

```
setup_var.efi CpuSetup(0x1):0x43
```

It should return 0x01 confirming CFG Lock is currently enabled.
If it says “No variable with specified name found” your varstore name is wrong — recheck Step 1.

### Unlock Security Variables First

Use YOUR PchSetup equivalent name and offsets:

```
setup_var.efi YourPchName(0xYourID):0xYourBIOSLockOffset=0x00
setup_var.efi YourPchName(0xYourID):0xYourRTCMemLockOffset=0x00
```

FA2082WM F.07 specific commands:

```
setup_var.efi PchSetup(0x6):0x1C=0x00
setup_var.efi PchSetup(0x6):0x1B=0x00
```

### Unlock CPU Variables

Use YOUR CpuSetup equivalent name and offsets:

```
setup_var.efi YourCpuName(0xYourID):0xYourCFGLockOffset=0x00
setup_var.efi YourCpuName(0xYourID):0xYourOCLockOffset=0x00
setup_var.efi YourCpuName(0xYourID):0xYourUVProtOffset=0x00
setup_var.efi YourCpuName(0xYourID):0xYourOCFeatureOffset=0x00
```

FA2082WM F.07 specific commands:

```
setup_var.efi CpuSetup(0x1):0x43=0x00
setup_var.efi CpuSetup(0x1):0x10E=0x00
setup_var.efi CpuSetup(0x1):0x381=0x00
setup_var.efi CpuSetup(0x1):0x1D9=0x00
```

### Verify Writes Succeeded

Read CFG Lock again — must return 0x00:

```
setup_var.efi CpuSetup(0x1):0x43
```

If it still shows 0x01 the write failed. Try adding –write_on_demand flag:

```
setup_var.efi --write_on_demand CpuSetup(0x1):0x43=0x00
```

-----

## Step 5 — Find and Apply ImonSlope

### Read Current ImonSlope Defaults First

While still in shell, read all ImonSlope offsets and write down current values:

```
setup_var.efi YourCpuName(0xYourID):0xImonSlope1
setup_var.efi YourCpuName(0xYourID):0xImonSlope2
setup_var.efi YourCpuName(0xYourID):0xImonSlope3
setup_var.efi YourCpuName(0xYourID):0xVCCINImonSlope
```

FA2082WM F.07 specific read commands:

```
setup_var.efi CpuSetup(0x1):0x16E
setup_var.efi CpuSetup(0x1):0x170
setup_var.efi CpuSetup(0x1):0x172
setup_var.efi CpuSetup(0x1):0x2EA
```

Note all values on paper — these are your restore values if needed.

### Apply ImonSlope — Set All 4 to Same Value

All 4 must be set to the same value for the scaling to work correctly.

Start with 0x4B (75%) and adjust based on thermals:

```
setup_var.efi YourCpuName(0xYourID):0xImonSlope1=0x3C
setup_var.efi YourCpuName(0xYourID):0xImonSlope2=0x3C
setup_var.efi YourCpuName(0xYourID):0xImonSlope3=0x3C
setup_var.efi YourCpuName(0xYourID):0xVCCINImonSlope=0x3C
```

FA2082WM F.07 specific commands (0x3C = recommended):

```
setup_var.efi CpuSetup(0x1):0x16E=0x3C
setup_var.efi CpuSetup(0x1):0x170=0x3C
setup_var.efi CpuSetup(0x1):0x172=0x3C
setup_var.efi CpuSetup(0x1):0x2EA=0x3C
```

Type exit to boot Windows:

```
exit
```

-----

## ImonSlope Value Reference Table

|Hex |Decimal|% Reporting |Actual draw at 30W reported|Recommended for                |
|----|-------|------------|---------------------------|-------------------------------|
|0x64|100    |100% (stock)|30W actual                 |Stock — no change              |
|0x50|80     |80%         |37W actual                 |Conservative start             |
|0x4B|75     |75%         |40W actual                 |Balanced                       |
|0x46|70     |70%         |43W actual                 |Moderate                       |
|0x3C|60     |60%         |50W actual                 |Aggressive (recommended)       |
|0x32|50     |50%         |60W actual                 |Too aggressive — thermal issues|

**How to choose:**

- Start at 0x4B and test thermals
- If CPU still throttles after 1 minute drop to 0x3C
- If thermals get too hot go back up to 0x46
- Never go below 0x3C — causes thermal runaway

**Note:** ThrottleStop and HWiNFO will show lower wattage than actual
after ImonSlope is applied. This is expected and normal — the CPU
is underreporting to the EC. Use clock speeds (GHz) as your
performance metric instead of wattage.

-----

## Step 6 — ThrottleStop Settings

After exiting shell open ThrottleStop as Administrator:

1. Click TPL button
1. Set PL1 = 40W
1. Set PL2 = 45W
1. Tau = 28 seconds
1. Back on main screen set Speed Shift EPP = 0
1. Hit Turn On

**Why these values with ImonSlope at 0x3C:**
ThrottleStop showing 40W = actual ~67W real power delivery.
Lower reported values prevent CPU from bursting beyond cooling capacity
while ImonSlope ensures EC never triggers its joule budget throttle.
This gives sustained high clock speeds without thermal runaway.

**Make ThrottleStop run on every boot:**

1. Open Task Scheduler
1. Create Basic Task
1. Name: ThrottleStop Gaming
1. Trigger: At log on
1. Action: Start ThrottleStop.exe with path to your exe
1. Check Run with highest privileges
1. Uncheck Start only if on AC power

-----

## Final Results After Full Optimization

### 75W VBIOS + ImonSlope 0x3C (Recommended Setup)

- CPU sustained at 4.5GHz under load
- No EC throttle past 1 minute
- GPU temps under 78C
- CPU temps 85-90C
- R6S: 160-180 FPS smooth
- Where Winds Meet: 60-65 FPS smooth
- No cooling pad required

### 105W VBIOS + UV (2600MHz@0.925V) + ImonSlope 0x3C

- 15% better GPU performance in AAA titles
- Same CPU intensive performance as 75W
- GPU max 78C, hotspot 87C
- CPU temps 87-91C
- Needs cooling pad at 30C+ ambient
- R6S: Similar to 75W (CPU bound game)
- Where Winds Meet: 63-68 FPS very smooth

-----

## Troubleshooting

**BIOS variables reverting after reboot:**

- Rerun all setup_var.efi commands from Step 4
- This can happen after Windows updates or BIOS updates
- Consider keeping startup.nsh with all commands for quick reapplication

**Random shutdowns:**

- Uninstall UXTU completely — causes EC conflicts on Intel systems
- Disable EC polling in HWiNFO (Settings → Safety → Disable EC Monitoring)
- Never run UXTU, HWiNFO with EC polling, and ThrottleStop simultaneously

**ThrottleStop controls greyed out:**

- BD PROCHOT and voltage controls remain locked at EC firmware level on this platform
- This is an HP Victus specific limitation — not fixable via software
- PL1/PL2 in TPL are what matter — those work correctly

**CPU still throttling after ImonSlope:**

- Verify all 4 ImonSlope offsets are set to same value
- Verify CFG Lock is still 0x00 (may have reverted)
- Try dropping ImonSlope value from 0x4B to 0x3C

**High CPU temps (above 90C):**

- Raise ImonSlope value (less aggressive reporting)
- Try 0x46 or 0x4B instead of 0x3C
- Ensure OmenCore fan program is active and set to Performance mode

**setup_var.efi says “No variable with specified name found”:**

- Your varstore name is wrong
- Go back to IFR text file and search VarStoreId to find correct name
- Common names: CpuSetup, Setup, PchSetup, SaSetup

**setup_var.efi says “Writing skipped”:**

- Value is already 0x00 — no action needed
- Or add –write_on_demand flag to force write

-----

## Important Warnings

- VBS and Memory Integrity will show as disabled after BIOS unlock — normal
- This slightly improves gaming performance by reducing CPU overhead
- Do not install UXTU on Intel systems ever
- Keep HP BIOS Recovery USB safe at all times
- Plundervolt (CPU voltage undervolting) remains locked on 13th gen Intel
  by Intel microcode — no tool can bypass this including ThrottleStop FIVR
  and Intel XTU — this is not an HP limitation
- 105W VBIOS hotspot temperatures above 95C during heavy load are normal
  for this GPU but long term reliability may be affected — use cooling pad

-----

## Software Setup Summary

|Software       |Purpose                     |Setting                |
|---------------|----------------------------|-----------------------|
|OmenCore       |Fan control + EC management |Power profile, max fans|
|ThrottleStop   |PL1/PL2 limits              |PL1=40W PL2=45W EPP=0  |
|MSI Afterburner|GPU monitoring + frame limit|GPU OC+UV if desired   |
|HWiNFO64       |System monitoring           |EC polling DISABLED    |
|UXTU           |Nothing — uninstall it      |N/A                    |

-----

## Links Summary

- OmenCore: https://omencore.pages.dev
- VBIOS Flash Guide: https://m.youtube.com/watch?v=fPgsv_unksk
- 75W Victus VBIOS: https://www.techpowerup.com/vgabios/278726/278726
- 105W MSI VBIOS: https://www.techpowerup.com/vgabios/260894/260894
- UEFITool NE: https://github.com/LongSoft/UEFITool/releases
- setup_var.efi: https://github.com/datasone/setup_var.efi/releases
- UEFI Shell: https://github.com/tianocore/edk2/releases
- ImonSlope Reddit Guide: https://www.reddit.com/r/overclocking/s/EBUduK8P4E

-----

*Guide by CkSenpai — HP Victus 15 FA2082WM owner*
*Tested at 30C ambient temperature*
*All tests on fresh Windows install with manufacturer drivers*
*No bloatware, no pre-installed software*
*Last updated: May 2026*
