This repo contains Dockerfiles for the [Segway RMPLite 220](https://robotics.segway.com/wp-content/uploads/2021/04/Segway-RMP-220Lite-Specs.pdf).

See our internal wiki for instructions how to connect to and use the robot in the first place:
https://cloud.cps.unileoben.ac.at/index.php/apps/collectives/CPS%20Robots/Robot%20Platforms/CPS%20Babsi%20-%20Segway%20RMP%20lite%20220

# Build

Download the git repo and build the Docker images:
```sh
docker compose build
```

# Usage

Two main modes are of interest:
1. teleoperation via joystick for data collection
2. autonomous navigation via Nav2

Teleoperation with Orbbec Femto:
```sh
docker compose up -d teleop orbbec
```

Then see our internal wiki for instructions how to command the robot via the joystick.

Once the robot is running, you can open a shell connected to the ROS network and log RGB-D data:
```sh
docker compose run shell
```
The Docker containers have the root folder of the repo mounted as a volume `/repo` inside the Docker container. You can execute the `rgbd_log.sh` script, or any other script of your choice from there:
```sh
/repo/rgbd_log.sh /repo/bags/$MY_BAG_FILE
```

You can also combine these commands and run arbitrary scripts or launch files mounted from outside:
```sh
# run a custom launch file
docker compose run --rm -d shell ros2 launch /repo/femto_NFOV_binned.launch.py
# run a custom script
docker compose run --rm shell /repo/rgbd_log.sh /repo/bags/$MY_BAG_FILE
```

Navigation:
```sh
docker compose up -d navigation
```

# Remote Connection

The Docker containers are using ROS 2 jazzy with the Zenoh RMW and run a dedicated Zenoh router on the roobot. To connect to the robot from inside the network, install the Zenoh RMW:
```sh
sudo apt install ros-jazzy-rmw-zenoh-cpp
```
and set the following environment variables:
```sh
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
export ZENOH_CONFIG_OVERRIDE='mode="client";connect/endpoints=["tcp/cps-seggy.local:7447"]'
```
where `cps-seggy.local` is the hostname of the robot running the Zenoh router on port `7447`.

Note that for this to work, any of the Docker images has to run as those will start the Zenoh router as dependency. To use your local Zenoh router again, undo this changes, e.g. via `unset ZENOH_CONFIG_OVERRIDE`.
