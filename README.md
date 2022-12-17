# JubileeKlipper
Klipper configuration for the Jubilee3d toolchanger using a BigTreeTech Octopus Pro board and the Filastruder/LDO Jubilee kit. 

The base toolchanging macros are inspired by [SteveJWallace's config](https://github.com/SteveJWallace/JubileeKlipper). They have been extended by [Xonman](https://github.com/Xonman/JubileeKlipper) to make the Tx commands more modular, and make specific tool/extruder configuration more maintainable.

The exact Jubilee3d setup this config is maintained for is:
- Standard LDO motors (42STH48-2504AC for X/Y, 3x 42STH47-1684MAC for Z)
- X,Y,Z,Z,Z,REL plugged into Motor0 through Motor5, in that order
- 2x Baby Bullet toolheads using 42mm toolboards connected via USB (not CAN!) (2x BTT EBB42)

## Tool posts
The philosophy of this config is to separate tool post and extruder. This is intended to allow for swapping tools in and out from each physical tool location without requiring huge rework.
In the `docks` folder are T0-T4.cfg files which define the physical location of each dock, and a variable named `variable_extruder` which defines which specific tool is located at that dock.
The settings and offsets for tools are defined separately in the corresponding config found in the `tools` folder.

## Tools
Tools are defined individually under the `tools` folder. This keeps each tool's config isolated and makes it easier to make changes. Unfortunately each extruder must be assigned a numeric extruder value rather than a name due to klipper's limited multi-extruder support. They do not need to be sequential, so removing a tool from rotation doesn't require re-numbering all other tools.
At the end of each file is a macro named `extruder<x>_values` which stores variables for:
- Specific tool offsets (including X, Y and Z)
- Purge Y location (controls where in Y to park the tool to purge filament during toolchange)
- Part fan assigned to the extruder
- Input shaping values specific to this extruder

These values are read by the Tx commands during tool change.

One thing to note is that klipper's multi-extruder support does not support tying a part fan to an extruder. To setup a part fan per extruder use the `variable_fan` variable in values. See `tools/crew-white.cfg` for an example.

## Macros
There are a few unexpected macros included.

### macros/temps.cfg
Klipper generates a M104 Tx command to handle multiple extruders, however these are numbered sequentually and not tied to the extruder number. This is a problem in cases where you define `extruder`, `extruder1` and `extruder3` but skip `extruder2` - `extruder3` is now controlled with `M104 T2` and not `M104 T3` as you might expect.
The override macros in this file look up the extruder tied to the toolpost that corresponds to Tx in M104/M109 which gives a more predictable behaviour.

### macros/fans.cfg
As outlined under Tools above Klipper does not natively support multiple part fans. The macros in this file will take the `P` param in M106/M107 commands and look up the extruder tied to the corresponding tool dock. If that extruder defines a `fan` variable in it's `extruder_values` macro. If no `fan` variable is supplied then the default M104 behaviour will apply by controlling the global part fan. If no `P` param is supplied then the currently active extruder is used.

### macros/shaping.cfg
A macro named `REFRESH_INPUT_SHAPING` lives in this file. When called the macro will look up the currently active extruder and apply it's custom input shaping values. If none are set then it defaults to the values set in the global config.
See `tools/crew-white.cfg` and `tools/takoto-1.cfg` as an example.
This macro is called by the tool pickup macro so that input shaping is applied whenever a tool is changed.

