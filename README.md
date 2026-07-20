# EtherCAT V2 Workspace

A ROS2 Humble workspace for testing real-time EtherCAT communication on a quadcopter drone platform, built on Ubuntu 22.04 with a PREEMPT_RT kernel.

## Overview

This workspace integrates the [EcatV2_Master](https://github.com/AIMEtherCAT/EcatV2_Master) EtherCAT stack with ROS2 via the `soem_bringup` package. It enables low-latency communication between the flight controller and EtherCAT slave devices — including IMU sensors and DShot-driven motors — over a dedicated real-time Ethernet interface.

## Hardware

- **Platform**: Quadcopter UAV
- **Flight Computer**: x86_64 SBC running Ubuntu 22.04
- **EtherCAT Interface**: Dedicated USB-to-Ethernet adapter (`enx207bd2d1cb70`)
- **Slave Devices**:
  - IMU sensor (PDO read → `/imu` topic, `imu_link` frame)
  - DShot motor controller (PDO write ← `/motor` topic)

## Architecture

```
ecat_ws/
├── src/
│   ├── soem_bringup/          # ROS2 launch & config package
│   │   ├── launch/
│   │   │   └── bringup.launch.py
│   │   ├── config/
│   │   │   └── config.yaml    # Slave & task definitions
│   │   ├── CMakeLists.txt
│   │   └── package.xml
│   └── EcatV2_Master/         # EtherCAT master library (submodule)
│       └── src/soem_wrapper/  # SOEM wrapper with DShot & IMU support
├── .gitmodules
└── README.md
```

## Prerequisites

- **OS**: Ubuntu 22.04 LTS
- **Kernel**: `linux-realtime` (PREEMPT_RT) for deterministic timing
- **ROS2**: Humble Hawksbill
- **Build tool**: `colcon`

### System Setup

```bash
# Install real-time kernel
sudo apt install linux-image-lowlatency-hwe-22.04

# Install ROS2 Humble (see https://docs.ros.org/en/humble/Installation.html)

# Grant real-time privileges
sudo usermod -aG rtprio $USER
```

## Build

```bash
cd ~/ecat_ws

# Install dependencies
rosdep install --from-paths src --ignore-src -r -y

# Build with colcon
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Source the workspace
source install/setup.bash
```

## Usage

```bash
# Launch the EtherCAT backend
ros2 launch soem_bringup bringup.launch.py
```

This starts the `soem_backend` node which:
1. Initializes the EtherCAT master on interface `enx207bd2d1cb70`
2. Pins the real-time thread to CPU core 0
3. Reads IMU data via PDO and publishes to `/imu`
4. Subscribes to `/motor` commands and writes to the DShot motor controller via PDO

## Configuration

Edit `src/soem_bringup/config/config.yaml` to adjust:

| Parameter | Description |
|-----------|-------------|
| `interface` | Ethernet interface name for EtherCAT |
| `rt_cpu` | CPU core for the real-time thread |
| `non_rt_cpus` | CPU cores for non-real-time tasks |
| `slaves` | EtherCAT slave device definitions (SDO length, tasks, PDO mapping) |

## Topics

| Topic | Direction | Type | Description |
|-------|-----------|------|-------------|
| `/imu` | Publisher | Custom | IMU sensor data (accel, gyro) |
| `/motor` | Subscriber | Custom | Motor control commands (DShot) |
| `/ecat/sn2555957/latency` | Publisher | Custom | Slave latency monitoring |

## License

MIT
