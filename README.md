# Happy-Hare-Plus4-Configs

## ISSUES / TODO
* Macros are WIP:
    * ENABLE / DISABLE_ALL_SENSORS not working
    * RESUME and RESUME_PRINT macros needs to be reworked for HH 
    * saved variables need to be reworked
    * better handling / testing of different cases and errors
* configs in general are still WIP (definitely needs some tuning)
* no LED signalling
* probably more

## Happy Hare install

1. Copy the configs (mmu folder, box_hh.cfg and box_macros.cfg).

2. Install Happy Hare from the [WIP Plus4 repo](https://github.com/Wazzup77/Happy-Hare).

3. Run the install script `Happy-Hare/install.sh` and pray that it does not break stuff.

4. Add `mmu__revision = 0` to `saved_variables.cfg'

If you're modifying the files for development purposes you will need to rerun the install script.

## `[printer.cfg]` changes

1. Remove Qidi's stock box config include `[include box.cfg]`

2. Add `[include box_hh.cfg]` at the top.

3. Remove the `[hall_filament_width_sensor]` section

4. Make sure Happy Hare files were included during install: `[include mmu/base/*.cfg]` & `[include mmu/optional/client_macros.cfg]`

## `[gcode_macro.cfg]` changes

1. In PRINT_START we need to change Box detection logic:
```bash
[gcode_macro PRINT_START]
gcode:
    AUTOTUNE_SHAPERS
    TOOL_CHANGE_END
    DISABLE_ALL_SENSOR
    CLEAR_PAUSE

    {% set bedtemp = params.BED|int %}
    {% set hotendtemp = params.HOTEND|int %}
    {% set chambertemp = params.CHAMBER|default(0)|int %}
    {% set extruder = params.EXTRUDER|default(0)|int %}
    SET_GCODE_OFFSET Z=0 # Zero out z-offset 
    M104 S0

    M106 P2 S0
    M106 P3 S0
    M106 S255
    {% if "xyz" in printer.toolhead.homed_axes %}
        G0 X50 Y50 F6000
    {% endif %}
    G28      
    M141 S{chambertemp}
    M140 S{bedtemp}    
    M106 S0
    {% if printer.mmu.num_gates >= 4 %} 
        SAVE_VARIABLE VARIABLE=load_retry_num VALUE=0 # saved variables probably need a rework
        SAVE_VARIABLE VARIABLE=retry_step VALUE=None
        SAVE_VARIABLE VARIABLE=is_tool_change VALUE=0
        {% for i in range(16) %}
            SAVE_VARIABLE VARIABLE=runout_{i} VALUE=0
            G4 P100
        {% endfor %}
        SAVE_VARIABLE VARIABLE=extrude_state VALUE=-1
        {% if printer.mmu.enabled == 1 %}
            BOX_PRINT_START EXTRUDER={extruder} HOTENDTEMP={hotendtemp}
            M400
            EXTRUSION_AND_FLUSH HOTEND={hotendtemp}
        {% endif %}
    {% endif %}
    M106 S0
    CLEAR_NOZZLE HOTEND={hotendtemp}
    M190 S{bedtemp}     
#    M141 S{chambertemp}    
    M104 S140
    Z_TILT_ADJUST
    G29
    G0 Z50 F600
    G0 X5 Y5  F6000
    
    {% if chambertemp == 0 %}
        M106 P3 S255
    {% endif %}
    M109 S{hotendtemp}
    M141 S{chambertemp}    
    M204 S10000
    SET_PRINT_STATS_INFO CURRENT_LAYER=1
    ENABLE_ALL_SENSOR
    save_last_file

```

## Slicer settings

None! The mod is intended to be transparent for the slicer. If your gcode works with a stock Plus4, it should work with Happy Hare. 
Tested on Orca Slicer using the following g-codes, which are meant to replicate the Qidi Slicer profile.

## Additional help

Refer to the [Happy Hare documentation](https://github.com/moggieuk/Happy-Hare/wiki/Slicer-Setup).

## User interface

If you want to have a section in your printer web interface, install baseline fluidd or mainsail, which both have HH implemented.

[Happy Hare docs](https://github.com/moggieuk/Happy-Hare/wiki)