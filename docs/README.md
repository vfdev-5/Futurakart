# Documentation

Get sources from Github :
```
git clone --recursive https://github.com/RoseTeam/Futurakart.git
```
If you didn't clone with the --recursive flag, then you'll need to manually clone `ackermann_controller`:
```
git submodule update --init --recursive
```

Information on the installation on RPi, please see [here](./RPI-ROS-Installation.md)

## Futurakart ROS packages  

- Core : `futurakart_control`, `futurakart_description`, `futurakart_msgs`, `futurakart_2dnav`
- 3rdParty packages : `ackermann_controller`, `dual_controller_interface`
- Desktop : `futurakart_viz`
- Robot : `futurakart_bringup`, `futurakart_base` 
- Simulator : `futurakart_gazebo`


### Core packages

These packages are destinated for the desktop and the robot.  

####`futurakart_description`
Package contains information about robot using URDF formalism. 
This defines joints and links, robot caracteristics, dimensions etc and can also associate with meshes for gazebo simulation.

**Launch files:**
- `description.launch` starts `robot_state_publisher` node with `futurakart.urdf`
    * Published topics : `/tf, /tf_static`
    * Subscribed topics : `/joint_states`

See docs on [robot_state_publisher](http://wiki.ros.org/robot_state_publisher/Tutorials/Using%20the%20robot%20state%20publisher%20on%20your%20own%20robot)


#### `futurakart_msgs`
Package contains custom messages to exchange with motors hardware part:
- MotorDrive
- MotorFeedback

#### `futurakart_control`

**Nodes:**
- `twist_to_ackermann` : transforms twist messages to ackermann messages (twist.angular.z is directly set to ackermann.steering_angle)
    * Published topics : `/ackermann_cmd`
    * Subscribed topics : `/cmd_vel` (twist command), `/twist_to_ackermann_switch` (enable/disable by a message with integer value : 1 / -1)
- `joy_interpreter_node`

**Controllers:**
- `ackermann_controller` following *ros_controller* formalism
    * Published topics : `/ackermann_controller/odom` 
    * Subscribed topics : `/ackermann_cmd`


**Launch files:**
- `control.launch` starts `twist_to_ackermann` node, indirectly loads `ackermann_controller`, and finally `ekf_localization_node`
    * Published topics : `/ackermann_controller/odom` (by `ackermann_controller`), `/odometry/filtered` (by `ekf_localization_node`)
    * Subscribed topics : `/cmd_vel` (twist command), `/twist_to_ackermann_switch` (enable/disable by a message with integer value : 1 / -1)
    
- `teleop.launch`
- `teleop_joy.launch`

#### `futurakart_2dnav`

Package contains few launch files for mapping and localization purposes: 

- `create_map.launch gmapping:={true|false} rtabmap:={true|false}` to create a map using `gmapping` or `rtabmap`. 
If you use `gmapping` and want to save the map, run `rosrun map_server map_saver -f mymap` before killing the node or use the alias `save_gmap`.
If you use `rtabmap`, the map is automatically saved as `futurakart/futurakart_2dnav/maps/_maps.db`. 

- `odom_navigation.launch`


### 3rdParty packages

#### `ackermann_controller` and  `dual_controller_interface`
are packages developed for [blinky-robot](https://github.com/blinky-robot) and the code is taken from [github](https://github.com/blinky-robot/ackermann_controller) 

We modified **`dual_controller_interface`** to adapt to the newer version of the `ros_control` modules.

Comments on `ackermann_controller` :

- `drive_joints={position, velocity}` : position and velocity of the propulsion rear wheels
- `steering_joints={position}` : angle in radians of the direction
- `steering_pos=steering_joints.position`
- `d_pos` : angle in radians driven by the vehicle
- `d_dist=d_pos * wheel_radius` : distance driven be the vehicle
- `base_length` : track distance between front and rear wheel axles ???
- `d_theta=tan(steering_pos) * d_dist / base_length` : angle ...

#### `twist_to_ackermann` node
is taken from `blinky_control` package


### Robot packages 

These packages are destinated for the robot only. 

Futurakart robot is composed of two Raspberry Pi cards, reponsible for the *mobile* part and the *vision* part
Both RPi cards should have *robot packages* and *core packages*, however the bringup procedure varies depending on the card.

#### `futurakart_bringup` 
The package is responsible to bringup the robot. 

**Nodes:** 
- TODO
- TODO

**Launch files:**
- TODO
- TODO

For instance, we use a simple bringup procedure: 
- Connect with SSH to the robot (*mobile* part RPi)
- Run the following on the robot side to start mobile part
```
roslaunch futurakart_bringup futurakart.launch
```
and it calls launch files from 
* `futurakart_base` (base.launch) to initialize the main *futurakart* node, `futurakart_description` and `futurakart_control` 
* connect to the *vision* part RPi and run the following `roslaunch futurakart_base vision.launch`. See below for details

When the *mobile* part RPi connects with SSH the *vision* part PRi, it uses ssh public keys. 
In case of any errors, make sure that the ssh key is added. To check run on the *vision* part RPi
```
less ~/.ssh/authorized_keys
```
and find your user id.

To add new ssh key, connect to the RPi and run the following, replacing "<key-part> and user@PC" by your data from `~/.ssh/id_rsa.pub`   
```
echo "ssh-rsa <key-part> user@PC" >> ~/.ssh/authorized_keys 
```

**TODO: There is another more 'pro' way to bringup a robot. See for example [here](http://wiki.ros.org/husky_bringup/Tutorials/Install%20Husky%20Software)**

#### `futurakart_base`
The package contains base code to communicate with
- Nucleo card which commands propulsion and direction motors
- Kinect sensor
- ...

**Nodes:** 
- `futurakart_base` 
    * Published topics : `/motorfeedback`
    * Subscribed topics : `/motordrive_cmd`
- `pid_coeff_publisher` (with dynamic reconfigure)
    * Published topics : `/pid_coeff_cmd`

**Launch files:**
- `base.launch`
- `vision.launch`




The hardware driver communicating with Nucleo card is `FuturakartHardware` interface from `futurakart_hardware.h / futurakart_hardware.cpp` following Robot_HW paradigm of `ros_control` framework. 
It contains :
- Propulsion Position / Velocity Joint Interfaces
- Direction Position Joint Interface


```
roslaunch futurakart_base vision.launch
```
which initializes the Kinect sensor.


### Desktop packages

`futurakart_viz` package helps to display the robot using RViz: 
 
In simulation mode:
```
$ roslaunch futurakart_gazebo futurakart_world.launch gui:=false
$ roslaunch futurakart_viz view_robot.launch 
``` 

You can also use available configurations `navigation`, `localization`:
```
$ roslaunch futurakart_viz view_robot.launch config:=<config>
```
 
 

#### Simulator 

The package `futurakart_gazebo` uses some of core packages and allows to run a gazebo simulation:
```
roslaunch futurakart_gazebo futurakart_world.launch
```


## Useful tools 

#### setup_all.bash

A script that sets aliases :
 
- `fkart` to perform `cd ~/futurakart_ws/; source devel/setup.bash`
- `save_gmap` to save map produced by gmapping (see `futurakart_2dnav` package)
- `src` to source from current folder : `source devel/setup.bash`
- Source from generic_ws path : add in `.bashrc` a line like `source ~/generic_ws/devel/setup.bash`

Also the script sets the variables ROS_MASTER_URI and ROS_HOSTNAME defined in `ros.conf`. 
Before the usage copy and rename ros.conf.example to ros.conf and setup your values.    
  
Usage :
```
$ cd /path/to/repo/futurakart_ws/; source setup_all.bash [--local]
```

The option `--local` sets ROS_MASTER_URI and ROS_HOSTNAME as `localhost` to work standalone.



## Troubleshooting

### Custom script does not start
- If you want to start a python script from a file .launch as `<node name="myname" pkg="mypkg" type"myscript.py"/>`, do not forget `chmod +x myscript.py`  

### Cannot find `rosserial` package

Make sure that you have `source`d the rosserial workspace. Check the ROS_PACKAGE_PATH and verify if the rosserial workspace path is present:
```
echo $ROS_PACKAGE_PATH
```
If it is not present, then execute:
```
source /path/to/rosserial_ws/devel/setup.bash
```

### Cannot find `ackermann_controller/AckermannController`

Probably, when you cloned `futurakart` source the flag `--recursive` was not present.
Do this to fix :
```
git submodule update --init --recursive
```
Then run `catkin_make` in `futurakart` workspace.

### RPi fails to compile a module

Probably, it is a memory problem. There are several ways to compile the module:

- Use only one thread : `catkin_make -j1`. If does not work
- Use `chroot` on your desktop on the RPi image. Google for more info or use [this](https://github.com/vfdev-5/qemu-rpi2-vexpress)
- Use a qemu emulator on RPi. Google for more info or use [this](https://github.com/vfdev-5/qemu-rpi2-vexpress)


### Network setup problem 

The master machine in our Ros network is RPi, thus : 
- ROS_MASTER_URI should be something like `http://<hostname>:11311` where `<hostname>` can be found running in terminal : `$ hostname`
- ROS_HOSTNAME should be `<hostname>`

Any desktop or slave machine should be configured as (let hostname of RPi master machine be `ubuntu-rpi`) :
- Edit /etc/hosts to add hostname resolution : `XX.XX.XX.XX    ubuntu-rpi`
- ROS_MASTER_URI should be something like `http://ubuntu-rpi:11311`
- ROS_HOSTNAME should be ip address in the network

