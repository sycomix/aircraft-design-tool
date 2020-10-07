# Aircraft Design Tool

- [Introduction](#introduction)
- [Modules](#modules)
- [Data I/O](#data-i/o)
- [Project File Structure](#project-file-structure)
  - [Concept Selection](#concept-selection)
  - [Mission Profile](#mission-profile)
    - [Mission Segments](#mission-segments)
  - [Vehicle Configuration](#vehicle-configuration)
    - [Vehicle Components](#vehicle-components)
    - [Global Properties](#global-properties)
  - [Energy Network](#energy-network)

## Introduction

Aircraft design tool created for the MECH 475 course at the University of Victoria.

You can use this tool to help you complete the conceptual design of an aircraft. You can define an arbitrarily complex mission profile and assemble an aircraft with an arbitrary number of lifting and non-lifting surfaces.

With an initial set of assumptions given as input, you can iteratively refine a design of choice and obtain a set of performace parameters as an output.

It follows the course outline very closely, with an initial AHP process for design selection, definition of mission profile, calculation of aircraft weight, design point selection and calculation of power and aerodynamic characteristics.

## Modules

The program includes the following design modules, each one responsible for a single task of the design process.

**`ahp`**

Implements the analytic hierarchy process (AHP) to let you organize and analyze a complex design selection decision, based on specified criteria and relative weights between them.

**`build_mission`**

Given a provided mission profile, calculates time and range of the overall mission and each individual segment.

**`plot_mission`**

Plot the aircraft mission profile.

**`mtow`**

Calculates the Maximum Take-off Weight of the aircraft.

**`design_point`**

Plots the forward flight and vertical flight design spaces and prompts the user to pick a design point.

**`lift_slope`**

Computes lift coefficients for each individual lifting surface of the aircraft.

**`drag_buildup`**

Computes the drag coefficient for each lifting (wing) and non-lifting (fuselage) surfaces of the aircraft and determines the total aircraft drag.


## Data I/O

Aircraft data is defined in a project file that is read by the `load_project` function of the program. The project file uses [JSON](https://www.json.org/json-en.html) notation. This data is stored as a MATLAB struct within the program and gets transfered around between modules. Optionally, the data can be saved at the end of the program execution by the function `save_project`.

## Project File Structure

The project is split into three different main groups: _concept selection_, _mission profile definition_, and _vehicle configuration_.

```json5
{
    "concept": {
        ...
    },
    "mission": {
        ...
    },
    "aircraft": {
        ...
    }
}
```

### Concept Selection

The concept selection section implements the analytic hierarchy process (AHP) and lets you specify different design concepts and weight them based on specified criteria (categories). The pair-wise weight between criteria is specified and the designs are ranked based on their calculated relative weight.

```json5
{
    "concept": {
        "categories": {
            ...
            "categories": [
                ...
            ]
        }
    },
    "designs": [
        ...
    ]
}
```

### Mission Profile

A mission profile is built by combining piecewise continuous segments together. Each segment requires a minimum number of parameters to be well defined. Extra parameters can be provided per segment, but will not affect the calculations necessary for that segment.

#### Mission Segments

There is a set of allowed mission segments that can be combined together to form a mission profile.

```json5
{
    "mission": {
        "segments": [
            ...
        ]
    }
}
```

**Taxi**

In a taxi segment the aircraft is on the runway waiting to take-off. The engines are on, so energy is being consumed (in the form of fuel or electric energy).

```json5
{
    "type": "taxi",
    "energy_network": "Fuel-based Energy Network",
    "time": 120.0, // Taxi time (s)
    "altitude": 0.0 // Taxi altitude (m)
}
```

**Hover**

In a hover situation, the aircraft maintains altitude and has zero forward velocity.

```json5
{
    "type": "hover",
    "energy_network": "Fuel-based Energy Network",
    "time": 10.0, // Taxi time (s)
    "altitude": 500.0 // Taxi altitude (m)
}
```

**Transition**

During a transition segment, the aircraft goes from a vertical climb condition to a forward flight condition at constant altitude.

```json5
{
    "type": "transition",
    "energy_network": "Fuel-based Energy Network",
    "altitude": 500.0, // Transition altitude (m)
    "transition_angle": 40.0, // Transition angle (deg)
    "time": 120.0, // Transition time (s)
    "velocity": [0.0, 40.0] // Transition velocity range (m/s)
}
```

**Climb**

During a climb segment, the aircraft climbs at a constant climb speed and angle.

```json5
{
    "type": "climb",
    "energy_network": "Fuel-based Energy Network",
    "velocity": 40.0, // Climb target velocity (m/s)
    "altitude": [500.0, 2500.0], // Climb altitude range (m)
    "angle": 7.2 // Climb angle (deg)
}
```

**Vertical Climb**

During a vertical climb segment, the aircraft climbs at a constant climb speed perpendicular to the ground.

```json5
{
    "type": "vertical_climb",
    "energy_network": "Fuel-based Energy Network",
    "velocity": 8.0, // Vertical climb velocity (m/s)
    "altitude": [0.0, 500.0] // Vertical climb altitude range (m)
}
```

**Acceleration**

In an acceleration segment, the aircraft changes speed while maintainng a constant altitude.

```json5
{
    "type": "acceleration",
    "energy_network": "Fuel-based Energy Network",
    "velocity": 15.0, // Acceleration target velocity (m/s)
    "altitude": 500 // Acceleration altitude (m)
}
```

**Cruise**

In a cruise segment, the aircraft flights at constant altitude and speed for a given range.

```json5
{
    "type": "cruise",
    "energy_network": "Electric-based Energy Network",
    "velocity": 50.0, // Cruise velocity (m/s)
    "range": 80000.0, // Cruise range (m)
    "altitude": 2500.0 // Cruise altitude (m)
}
```

**Hold**

The hold segment is equivalent to a cruise segment, but the hold time is used instead of the cruise range.

```json5
{
    "type": "hold",
    "energy_network": "Hybrid Energy Network",
    "velocity": 40.0, // Hold velocity (m/s)
    "time": 300.0, // Loiter time (s)
    "altitude": 1000.0 // Hold altitude (m)
}
```

**Descent**

During a descent segment, the aircraft descends at a constant descent speed and angle.

```json5
{
    "type": "descent",
    "energy_network": "Fuel-based Energy Network",
    "velocity": -6.0, // Descent velocity (m/s)
    "altitude": [2500.0, 400.0], // Descent altitude range (m)
    "angle": -5.0 // Descent angle (deg)
}
```

**Vertical Descent**

During a vertical descent segment, the aircraft descends at a constant descent speed perpendicular to the ground.

```json5
{
    "type": "vertical_descent",
    "energy_network": "Fuel-based Energy Network",
    "velocity": -6.0, // Vertical descent velocity (m/s)
    "altitude": [400.0, 0.0] // Vertical descent altitude range (m)
}
```

**Landing**

In a landing segment, the aircraft touches down and decelerates to a complete stop.

```json5
{
    "type": "landing",
    "energy_network": "Fuel-based Energy Network",
    "time": 120.0, // Landing time (s)
    "altitude": 0.0 // Landing altitude (m)
}
```

**Load step**

In a load step segment, payload is added to or dropped from the aircraft instantaneously. Positive values of load mass are added to the aircraft and negative values are dropped from the aircraft.

```json5
{
    "type": "load_step",
    "mass": 10, // Load mass (kg)
}
```

**⚠️ NOTES:**

- Some segments cannot operate using all types of energy sources.
- Some segments require a range of values for altitude.
- Units must be consistent throught the definition of the project file.

### Vehicle configuration

A vehicle configuration is built by defining the different components that make up the whole vehicle. Components can be lifting (wing) and non-lifting (fuselage) surfaces, mass points, propulsion systems and energy sources.

```json5
{
    "vehicle": {
        "components": [
            ...
        ]
    }
}
```

#### Vehicle Components

**Point Mass**

A point mass has no volume and simply adds mass to the vehicle.

```json5
{
    "name": "Passengers",
    "type": "point_mass",
    "mass": 10, // Mass value (kg)
    // "inertia": [
    //     [1, 2, 3],
    //     [2, 3, 4],
    //     [3, 4, 5]
    // ], // Inertia (kg m^2) - NOT USED AT THE MOMENT
    // "position": [10, 20, 30] // Position of point mass in the vehicle (m) - NOT USED AT THE MOMENT
}
```

**Fuselage**

A fuselage is a non-lifting surface with specified length, diameter, wetted area and mass.

```json5
{
    "name": "Fuselage",
    "type": "fuselage",
    "interf_factor": 1.0, // Fuselage interference factor
    "diameter": 1.0, // Fuselage diameter (m)
    "length": 4.0, // Fuselage length (m)
    "area": 2.0, // Surface are in contact with flow (wetted area) (m^2)
    "mass": 1500 // Fuselage mass (kg)
}
```

**Wing**

A wing is a lifting surface with specified geometric properties, airfoil profile and mass properties.

```json5
{
    "name": "Main Wing",
    "type": "wing",
    "interf_factor": 1.0,
    "aspect_ratio": 5.0,
    "span": 10.0,
    "mean_chord": 1.5,
    "airfoil": {
        "type": "naca0012",
        "tc_max": 0.15,
        "xc_max": 0.3,
        "cl_aa": 6.2,
        "cl_max": 2.0
    },
    "sweep_le": 10.0,
    "sweep_c4": 15.0,
    "sweep_tc_max": 20.0,
    "area_ref": 15.0,
    "area_wet": 30.0,
    "mass": 400
}
```

**Engine**

An engine is specified by its efficiency, specific fuel consumption and mass. And engine can be part of a larger 

```json5
{
    "name": "Prop Engine",
    "type": "engine",
    "efficiency": 0.8, // Engine efficiency
    "specific_fuel_consumption": 4.25e-5, // Specific Fuel Consumption (1/s)
    "mass": 200.0 // Engine mass (kg)
}
```

**Motor**

A motor is specified by its efficiency and mass.

```json5
{
    "name": "Electric motor",
    "type": "motor",
    "efficiency": 0.9, // Motor efficiency
    "mass": 200.0 // Motor mass (kg)
}
```

**Rotor**

A rotor block needs its radius, solidity ratio and efficiency to be specified.

```json5
{
    "name": "Propeller",
    "type": "propeller",
    "radius": 10, // Rotor radius (m)
    "solidity_ratio": 0.5, // Solidity ratio
    "efficiency": 0.8, // Rotor efficiency (kg)
}
```

**Fan**

A fan block needs its radius and efficiency to be specified.

```json5
{
    "name": "Rotor",
    "type": "rotor",
    "radius": 10, // Fan radius (m)
    "efficiency": 0.8, // Fan efficiency
}
```

**Battery**

A battery block is defined by its mass, efficiency and an optional reserve percentage.

```json5
{
    "name": "Battery",
    "type": "battery",
    "mass": 10, // Battery mass (kg)
    "efficiency": 0.8, // Battery efficiency
    "reserve": 0.2 // Battery reserve (%)
}
```

**Fuel Tank**

A fuel tank is defined by an optional reserve percentage.

```json5
{
    "name": "Fuel Tank",
    "type": "fuel_tank",
    "reserve": 0.2, // Fuel tank reserve (%)
}
```

**Efficiency**

An efficinecy block simply multiplies the network by the given efficinecy.

```json5
{
    "name": "Efficienncy",
    "type": "efficiency",
    "efficiency": 0.8, // Efficiency
}
```

#### Global Properties

Global properties of the aircraft must also be defined in the project file. For example, the the main wing component of the aircraft must be referenced in the `aircraft` section of the file, so that the program knows what reference area to use for the calculation of aerodynmic coefficients.

```json5
{
    "vehicle": {
        "components": [
            ...
        ],
        "main_wing": "Main Wing"
    }
}
```

### Energy Networks

Simple energy networks can be defined by aggregating vehicle powerplant components together. The efficinecy of the energy network is automatically calculated from the individual component efficinecies.

```json5
{
    "energy" : {
        "networks": [
            ...
        ]
    }
}
```

A simple fuel-based energy network can be defined like this.

```json5
{
    "name": "Fuel-based Energy Network",
    "network" : [
        {
            "name": "Fuel Tank"
        },
        {
            "name": "Engine"
        },
        {
            "name": "Rotor"
        }
    ]
}
```

The names are references to vehicle components defined in the previous section. The efficiency of the network is calculated from the individual component efficiencies.

A more complicated network could be defined like this

```json5
{
    "name": "Hybrid series Energy Network",
    "network" : [
        {
            "name": "Fuel Tank"
        },
        {
            "name": "ICE"
        },
        {
            "name": "Generator"
        },
        {
            "name": "Electric motor"
        },
        {
            "name": "Rotor"
        }
    ]
}
```

## Project File

The mission profile and aircraft configuration need to be defined in a project file. The project file uses [JSON](https://www.json.org/json-en.html) notation. An example project file is as follows (with comments added):

```json5
{
    "mission": {
        "mf_struct": 0.24, // Structural mass fraction
        "segments": [
            {
                "type": "taxi",
                "energy": {
                    "type": "electric"
                },
                "time": 600.0, // Taxi time (s)
                "altitude": 0.0 // Taxi altitude (m)
            },
            {
                "type": "vertical_climb",
                "energy": {
                    "type": "electric"
                },
                "velocity": 20.0, // Rate of climb (m/s)
                "altitude": [ 0.0, 1000.0] // Altitude range (m)
            },
            {
                "type": "cruise",
                "propulsion": {
                    "type": "prop"
                },
                "energy": {
                    "type": "electric"
                },
                "velocity": 100.0, // Cruise velocity (m/s)
                "range": 5000.0, // Cruise range (m)
                "altitude": 1000.0 // Cruise altitude (m)
            },
            {
                "type": "descent",
                "energy": {
                    "type": "electric"
                },
                "velocity": -60.0, // Rate of descent (m/s)
                "altitude": [ 1000.0, 100.0], // Altitude range (m)
                "angle": -20.0 // Descent angle (deg)
            },
            {
                "type": "drop",
                "mass_drop": 1000.0,
                "time": 0.0, // Assuming instantaneous drop
                "altitude": 100.0 // Drop altitude (m)
            },
            {
                "type": "hold",
                "propulsion": {
                    "type": "prop"
                },
                "energy": {
                    "type": "fuel",
                    "sfc": 4.25e-5 // Specific fuel Consumption
                },
                "velocity": 40.0, // Loiter velocity (m/s)
                "time": 300.0, // Loiter time (s)
                "altitude": 100.0 // Loiter altitude (m)
            },
            {
                "type": "load",
                "mass_reload": 1000.0, // Load mass
                "time": 0.0, // Assuming instantaneous reload
                "altitude": 100.0 // Drop altitude (m)
            },
            {
                "type": "climb",
                "propulsion": {
                    "type": "prop"
                },
                "energy": {
                    "type": "fuel"
                },
                "velocity": 50.0, // Rate of climb (m/s)
                "altitude": [100.0, 1000.0], // Altitude range (m)
                "angle": 20.0 // Climb angle (deg)
            },
            {
                "type": "cruise",
                "propulsion": {
                    "type": "jet"
                },
                "energy": {
                    "type": "fuel",
                    "sfc": 4.25e-5 // Specific fuel Consumption
                },
                "velocity": 100.0, // Cruise velocity (m/s)
                "range": 5000.0, // Cruise range (m)
                "altitude": 1000.0 // Cruise altitude (m)
            },
            {
                "type": "vertical_descent",
                "energy": {
                    "type": "electric"
                },
                "velocity": -50, // Rate of descent (m/s)
                "altitude": [1000.0, 0.0] // Altitude range (m)
            },
            {
                "type": "landing",
                "energy": {
                    "type": "electric"
                },
                "time": 600.0, // Landing time (s)
                "altitude": 0.0 // Landing altitude (m)
            }
        ]
    },
    "aircraft": {
        "mass_to": 4000.0, // Initial estimate for maximum take-off mass
        "mass_payload": 680.0, // Payload mass
        "propulsion": {
          "gear_efficiency": 0.6, // Gearbox efficiency
          "em_efficiency": 0.9, // Electric motor efficiency
          "esc_efficiency": 0.8, // Electronic speed controller efficiency
          "config": "series", // Propulsion configuration: 'series', 'parallel'
          "dist_efficiency": 0.6, // Power distribution system efficiency
          "prop_efficiency": 0.8, // Propeller efficiency
          "k_i": 1.15, // Factor to account for losses in hover
          "c_d": 0.02, // Average blade drag coefficient
          "ss": 0.08, // Main rotor solidity ratio
          "tip_velocity": 240.1, // Speed at the blade tip (m/s)
          "fm": 0.6, // Figure of merit
          "a": 10.0, // Rotor area (m^2)
          "n": 4.0, // Number of rotors
          "tt_tilt": 40.0, // Final rotor tilt angle during transition (deg)
          "fwd_power": 500000.0, // Engine power for forward flight (W)
          "vert_power": 500000.0, // Engine power for vertical flight (W)
          "mass": 500.0 // Engine mass (kg)
        },
        "energy": {
            "e_spec_batt": 360000.0, //Battery specific energy (J/kg)
            "batt_efficiency": 0.9, // Battery efficiency
            "f_usable_batt": 0.9, // Usable capacity, i.e. ratio between usable capacity and stored energy of the battery
            "reserve_batt": 1.2, // Battery reserve factor
            "reserve_fuel": 1.06 // Fuel reserve factor
        },
        "performance": {
            "c_d0": 0.02, // Aircraft base drag
            "aspect_ratio": 5.0, // Aspect ratio
            "ld_max": 15.0, // Maximum L/D ratio
            "dl": 700.0, // Disk loading (N/m^2)
            "area_ref": 15.0,
            "area_wet": 30.0
        },
        "fuselages": [
            {
                "interf_factor": 1.0,
                "diameter": 1.0,
                "length": 4.0,
                "area_ref": 1.0,
                "area_wet": 2.0
            }
        ],
        "lifting_surfaces": [
            {
                "type": "wing",
                "interf_factor": 1.0,
                "aspect_ratio": 5.0,
                "span": 10.0,
                "mean_chord": 1.5,
                "airfoil": {
                    "type": "naca0012",
                    "tc_max": 0.15,
                    "xc_max": 0.3,
                    "cl_aa": 6.2,
                    "cl_max": 2.0
                },
                "sweep_le": 10.0,
                "sweep_c4": 15.0,
                "sweep_tc_max": 20.0,
                "area_ref": 15.0,
                "area_wet": 30.0
            },
            {
                "type": "h-tail",
                "interf_factor": 1.0,
                "aspect_ratio": 5.0,
                "span": 2.0,
                "mean_chord": 0.5,
                "airfoil": {
                    "type": "naca0012",
                    "tc_max": 0.15,
                    "xc_max": 0.3,
                    "cl_aa": 6.2,
                    "cl_max": 2.0
                },
                "sweep_le": 10.0,
                "sweep_c4": 15.0,
                "sweep_tc_max": 20.0,
                "area_ref": 1.0,
                "area_wet": 2.0
            },
            {
                "type": "v-tail",
                "interf_factor": 1.0,
                "aspect_ratio": 5.0,
                "span": 2.0,
                "mean_chord": 1.0,
                "airfoil": {
                    "type": "naca0012",
                    "tc_max": 0.15,
                    "xc_max": 0.3,
                    "cl_aa": 6.2,
                    "cl_max": 2.0
                },
                "sweep_le": 10.0,
                "sweep_c4": 15.0,
                "sweep_tc_max": 20.0,
                "area_ref": 2.0,
                "area_wet": 4.0
            }
        ]
    }
}
```
