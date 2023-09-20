### Determine extruder maximum flow rate(mm^3/s)(maximum printing speed)
  - Maximum Feed rate/Melting rate (extruder)(issue commands from mainsailOS web interface)
    - > M83                #Set e to relative
    - > M109 S210          #Set and wait for nozzle temp [210C]
    
    - Increase the extrusion speed until the extruder starts to lose steps then take last value
    - > G1 E50 F60         #Start at 1mm/s extrude 50mm (F60 = 60mm/min = 1mm/s)
    - > G1 E50 F120        #2mm/s
    - > G1 E50 F180        #3mm/s
    - > G1 E50 F240        #4mm/s
    - > G1 E50 F300        #5mm/s
    - > G1 E50 F360        #6mm/s
    - > G1 E50 F420        #7mm/s
    - > G1 E50 F480        #8mm/s
    - > G1 E50 F540        #9mm/s
    - > G1 E50 F600        #10mm/s losing steps
    
    - Convert mm/min to mm/sec
    - 540 / 60 = 9
    - Max feed rate = 9mm/sec
    
  - Determine filament cross section
    - Diameter: 1.75mm
    - Radius = 1.75 / 2 = 0.875mm
    - Cross section = pi * 0.875^2 = 2.405mm^2
    
  - Approximate maximum flow rate (theoretical limit)
    - maximum_flow_rate(mm^3/sec) = <max_extrusion_speed(mm/sec)> * <filament_cross_section(mm^2)>
    - 9 * 2.405 = 21.645mm^3/sec
    
  - Approximate max speed (theoretical limit)
    - Recommnded line width: 1.2 * nozzle (0.48)
    - speed(mm/sec) = <volumetric_flow(mm^3/sec)> / <layer_height(mm)> / <line_width(mm)>
    - 21.645 / 0.2 / 0.48 = 225.469mm/sec
    
  - Approximate volumetric flow (theoretical limit)
    - volumetric_flow(mm^3/sec) = <speed(mm/sec)> * <layer_height(mm)> * <line_width(mm)>
    - 225.469 * 0.2 * 0.48 = 21.645mm^3
    
### Print and measure for maximum flow
  - print model 20230124_external_speed_test_2_0.2mm_PLA_MINI_23m.gcode
  - gcode details (nozzle: 0.4, line_width: 0.48, layer_height: 0.2, perimeters: 1, Temp_bed: 60, Temp_nozzle: 215)
    - 0-0.5, external perimeters: 60mm/s
    - 0.5-5, external perimeters: 80mm/s
    - 5-10,  external perimeters: 100mm/s
    - 10-15, external perimeters: 120mm/s
    - 15-20, external perimeters: 140mm/s
    - 20-25, external perimeters: 160mm/s
    - 25-30, external perimeters: 180mm/s
    - 30-35, external perimeters: 200mm/s
    - 35-40, external perimeters: 220mm/s
    - 40-45, external perimeters: 240mm/s
    - 45-50, external perimeters: 250mm/s
    
  - Measure from base up to where under extrusion begins
    - 25mm = 160mm/s
    
  - Calculate maximum volumetric flow rate from measurement
    - 160 * 0.2 * 0.48 = 15.36mm^3/sec
  
  - Confirm Calculation (speed)
    - 15.36 / 0.2 / 0.48 = 160mm/sec

  - Max filament feed rate
    - 15.36 / 2.405 = 6.387mm/sec

### For more accurate flow rate calculation CNC Kitchen 
  - (https://www.cnckitchen.com/blog/extrusion-system-benchmark-tool-for-fast-prints)
  - (https://www.printables.com/model/160335-hotened-benchmark-test-g-codes)
  - will require a milligram scale
  
