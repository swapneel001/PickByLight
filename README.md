 
<a href="https://github.com/adricpjw/PickByLight"><img src="https://i.imgur.com/9UanW2G.png"></a>

# Mission Loader

> To load missions in a given sequence 

[![Build Status](http://img.shields.io/travis/badges/badgerbadgerbadger.svg?style=flat-square)](https://travis-ci.org/badges/badgerbadgerbadger) 

- Tested and ran on ROS1 Melodic, Ubuntu 18.04 LTS

---

## Table of Contents 

- [Installation](#installation)
- [Usage](#usage)
- [Documentation](#documentation)
- [Tests](#tests)

---

## Installation

- Clone this repo to your local machine using `https://github.com/adricpjw/PickByLight`
- Make sure this is in your catkin workspace

### Setup

- Catkin make and build the package

```shell
$ cd ~/catkin_ws && catkin_make
```

### Dependant Packages

#### niryo_one_ros
```shell
$ git clone https://github.com/zakimohzani/niryo_one_ros
$ git checkout FinalCameraLightsIntegration
```

---
## Usage

### Adjusting the mission sequence
Before launching the PickByLight launch, ensure that the configurations and parameters are in order by navigating to `/config/params.yaml`
```yaml
---
  lightcmd_topic: /lightcmd
  objDetected_sub_topic: /activeObjects
  lightarray_frame: light_array
  origin_frame: base_link
  num_lights: 10 # the number of lights mounted
  timer_rate: 5 # Hz
  y_tolerance : 0.10 # Distance from object the belt-axis to turn on light (in meters)
  array_width : 0.44 # Length of laser diodes from end to end (in meters)

  #---- FILTERS ---- #
  FIL_BY_X : true
  FIL_BY_Y : true
  FIL_BY_TYPE : false
  

```

Another config file (`/config/lightarray.yaml`) is also important in determining the position of light array.
```yaml
---
  x_pos: 0.50
  y_pos: -0.20
  z_pos: 0.37
  # roll: -45
  roll: 45

  frame_id: light_array
  origin_frame: base_link
  timer_rate: 5
  
```

### Launch the node
You can now load the mission by running:
```shell
$ roslaunch hmiCmd hmiCmd.launch
```


---
## Documentation 

### Filters:

Currently there are three implemented types of filters: 

> Filter by X
This is the most important filter as it is currently directly tied to the command sent to lights. Without it enabled the nothing will be published to `/lightcmd`
It determines which light index should be switch to HIGH by checking the x pose and width of each active object.
```cpp
void filterbyX(vpsi &frames) {
    /* Obtaining Frame Transforms */

    // For each object, obtain its x pose and width 
    double x = transformStamped_.transform.translation.x;
    double width = frame.boundingBox[3];

    // Determine which lights to turn on to cover the entire width of the object
    
    int lower_idx = (x - width / 2) < lower_x_
                        ? 0
                        : ((x - width / 2 - lower_x_) / diode_coverage_);
    int upper_idx = (x + width / 2) < lower_x_
                        ? 0
                        : ((x + width / 2 - lower_x_) / diode_coverage_);
}

```

> Filter by Y
This filter determines whether lights should turn on by checking if the object is within a certain distance from where the lights are shining along the belt-axis.

```cpp
void hmiCtrl::filterbyY(vecWaste &frames) {
    if (fabs(transformStamped_.transform.translation.y - y_boundary_) >
        (y_tolerance_ + length / 2)) {
        //ignore the frame by lazy deletion
    }
}
```

> Filter by Type
This filter determines if the active Object should be ignored based from its plastic type, beit HDPE OR PET


---
## Common Issues





---