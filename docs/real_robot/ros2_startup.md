# Bringing Up a Real Dobot Robot from the Command Line (Linux)

This page documents the **exact procedure** for bringing up a real Dobot robot
from the command line on a Linux machine, without Dobot Studio, Gazebo, or MoveIt.

It is based on **empirical testing** with the Dobot ROS 2 V4 driver and reflects
real controller behaviour.

---

## Prerequisites

- Ubuntu 22.04
- ROS 2 Humble
- Wired Ethernet connection to the robot
- Dobot robot in **TCP/IP Secondary Development** mode
- No Dobot Studio running

---

## 1. Power on the robot

1. Power on the Dobot controller
2. Release E-stop (if present)
3. Ensure the robot is **not moving**
4. Do **not** open Dobot Studio

---

## 2. Network setup

The Dobot controller uses a fixed IP.

Typical defaults:

| Connection type | Robot IP |
|-----------------|----------|
| Wired Ethernet  | `192.168.5.1` |
| Wi-Fi           | `192.168.1.6` |

Set your computer to the same subnet if required.

---

## 3. Configure environment variables

Edit your `~/.bashrc`:

```bash
export IP_address=192.168.5.1
export DOBOT_TYPE=nova2   # example: nova2, cr5, cr7, etc
```

Reload:

```bash
source ~/.bashrc
```

> ⚠️ Selecting the wrong `DOBOT_TYPE` will cause silent failures.

---

## 4. Download and build the Dobot ROS 2 repository

```bash
mkdir -p ~/dobot_ws/src
cd ~/dobot_ws/src
git clone https://github.com/Dobot-Arm/DOBOT_6Axis_ROS2_V4.git
cd ..
colcon build
source install/setup.bash
```

(Optional, but recommended)

```bash
echo "source ~/dobot_ws/install/setup.bash" >> ~/.bashrc
```

---

## 5. Start the real robot controller (NOT Gazebo)

In **Terminal 1**:

```bash
source ~/dobot_ws/install/setup.bash
ros2 launch dobot_bringup_ros2 dobot_bringup_ros2.launch.py
```

You should see logs indicating:
- robot IP
- robot type
- TCP connection established

---

## 6. Verify network connectivity

In **Terminal 2**:

```bash
ping 192.168.5.1
```

Expected:
- low latency (<1 ms)
- no packet loss

If ping is unstable, **do not proceed**.

---

## 7. Initial robot state check

In **Terminal 2**, check the robot mode:

```bash
ros2 service call   /dobot_bringup_ros2/srv/RobotMode   dobot_msgs_v4/srv/RobotMode   "{}"
```

### RobotMode values (observed)

| Value | Meaning |
|------|--------|
| `{1}` | Initialising |
| `{3}` | Error / alarm state |
| `{4}` | Disabled / idle |
| `{5}` | Enabled / operational |

At this stage, the robot typically reports:

```
{4}
```

---

## 8. Clear alarms and errors

Even if no error is visible, this step is required.

```bash
ros2 service call   /dobot_bringup_ros2/srv/ClearError   dobot_msgs_v4/srv/ClearError   "{}"
```

Expected:
```
res = 0
```

---

## 9. Set payload mass and centre of gravity

The controller requires a valid payload before enabling.

Example: very light payload at TCP origin.

```bash
ros2 service call   /dobot_bringup_ros2/srv/SetPayload   dobot_msgs_v4/srv/SetPayload   "{load: 0.01, x: 0.0, y: 0.0, z: 0.0}"
```

> ⚠️ Some firmware versions return `res = -10000` but still accept the payload.
> This is a known Dobot quirk.

---

## 10. Enable the robot

```bash
ros2 service call   /dobot_bringup_ros2/srv/EnableRobot   dobot_msgs_v4/srv/EnableRobot   "{}"
```

Expected:
```
res = 0
```

---

## 11. Verify enabled state

Check robot mode again:

```bash
ros2 service call   /dobot_bringup_ros2/srv/RobotMode   dobot_msgs_v4/srv/RobotMode   "{}"
```

Expected:
```
{5}
```

This confirms:
- robot is enabled
- TCP control is active
- ready for motion commands

---

## Notes & Known Behaviour

- The TCP connection **must not be left idle**
- After long inactivity, the controller may stop responding
- A keepalive (e.g. periodic `RobotMode`) is recommended
- If TCP becomes unresponsive:
  - stop sending commands
  - kill the ROS node
  - restart bringup

---

## Next steps

- Seed kinematics using `GetAngle`
- Configure velocity and acceleration
- Issue motion commands
- Add a software keepalive

These steps are documented in the Motion section.
