# klipper_tmc_autotune
TMC stepper driver autotuning Klipper python extra.

This extra calculates good values for most registers of TMC stepper motor drivers given the datasheet information for the motor.

In particular, it enables StealthChop by default on Z motors and extruders, CoolStep where possible, and correctly switches to full step operation at very high velocities. Where multiple modes are possible, it should choose the lowest power consumption and quietest modes available, subject to the constraints of sensorless homing (which do not allow certain combinations).

# Current status

- Support for TMC 2209, 2240, 5160 at least partially tested. 2130, 2208 and 2660 may work, but are completely untested.
- Sensorless homing known to work on 2209, 2240 and 5160, provided you home fast enough (homing_speed should be numerically greater than the rotation_distance for those axes using sensorless homing). As always, be very careful attempting sensorless homing for the first time.

# Installation

To install this plugin, run the installation script using the following command over SSH. This script will download this GitHub repository to your RaspberryPi home directory, and link the files in the Klipper folder.

```bash
wget -O - https://raw.githubusercontent.com/andrewmcgr/klipper_tmc_autotune/main/install.sh | bash
```

Then, add the following to your printer.cfg, which assumes you have a Voron 2.4. Remove any sections for steppers you don't have (e.g. if you have less than four Z motors), add more if required. Motor names are in the `motor_database.cfg`. If a motor is not listed, add it, taking careful note of the units. PRs for more motors gratefully accepted.
```
[autotune_tmc stepper_x]
motor: ldo-42sth48-2004mah
sg4_thrs: 80
# Sensorless homing threshold, tune if using sensorless
voltage: 24.0
# Motor supply voltage for this stepper driver
stealth: False
# Enable StealthChop on this driver
extra_hysteresis: 3
# Add some hysteresis to make the motor quieter (use only as much as necessary)
[autotune_tmc stepper_y]
motor: ldo-42sth48-2004mah
sg4_thrs: 10
# Sensorless homing threshold, tune if using sensorless
voltage: 24.0
# Motor supply voltage for this stepper driver
stealth: False
# Enable StealthChop on this driver
extra_hysteresis: 3
# Add some hysteresis to make the motor quieter (use only as much as necessary)

[autotune_tmc stepper_z]
motor: ldo-42sth48-2004ac
[autotune_tmc stepper_z1]
motor: ldo-42sth48-2004ac
[autotune_tmc stepper_z2]
motor: ldo-42sth48-2004ac
[autotune_tmc stepper_z3]
motor: ldo-42sth48-2004ac

[autotune_tmc extruder]
motor: ldo-36sth20-1004ahg
```

This is compatible with homing overrides for sensorless homing, but take care to tune sg4_thrs and/or sgt through the autotune section if using 2240 or 5160 drivers, rather than by attempting to use gcode (that will not error, but won't do anything either as the autotune will override it). It should also work with any other homing overrides. Also use sg4_thrs via autotune with 2209, the code will correctly apply the value to the 2209's sgthrs field.

For sensorless homing, use the sgt field as the homing threshold for 2240 and 5160, the sg4_thrs for 2209 or 2260. sg4_thrs is also the CoolStep current regulation threshold, and can be used to tune that on 2209, 2240 and 5160 (on 2209, the same value as is used for homing will work correctly for CoolStep). A default value of sg4_thrs = 80 is usually reasonably close for CoolStep.

Add a `overvoltage_vth` if you have 2240 or 5160 drivers and wish to enable the overvoltage snubber (BTT SB2240 users should use `overvoltage_vth` for the extruder, reading the actual toolhead voltage and adding 0.8 V)

If you want, add the following to your moonraker.conf, which will enable automatic updates:
```
[update_manager klipper_tmc_autotune]
type: git_repo
channel: dev
path: ~/klipper_tmc_autotune
origin: https://github.com/andrewmcgr/klipper_tmc_autotune.git
managed_services: klipper
primary_branch: main
install_script: install.sh
```

