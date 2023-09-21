My configs and macros for 2in1out Phateus Taichi hotend
DD configuration with 2x Sailfin extruder

![IMG_20230921_111743](https://github.com/stenc55/Klipper-Taichi/assets/68291385/6bb01c93-cdf1-4674-9d31-9d43e378dc07)




Prusa slicer settings
Disable pressure_advance insert from Prusa slicer

![image](https://github.com/stenc55/Klipper-Taichi/assets/68291385/386f90eb-79a6-4eca-a2e0-721438306721)

Taichi parameters

![image](https://github.com/stenc55/Klipper-Taichi/assets/68291385/0a7c6689-1078-465a-836c-3671f1524ffc)


Start print G code (for Prusaslicer, not Klipper!!!!)

G90 ; use absolute coordinates
M83 ; extruder relative mode
M104 S150 ; set temporary nozzle temp to prevent oozing during homing
M140 S{first_layer_bed_temperature[0]} ; set final bed temp
G4 S30 ; allow partial nozzle warmup
G28 ; home all axis
G29  ;if you need to level bed each print, comment out if not
BED_MESH_PROFILE LOAD=p1

G1 Z50 F240
G1 X2.0 Y12 F3000
M104 S{first_layer_temperature[0]} ; set final nozzle temp
M190 S{first_layer_bed_temperature[0]} ; wait for bed temp to stabilize
M109 S{first_layer_temperature[0]} ; wait for nozzle temp to stabilize 
;T0
;T1
LOAD_FILAMENT
G1 Y30 Z0.28 F240
G92 E0
G1 X2.0 Y155 E10 F1500 ; prime the nozzle
G1 X2.3 Y140 F5000
G92 E0
G1 X2.3 Y30 E10 F1200 ; prime the nozzle
G1 Z4 F240
G92 E0

#NOTE! #####################################################################

;T0
;T1    Will activate and with single or both extruders (starting extruder T0) depending if object is single or two materials

;T0
T1     Will activate and with single or both extruders (starting extruder T1) depending if object is single or two materials and if material to use is already loaded in T1 extruder and you don't feel like swapping it to T0 extruder :D.

____________________________________________________________________________________________________________________________________


# Klipper-Taichi


[delayed_gcode activate_default_extruder]
initial_duration: 1
gcode:
  ACTIVATE_EXTRUDER EXTRUDER=extruder

[gcode_macro T0]
gcode:
  # Deactivate stepper in my_extruder_stepper
  SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder1 EXTRUDER=
  # Activate stepper in extruder
  SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder EXTRUDER=extruder

[gcode_macro T1]
gcode:
  # Deactivate stepper in extruder
  SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder EXTRUDER=
  # Activate stepper in my_extruder_stepper
  SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder1 EXTRUDER=extruder

[gcode_macro ACTIVATE_EXTRUDER]
description: Replaces built-in macro for a X-in, 1-out extruder configuration SuperSlicer fix
rename_existing: ACTIVATE_EXTRUDER_BASE
gcode:
  {% if 'EXTRUDER' in params %}
    {% set ext = params.EXTRUDER|default(EXTRUDER) %}
    {% if ext == "extruder"%}
      {action_respond_info("Switching to extruder0.")}
      T0
    {% elif ext == "extruder1" %}
      {action_respond_info("Switching to extruder1.")}
      T1
    {% else %}
      {action_respond_info("EXTRUDER value being passed.")}
      ACTIVATE_EXTRUDER_BASE EXTRUDER={ext}
    {% endif %}
  {% endif %}
__________________________________________________________________________________________________________________________________

#Phateus Taichi 
#Those two macros are called from Klipperscreen-> Extrude-> Load/ Unload filament

[gcode_macro LOAD_FILAMENT]
#description: This will move filament to the nozzle assuming parked position (TaiChi hotend)
gcode:
  M83
  G1 E8.6000 F120
  G1 E30.1000 F207
  G1 E4.3000 F145
  #G1 E60 F145
  M82

[gcode_macro UNLOAD_FILAMENT]
#description: This will form tip and move filament to parked position assuming filament is at nozzle (TaiChi hotend)
gcode:
  M83
  G1 E20.0000 F300
  G1 E-15.0000 F1500
  G1 E-9.8000 F2400
  G1 E-2.8000 F1200
  G1 E-1.4000 F720

  G1  E5.0000 F3000
  G1  E-5.0000 F2780
  G1  E5.0000 F3000
  G1  E-5.0000 F2750
  G1  E5.0000 F3000
  G1  E-5.0000 F2700
  G1  E5.0000 F3000
  G1  E-5.0000 F262
  G1  E-16.000 F200
  M82

_________________________________________________________________________________________________________________________________


#KIAUH and other macros


[delayed_gcode bed_mesh_init]
initial_duration: .01
gcode:
  BED_MESH_PROFILE LOAD=p1

[gcode_macro G29]
gcode:
 #G1 E-1 F500
 G28
 BED_MESH_CALIBRATE
 BED_MESH_PROFILE SAVE=p1
 #BED_MESH_PROFILE LOAD=p1
 G1 X20 Y70 Z10 F4000
 @BEDLEVELVISUALIZER
 BED_MESH_OUTPUT

[gcode_macro PID_TUNE_EXTRUDER]
gcode: PID_CALIBRATE HEATER=extruder TARGET=210 WRITE_FILE=1

[gcode_macro PID_TUNE_BED]
gcode: PID_CALIBRATE HEATER=heater_bed TARGET=60 WRITE_FILE=1

[gcode_macro Resonance_compensation]
gcode:
  SET_VELOCITY_LIMIT ACCEL_TO_DECEL=7000
  SET_PRESSURE_ADVANCE ADVANCE=0
  SET_INPUT_SHAPER SHAPER_FREQ_X=0 SHAPER_FREQ_Y=0
  TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5

[gcode_macro Pressure_advance]
gcode:
  SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=1 ACCEL=500
  TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.002
	
[pause_resume]

[display_status]

[gcode_macro CANCEL_PRINT]
rename_existing: BASE_CANCEL_PRINT
gcode:
  G1 X0 Y220 F900
  TURN_OFF_HEATERS
  CLEAR_PAUSE
  M84
  M107
  SDCARD_RESET_FILE
  BASE_CANCEL_PRINT

[gcode_macro PAUSE]
rename_existing: BASE_PAUSE
# change this if you need more or less extrusion
variable_extrude: 1.0
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  ##### set park positon for x and y #####
  # default is your max posion from your printer.cfg
  {% set x_park = printer.toolhead.axis_maximum.x|float - 25.0 %}
  {% set y_park = printer.toolhead.axis_maximum.y|float - 25.0 %}
  ##### calculate save lift position #####
  {% set max_z = printer.toolhead.axis_maximum.z|float %}
  {% set act_z = printer.toolhead.position.z|float %}
  {% if act_z < (max_z - 10.0) %}
  {% set z_safe = 10.0 %}
  {% else %}
  {% set z_safe = max_z - act_z %}
  {% endif %}
  ##### end of definitions #####
  SAVE_GCODE_STATE NAME=PAUSE_state
  BASE_PAUSE
  G91
  G1 E-{E} F2100
  G1 Z{z_safe} F900
  G90
  G1 X{x_park} Y{y_park} F6000
  #G1 E15 F200

[gcode_macro RESUME]
rename_existing: BASE_RESUME
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  ##### end of definitions #####
  G91
  G1 E{E} F2100
  RESTORE_GCODE_STATE NAME=PAUSE_state
  BASE_RESUME
    
[gcode_macro idle_timeout]
gcode:
  timeout: 36000

[gcode_macro BL_Touch_reset]
gcode:
  BLTOUCH_DEBUG COMMAND=reset
  timeout: 5
  BLTOUCH_DEBUG COMMAND=pin_down
  timeout: 5
  BLTOUCH_DEBUG COMMAND=pin_up

[force_move]
#enable_force_move: True

#########################################################################################
#########################################################################################


#Printer.cfg 

########    Note, I am using EBB CAN42. Second extruder driver is on Manta 5P  ############################

[extruder]   #EBB CAN 42
step_pin: EBBCan: PD0
dir_pin: EBBCan: PD1
enable_pin: !EBBCan: PD2
microsteps: 16
rotation_distance: 4.653
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: EBBCan: PB13
sensor_type: EPCOS 100K B57560G104F
sensor_pin: EBBCan: PA3
#control: pid
#pid_Kp: 21.527
#pid_Ki: 1.063
#pid_Kd: 108.982
min_temp: 0
max_temp: 270
pressure_advance: 0.05
min_extrude_temp: 180
max_extrude_only_distance: 700.0
max_extrude_cross_section: 50.0

[tmc2209 extruder]
uart_pin: EBBCan: PA15
run_current: 0.850
stealthchop_threshold: 999999
___________________________________________________________

[extruder_stepper extruder1]  # Manta5P
extruder: extruder
step_pin: PB0
dir_pin: !PB1
enable_pin: !PC4
microsteps: 16
rotation_distance: 4.653 
pressure_advance: 0.05


[tmc2209 extruder_stepper extruder1]
uart_pin: PA6
run_current: 0.850
#diag_pin:
