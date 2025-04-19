# Background

This Fork of the QIDI_PLUS4 Repository will enable you to get perfect first layers.

Most importantly, you need to make sure that your bed is physically level, with deviation of preferably less than
0.2mm or 0.1mm accross the entire bed. Even with auto bed leveling, I have found that my first layers are not
consistent unless my bed has been physically leveled using the built-in QIDI proceedure to reset the platform using the
printed platform blocks, and adjusting the four bed corner screws until it is level.

Once your bed is confirmed that the mesh shows less than 0.2mm or 0.1mm deviation, you will still have issues with
first layers unless you preheat your bed. However, if you use the `gcode_macro.cfg` config from this repository, the
bed will preheat for you during the `PRINT_START` proceedure! And the preheating can be disabled from the slicer.

## Usage:

The way this configuration works is like this:

- Slice your part as usual
- Hit print
- The printer will proceed as usual, but before probing the bed, there will be a delay allowing the bed to preheat.
    - If the bed temp target is above 80c, the bed probing will be delayed for **15 minutes** allowing the thick heated plate to preheat and shift and expand.
    - If the bed temp target is above 45c, the bed probing will be delayed for **10 minutes** for the bed to preheat.
    - If the bed temp target is above 30c, the bed probing will be delayed for **5 minutes** for the bed to preheat.
- If your bed is already preheated or you dont want to wait for this delay, you can easily disable the delay in the slicer.
- During the preheat period, the layer numbers on the printer screen will indicate the progress in minutes.
- The layers will decrease from 15 -> 14 -> 13 -> ... -> 1 -> 0.
- You can tell when the printer is preheating because the total number of layers will show `69420`.
- During this preheating period, the chamber heater will also be running if the print settings have a chamber temp set.

## Setup:

https://www.youtube.com/watch?v=-zjfj2JZXnY

## Details

This Fork has a modified `gcode_macro.cfg` that has several improvements:

- Added custom 5 minute, 10 minute, and 15 minute delays before probing the bed for better first layers.
- Added an OPTIONAL boolean (1/0) argument to the `PRINT_START` macro called `WAIT` so you can disable the preheating
in the slicer.
- Added M600 G-code macro so slicers can use the filament change feature at specified layers.
- Improved logic on chamber heating so you don't have to wait as long for the chamber heater to reach the target temp.

This fork also provides a couple change to the QIDI Slicer and Orca Slicer **Machine Start G-code** and **Change Filament
G-code**.

- Makes Purge line much easier to remove in one piece especially with tricky filaments like CF-PET and TPU.
- Has comments for how to enable/disable the bed-preheating behavior.
- Adds M600 command to **Change Filament G-code**.

## Orca Slicer Machine G-code:

### Machine Start G-code:

```
; To Disable the bed preheating, change WAIT argument to '0' like this:
; PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer] CHAMBER=[chamber_temperature] WAIT=0

; To Enable the bed preheading, change WAIT argument to '1' like this:
; PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer] CHAMBER=[chamber_temperature] WAIT=1

; If WAIT argument is not present, the default behavior is to enable the bed preheating for better first layers
PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer] CHAMBER=[chamber_temperature]

SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count]
M83
M140 S[bed_temperature_initial_layer_single]
M104 S[nozzle_temperature_initial_layer]
M141 S[chamber_temperature]
G4 P3000
T[initial_tool]

; This Purge Line has been modified to ONLY use 0.3 mm First Layer, and use 120% Flow rate
; These changes make the purge line much easier to remove in one peice, especially for materials that like to stick
; too well to PEI like PET-CF and TPU
G0 X{max((min(print_bed_max[0] - 12, first_layer_print_min[0] + 80) - 85), 0)} Y{max((min(print_bed_max[1] - 3, first_layer_print_min[1] + 80) - 85), 0)} Z5 F6000
G0 Z0.4 F600
G1 E3 F1800
G1 X{(min(print_bed_max[0] - 12, first_layer_print_min[0] + 80))} E{85 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 Y{max((min(print_bed_max[1] - 3, first_layer_print_min[1] + 80) - 85), 0) + 2} E{2 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 X{max((min(print_bed_max[0] - 12, first_layer_print_min[0] + 80) - 85), 0)} E{85 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 Y{max((min(print_bed_max[1] - 3, first_layer_print_min[1] + 80) - 85), 0) + 85} E{83 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 X{max((min(print_bed_max[0] - 12, first_layer_print_min[0] + 80) - 85), 0) + 2} E{2 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 Y{max((min(print_bed_max[1] - 3, first_layer_print_min[1] + 80) - 85), 0) + 3} E{82 * 0.5 * 0.3 * 1.2 * nozzle_diameter[0]} F3000
G1 X{max((min(print_bed_max[0] - 12, first_layer_print_min[0] + 80) - 85), 0) + 3} Z0
G1 X{max((min(print_bed_max[0] - 12, first_layer_print_min[0] + 80) - 85), 0) + 6}
G1 Z1 F600
SET_PRINT_STATS_INFO CURRENT_LAYER=1
```

