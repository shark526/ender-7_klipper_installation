# SSH to RPi4 running MainsailOS
  - usb cable connected from RPi to printer
Confirm printer serial COM port
> ls /dev/serial/by-id/*
  - return: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
  
###20320508 update, lost connection to mainboard
  - usb connection changed after software update
  - > sudo dmesg -w
  - locate new usb serial (ttyUSB0)
  - update printer.cfg (serial: /dev/ttyUSB0)

# Connect to Mainsail web interface

Create printer.cfg for Klipper (/home/pi/printer_data/config/printer.cfg)
- > Machine > Create File > printer.cfg

//**printer.cfg**\\ (minimum config with bltouch)
[include mainsail.cfg]

[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
restart_method: command
#baud: 250000

[printer]
kinematics: corexy
max_velocity: 500          # max feedrate (E7:500)(Other:300)
max_accel: 3000
max_z_velocity: 10
max_z_accel: 100

[stepper_x]
step_pin: PC2
dir_pin: PB9
enable_pin: !PC3
rotation_distance: 32
microsteps: 32
endstop_pin: ^PA5
position_endstop: 0
position_max: 250
homing_speed: 50

[stepper_y]
step_pin: PB8
dir_pin: !PB7
enable_pin: !PC3
rotation_distance: 32
microsteps: 32
endstop_pin: ^PA6
position_endstop: 250
position_max: 250
homing_speed: 50

[stepper_z]
step_pin: PB6
dir_pin: PB5
enable_pin: !PC3
rotation_distance: 8
microsteps: 16
#endstop_pin: ^PA7                            # disable to use BLTouch
#position_endstop: 3.0                        # disable to use BLTouch
endstop_pin: probe:z_virtual_endstop          # enable to use BLTouch
position_min: -3.5                            # enable to use BLTouch
position_max: 300

[extruder]
step_pin: PB4
dir_pin: !PB3
enable_pin: !PC3
microsteps: 16
rotation_distance: 23.564                    # Re-tune to suit
full_steps_per_rotation: 200
nozzle_diameter: 0.400
filament_diameter: 1.750
max_extrude_only_distance: 200.0
pressure_advance = 0                         # 0 to disable
#HOT END VARIABLES
heater_pin: PA1
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC5
#control: pid                                #(PID_CALIBRATE HEATER=extruder TARGET=200)
pid_kp = 74.327                              # updated after PID_CALIBRATE
pid_ki = 0.985                               # updated after PID_CALIBRATE
pid_kd = 1401.994                            # updated after PID_CALIBRATE
min_temp: 0
max_temp: 230

[heater_bed]
heater_pin: PA15
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC4
#control: pid                                #(PID_CALIBRATE HEATER=heater_bed TARGET=60)
pid_kp = 19.588                              # updated after PID_CALIBRATE
pid_ki = 1.005                               # updated after PID_CALIBRATE
pid_kd = 95.493                              # updated after PID_CALIBRATE
min_temp: 0
max_temp: 75

[fan]
pin: PA0          # Hotend additional fans

[filament_switch_sensor filament_sensor]
pause_on_runout: true
switch_pin: !PA4

[bltouch]                 # enable for BLTouch (requires self_test after each firmware restart)
sensor_pin: ^PB1
control_pin: PB0
z_offset: 2.550          # initial safe value, get correct value by PROBE_CALIBRATE
x_offset: -28.28
y_offset: -28.28

[safe_z_home]                       # BLTouch home x,y axis then move to centre and home z
home_xy_position: 125, 125          # Change coordinates to the center of your print bed
speed: 50
z_hop: 10                           # Move up 10mm
z_hop_speed: 5

[bed_mesh]
speed: 120
horizontal_move_z: 10       #lift z between movements
mesh_min: -25, -25          # x, y (x: 0-28.28=-28.28, -28.28+250= 221.72)
mesh_max: 190, 190
probe_count: 4
//**printer.cfg**\\

# Create Klipper.bin to flash microcontroller

### SSH to RPi4 running MainsailOS (user: pi)
  - usb cable connected from RPi to printer
Confirm printer serial COM port
> ls /dev/serial/by-id/*
  - return: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
  
### Navigate to Klipper directory
> cd ~/klipper/

### Klipper configure firmware
> make menuconfig
  - !PA15 required to prevent bed heater turning on automatically at printer start
  
//**make menuconfig**\\ (GUI)
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32) --->
    Processor model (STM32F103) --->
[ ] Disable SWD at startup (for GigaDevice stm32f103 clones)
    Bootloader offset (28Kib bootloader) --->
    Clock Reference (8 MHz crystal) --->
    Communication interface (Serial (on USART1 PA10/PA9)) --->
(250000) Baud rate for serial port
(!PA15) GPIO pins to set at micro-controller startup
//**make menuconfig**\\

### Compile Klipper firmware for microcontroller
Stop Klipper service
> sudo service klipper stop
Compile firmware
> make
  - File output: ~/klipper/out/klipper.bin
Start Klipper service
> sudo service klipper start

### Recommended to retrive printer settings with M503 before flashing to aid in configuration after flashing Klipper
  - example: > M503, M92 X200.00 Y200.00 Z400.00 E140.00, Axis Steps-per-unit (e-steps)

### Flash firmware to microcontroller
Retrieve compiled firmware (klipper.bin)
  - WinSCP
    - download: ~/klipper/out/klipper.bin

- Rename klipper.bin to unique filename (must not match any previous filename)(add date)
- Upload exampleklipper.bin to micro-sd card
- With printer off ensure usb cable is disconnected (nothing connected during flashing)
- Insert micro-sd card and turn printer on
- Wait for board to flash (wait several min) (screen will no longer display current status)
- Connect usb and login to mainsailOS web interface to confirm flashing is complete

### DO NOT PRINT yet still need to test and calibrate
