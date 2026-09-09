# Repository Overview

This repository presents the training and deployment projects for the M3 robot.

Document version: V1.0.1.

> 🚧 This repository is still under construction, and its contents are not yet complete. For the full version history, see [CHANGELOG.md](./CHANGELOG.md).

# Repository Structure

```text
mini3/
├── README.md                 # Overview, features, and quick start
├── LICENSE
├── THIRD_PARTY_NOTICES.md    # Third-party sources and license notices
├── CONTRIBUTING.md
├── CHANGELOG.md
├── .gitignore
├── .github/workflows/        
│
├── docs/                   
│   ├── architecture.md       # Software and algorithm architecture
│   ├── model_contract.md     # Model inputs/outputs and joint order
│   ├── train/                # Training tutorials
│   ├── deploy/               # Simulation, physical robot, and teleoperation tutorials
│   └── media/                # Images and demonstration assets
│
├── train/                   
│   ├── robolab/
│   └── rsl_rl/
│
├── deploy/                  
│   ├── src/
│   ├── robot_assets/
│   ├── pre_train/
│   ├── pico_sim2sim/
│   ├── skeleton_retarget/
│   ├── scripts/
│   ├── tools/
│   └── assets/
│
└── release/                  # Model asset manifest and version compatibility mapping
    ├── assets_manifest.yaml
    └── compatibility.md
```

The table below describes the top-level directories of the repository, with paths relative to `mini3/`. This is currently a working draft: some planned directories have not yet been created, and the public rele...

> Content cannot be displayed outside Feishu Docs at this time.

# Quick Start

## Prerequisites

MINI3-Train is built on the latest versions of Isaac Sim and Isaac Lab. We recommend following the latest updates to MINI3-Train.

- Install Isaac Lab according to the installation guide. We recommend using conda, as it simplifies running Python scripts from the terminal.
- Clone this repository separately from Isaac Lab (that is, outside the `IsaacLab` directory):

```bash
git clone https://github.com/humanoidailab/mini3_lab.git
```

- Install the packages using the Python interpreter provided by the installed Isaac Lab:

```bash
cd mini3_lab
git submodule update --init --recursive
cd robolab
pip install -e .
cd ..
cd rsl_rl
pip install -e .
cd ..
```

- Verify that the extensions are installed correctly and print the list of available environments:

```bash
python robolab/scripts/tools/list_envs.py
```

The deployment code runs on an Orange Pi 5 Plus with Ubuntu 22.04 and kernel version 5.10. The following describes the deployment environment configuration for the Orange Pi 5 Plus.

First, install ROS 2 Humble. Refer to the official ROS documentation for installation instructions.

Deployment also depends on libraries including `ccache`, `fmt`, `spdlog`, and `eigen3`. Run the following command on the host computer to install them:

```bash
sudo apt update && sudo apt install -y ccache libfmt-dev libspdlog-dev libeigen3-dev
```

Then install the Orange Pi 5 Plus 5.10 real-time kernel:

```bash
git clone https://github.com/Roboparty/atom01_deploy.git
cd atom01_deploy/assets
sudo apt install ./*.deb
cd ..
```

Next, grant the user permission to configure real-time priorities:

```bash
sudo nano /etc/security/limits.conf
```

Add the following two lines to the end of the file, replacing `orangepi` with your actual username:

```text
# Allow user 'orangepi' to set real-time priorities
root   -   rtprio   98
root   -   memlock  unlimited
```

Save the file, exit, and reboot the Orange Pi for the changes to take effect.

# Training

Most scripts use `--task` to select an environment. You can first run `_enpython robolab/scripts/tools/listvs.py` to view all currently registered task IDs.

Common Mini3 task examples:

> Content cannot be displayed outside Feishu Docs at this time.

For debugging, you can run a smoke test with a smaller number of environments:

```bash
python robolab/scripts/rsl_rl/train.py \
  --task Mini3-Walk \
  --logger tensorboard \
  --num_envs 9 \
  --max_iterations 500 \
  --run_name Test
```

```bash
python robolab/scripts/rsl_rl/train.py \
  --task Mini3-BeyondMimic \
  --logger tensorboard \
  --num_envs 9 \
  --max_iterations 500 \
  --run_name Mini3_BM_Test \
```

For full training, playback, export, and motion imitation procedures, see the training documentation.

# Dataset Preparation

For the motion data used by BeyondMimic, refer to GMR.

The joint order in the GMR data is consistent with the robot URDF/XML, but differs from the joint order used during Isaac Lab training. First, prepare a mapping file similar to `robolab/scripts/tools/retarget/config/mini3.yaml` to retarget `.pkl` files to the Isaac Lab joint order. BeyondMimic training additionally requires conversion to `.npz`.

```bash
python robolab/scripts/tools/retarget/dataset_retarget.py \
  --robot mini3 \
  --config_file robolab/scripts/tools/retarget/config/mini3.yaml \
  --input_dir robolab/data/motions/mini3_gmr \
  --output_dir robolab/data/motions/mini3_lab
```

```bash
python3 robolab/scripts/tools/retarget/prepend_default_transition.py \
  --input_file robolab/data/motions/mini3_gmr/bend_hip.pkl \
  --output_file robolab/data/motions/mini3_gmr/mini3_bend_hip_gmr_default_transition.pkl \
  --transition_frames 60 \
  --placement both
```

```bash
python robolab/scripts/tools/beyondmimic/pkl_to_npz.py \
  -f robolab/data/motions/mini3_lab/joyin_mini3_mengguwu.pkl \
  --output_name robolab/data/motions/mini3_dance_lab/joyin_mini3_mengguwu.npz \
  --output_fps 60 \
  --robot mini3 \
  --config_file robolab/scripts/tools/retarget/config/mini3.yaml
```

```bash
python robolab/scripts/tools/beyondmimic/pkl_to_npz.py \
  --input_dir robolab/data/motions/mini3_lab \
  --output_dir robolab/data/motions/mini3_dance_lab \
  --output_fps 60 \
  --robot mini3 \
  --config_file robolab/scripts/tools/retarget/config/mini3.yaml
```

# Simulation Deployment

## Simulation Results

# Physical Robot Deployment

## Using the Physical Robot

After the program starts, the robot enters standby mode. Use the controller to perform the following steps in order: **enable motors → reset → start walking or motion imitation**. Then control the robot according to the selected mode.

For specific button mappings, refer to `src/inference/config/system.yaml`. To exit the program, run:

```bash
./tools/stop_robot.sh
```

For detailed procedures, see the physical robot deployment and physical robot usage documentation.

# Software System Architecture Diagram

[Image]

# Motion Control Framework

The motion control framework can be considered a detailed expansion of the green module. At this point, some upper-level and lower-level components can be described more briefly.

# Python SDK

This repository provides a Python SDK to help users control the hardware with Python scripts. Before use, make sure that the ROS 2 environment and `install/setup.bash` in this workspace have been sourced.

## 1. IMU SDK (`imu_py`)

### Static Method

- `create_imu(imu_id: int, interface_type: str, interface: str, imu_type: str, baudrate: int = 0) -> IMUDriver`: Creates an IMU driver instance.

### Member Methods

- `get_imu_id() -> int`: Returns the IMU ID.
- `get_ang_vel() -> List[float]`: Returns angular velocity `[x, y, z]`.
- `get_quat() -> List[float]`: Returns the quaternion `[w, x, y, z]`.
- `get_lin_acc() -> List[float]`: Returns linear acceleration `[x, y, z]`.
- `get_temperature() -> float`: Returns the temperature.

### Example

```python
import imu_py
imu = imu_py.IMUDriver.create_imu(8, "serial", "/dev/ttyUSB0", "HIPNUC", 921600)
quat = imu.get_quat()
```

## 2. Motor SDK

Provides the `MotorControlMode` enumeration: `NONE`, `MIT`, `POS`, and `SPD`.

### Static Method

- `create_motor(motor_id: int, interface_type: str, interface: str, motor_type: str, motor_model: int, master_id_offset: int = 0) -> MotorDriver`: Creates a motor driver instance.

### Member Methods

- `init_motor()`: Initializes the motor.
- `deinit_motor()`: Deinitializes the motor.
- `set_motor_control_mode(mode: MotorControlMode)`: Sets the control mode.
- `motor_mit_cmd(pos: float, vel: float, kp: float, kd: float, torque: float)`: Sends a control command in MIT mode.
- `motor_pos_cmd(pos: float, spd: float, ignore_limit: bool = False)`: Sends a control command in position mode.
- `motor_spd_cmd(spd: float)`: Sends a control command in speed mode.
- `lock_motor() / unlock_motor()`: Locks or unlocks the motor.
- `set_motor_zero()`: Sets the current position as the zero position.
- `clear_motor_error()`: Clears errors.
- `get_motor_pos() -> float`: Returns the position in radians.
- `get_motor_spd() -> float`: Returns the speed in radians per second.
- `get_motor_current() -> float`: Returns the current in amperes.
- `get_motor_temperature() -> float`: Returns the temperature in degrees Celsius.
- `get_error_id() -> int`: Returns the error code.
- `write_motor_flash()`: Writes the current parameters to Flash.
- `reset_motor_id(new_id: int)`: Resets the motor ID.

### Example

```python
import motors_py
motor = motors_py.MotorDriver.create_motor(1, "can", "can0", "DM", 0, 16)
motor.init_motor()
motor.set_motor_control_mode(motors_py.MotorControlMode.MIT)
motor.motor_mit_cmd(0.0, 0.0, 5.0, 1.0, 0.0)
```

## 3. Robot SDK

The `RobotInterface` class provides unified control of the entire robot. It automatically loads the motors and IMU from a configuration file.

### Constructor

- `RobotInterface(config_file: str)`: Creates an instance based on the configuration file path.

### Member Methods

- `init_motors()`: Initializes all motors.
- `deinit_motors()`: Deinitializes all motors.
- `reset_joints(joint_default_angle: List[float])`: Resets all joints to their default angles.
- `apply_action(action: List[float])`: Applies a control action (joint target positions, torques, etc., depending on the internal implementation).
- `refresh_joints()`: Refreshes the state of all joints.
- `set_zeros()`: Sets the current positions of all joints as zero positions.
- `clear_errors()`: Clears all motor errors.
- `get_joint_q() -> List[float]`: Returns all joint positions.
- `get_joint_vel() -> List[float]`: Returns all joint velocities.
- `get_joint_tau() -> List[float]`: Returns all joint torques.
- `get_quat() -> List[float]`: Returns the IMU quaternion `[w, x, y, z]`.
- `get_ang_vel() -> List[float]`: Returns the IMU angular velocity.

### Property

- `is_init`: (Read-only) Indicates whether the robot has been initialized.

### Example

```python
import robot_py
robot = robot_py.RobotInterface("config/robot.yaml")
robot.init_motors()
robot.apply_action([0.0] * 23)
```

> **Note:** For detailed Python script examples, refer to the `scripts/` directory.

# FAQ

# Release Notes

V1.0.1: 2026.9.9

# References and Acknowledgements


# Content to Be Added

- Simulation demos
- Out-of-the-box code
- FAQ
- Debugging guides
- Common errors

# License

[GNU General Public License v3.0](LICENSE).