### Change Filament G-code:

```
; This requires the M600 addition to the gcode_macro.cfg
M600
```

# Original README from QIDI Below:

# QIDI_PLUS4

<p align="center"><img src="other/QIDI.png" height="240" alt="QIDI's logo" /></p>
<p align="center"><a href="/LICENSE"><img alt="GPL-V3.0 License" src="other/qidi.svg"></a></p>

# Documentation Guidelines

QIDI PLUS4 is a 3D printer that uses Klipper as its foundation. This repository is used for updates and releases for the PLUS4 model, as well as for issue tracking.

For convenience, QIDI provides version-specific packaged files. Please download the necessary compressed package file prefixed with "PLUS4." Select the appropriate branch for download, with each branch name reflecting the corresponding version.

## V1.6.0 Update Content

**Note:** After updating, the Klipper configuration file will be replaced. The previous configuration file will be backed up as `printer_{datetime}.cfg`, printer recalibration will be required.

1. Added network testing functionality.
2. Added support for AX300 and AX600 Wi-Fi adapters.
3. Optimized chamber heating logic: chamber heating starts only when the heated bed reaches the specified temperature.
4. Added temperature limits and reminders (chamber, heated bed, nozzle) on the Fluidd page.
5. Fixed an issue where disconnection caused online updates to crash.
6. Fixed the problem of losing settings after rebooting following a language switch.
7. Enhanced support for LAN mode.


## The online update version will be uploaded later. If you have already updated this version earlier, please ignore the online update prompt.


## Detailed update process

#### Packaged files

Note that all updates can not be updated from higher versions

1. Select the latest version in the version release bar next to it, download the compressed file package starting with PLUS4 and extract it locally. <a href="https://github.com/QIDITECH/QIDI_PLUS4/releases">Jump link</a>

2. Transfer the files to a USB drive. For example:

    <p align="left"><img src="other/sample.png" height="240" alt="sample"></p>

3. Insert the USB drive into the machine's USB interface, click the `Check for updates` button and an update prompt will appear on the version information interface. Update according to the prompt.

## Report Issues and Make Suggestions

For any concerns or suggestions, feel free to reach out to our [After-Sales Service](https://qidi3d.com/pages/warranty-policy-after-sales-support).

Should you encounter any issues related to machine mechanics, slicing software, firmware, or various other machine-related problems, our after-sales team is ready to assist. They aim to respond to all inquiries within twelve hours. Alternatively, you can post an issue in this repository, our developers will reply to you directly.

## Others
QIDI's 3D printers operate based on the Klipper system. Building on the Klipper open-source project, we've tailored its source code to meet specific user requirements.

Similarly, we've adapted Moonraker to ensure our designed screens align with web operations.

We also use Fluidd as one of the ways for users to interact with the printer.

We extend our gratitude to the developers and maintainers of these open-source projects and encourage users to explore or support these great projects.

| Software      |Official| QIDI edition                                                                     |
| ------------- |----------| -------------------------------------------------------------------------------- |
| **Klipper**   |**[https://github.com/Klipper3d/klipper](https://github.com/Klipper3d/klipper)**| **[https://github.com/QIDITECH/klipper](https://github.com/QIDITECH/klipper)**   |
| **Moonraker** |**[https://github.com/Arksine/moonraker](https://github.com/Arksine/moonraker)**| **[https://github.com/QIDITECH/moonrake](https://github.com/QIDITECH/moonrake)** |
|**Fluidd**|**[https://github.com/fluidd-core/fluidd](https://github.com/fluidd-core/fluidd)**||
