# Docker with ROS 2, Gazebo and ArduPilot

This repo was based on [ryomo's repo](https://github.com/ryomo/ros2-gazebo-docker/tree/main). To include ArduPilot following ArduPilot Installation Guides: [ROS 2](https://ardupilot.org/dev/docs/ros2.html#ros2) and [Gazebo](https://ardupilot.org/dev/docs/ros2-gazebo.html#ros2-gazebo).

# Description 
- A Docker Container that has ROS 2, Gazebo and ArduPilot.
- GUI is enabled if WSLg is installed.
- Open ports for UDP that is relavent to MissionPlanner/QGroundControl (14550), GStreamer from Gazebo (5600) and X-Plane (49001)
- A Network called "shared_network_for_robotics" was made to communicate with another docker container within the same network and can be commented out if you do not require it.

## Note: 
- If you want to modify the files that were installed via the dockerfiles then modify the ones in ```dev_ws/install```, not in ```dev_ws/src```.
- change ```..``` on line 48 of ```..:/home/dockeruser/project```within ```{name-of-docker-compose-file}.yml``` to a location of your chosing, currently it is linking to the folder before {this_repository}. This allows for a link between your folder within WSL to the docker container folder located in ```/home/dockeruser/project``` for you to have access to it while not being inside the docker container.

# Versions
ROS 2: Humble
Gazebo: Harmonic

# Prerequisites
- docker, docker-compose
- (Optional) WSLg
- VS Code (Optional) **Recommended**

# Building and Creating the Docker Container 
```
git clone {this-repository}
cd {this-repository}
docker compose -f {name-of-docker-compose-file}.yml build
docker compose -f {name-of-docker-compose-file}.yml up -d
```
# Getting Started
To enter the docker container via command-line interface (CLI) otherwise I advise to use VS Code and attach "Attach VS Code..."
```
cd {this-repository}
docker compose -f {name-of-docker-compose-file}.yml exec ros2 bash
```

## ROS 2
From [Using turtlesim, ros2 and rqt](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Introducing-Turtlesim/Introducing-Turtlesim.html)
```
ros2 run turtlesim turtlesim_node
```

## ROS 2 with Software in the Loop (SITL) [ArduPilot]
From [ROS 2 with SITL](https://ardupilot.org/dev/docs/ros2-sitl.html)

Copy the entire code below and run it in 1 Terminal.
```
ros2 launch ardupilot_sitl sitl_dds_udp.launch.py \
transport:=udp4 \
synthetic_clock:=True \
wipe:=False \
model:=quad \
speedup:=1 \
slave:=0 \
instance:=0 \
defaults:=$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/copter.parm,$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/dds_udp.parm \
sim_address:=127.0.0.1 \
master:=tcp:127.0.0.1:5760 \
sitl:=127.0.0.1:5501
```

Open another Terminal do the following commands to check if there is an interaction between ROS 2 with ArduPilot
This will show the node in the ROS graph
```
ros2 node list
```
This will show all the topics within that node
```
ros2 node info /ap
```
This will provide information that is being published by a topic which in this case is the geo position
```
ros2 topic echo /ap/geopose/filtered
```
## Gazebo with SITL
From [Using SITL with Gazebo](https://ardupilot.org/dev/docs/sitl-with-gazebo.html)
In 1 Terminal run Gazebo example:
```
gz sim -v4 -r iris_runway.sdf
```
In another terminal start SITL:
```
sim_vehicle.py -v ArduCopter -f gazebo-iris --model JSON --map --console
```

## Gazebo with ROS 2 and ArduPilot
From [ROS 2 with Gazebo](https://ardupilot.org/dev/docs/ros2-gazebo.html#ros2-gazebo)
```
ros2 launch ardupilot_gz_bringup iris_runway.launch.py
```

# Useful Commands for docker-compose via CLI
For stopping the container
```
docker compose stop
```
For removing the container
```
docker compose down
```
For finding docker's IP 
```
docker exec -it {created_container_name} cat /etc/hosts | grep host.docker.internal
```
For finding IPs associated with the container
```
docker exec -it {created_container_name} cat /etc/hosts
```
For all created container
```
docker ps -a
```
For creating a network for your docker containers sharing the same network name
```
docker network create {name_of_network}
```
