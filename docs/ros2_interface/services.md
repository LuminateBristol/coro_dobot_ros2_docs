# Dobot ROS 2 Services Reference

This page documents the **ROS 2 services exposed by the Dobot ROS 2 V4 driver**
(`cr_robot_ros2`), as used for real robot control via the TCP/IP secondary
development interface.

This reference is **not MoveIt-specific** and applies to direct command-line
and programmatic control of Dobot robots (Nova and CR series).

---

## Namespace

All services are exposed under:

```
/dobot_bringup_ros2/srv/
```

---

## Dashboard & Lifecycle Services

These services control robot state, power, and recovery.

---

### EnableRobot

Enables the robot for motion.

**Service**
```
/EnableRobot
```

**Type**
```
dobot_msgs_v4/srv/EnableRobot
```

**Request**
```
(empty)
```

**Response**
```
int32 res
```

**Notes**
- Must be preceded by `ClearError`
- Does not move the robot
- Required before any motion command

---

### DisableRobot

Safely disables the robot.

**Service**
```
/DisableRobot
```

**Notes**
- Preferred over E-stop for software shutdown
- Releases servo power

---

### ClearError

Clears alarms and error states.

**Service**
```
/ClearError
```

**Notes**
- Required after E-stop
- Safe to call even if no visible error exists

---

### RobotMode

Returns the current robot mode.

**Service**
```
/RobotMode
```

**Response**
```
string robot_return
int32 res
```

**Observed values**

| Value | Meaning |
|------|--------|
| `{1}` | Initialising |
| `{3}` | Error / alarm |
| `{4}` | Disabled / idle |
| `{5}` | Enabled / operational |

---

### RequestControl

Requests TCP control ownership.

**Service**
```
/RequestControl
```

**Notes**
- Often unnecessary if using ROS bringup
- May return `-10000` on Nova
- Use with caution

---

## State & Feedback Services

---

### GetAngle (MANDATORY)

Seeds the kinematic model and returns joint angles.

**Service**
```
/GetAngle
```

**Notes**
- Must be called before any motion
- Required to avoid `-30001` motion rejection

---

### GetPose

Returns the current TCP pose.

**Service**
```
/GetPose
```

---

### GetErrorID

Returns active error codes.

**Service**
```
/GetErrorID
```

---

## Motion Parameter Services

These configure global motion limits.

---

### VelJ

Sets joint velocity.

**Service**
```
/VelJ
```

**Request**
```
int32 r   # 1–100 (%)
```

---

### AccJ

Sets joint acceleration.

**Service**
```
/AccJ
```

**Request**
```
int32 r   # 1–100 (%)
```

---

### VelL / AccL

Linear velocity and acceleration (Cartesian).

---

## Motion Command Services

---

### MovJ

Joint-space motion.

**Service**
```
/MovJ
```

**Request**
```
bool mode        # true = joint
float64 a–f      # joint values (rad)
string[] param_value
```

**Notes**
- Blocking TCP command
- Must not be retried on error
- Invalid state returns `-30001`

---

### MovL

Linear Cartesian motion.

---

### Stop / Pause / Continue

Motion control commands.

---

## Tool & Payload Services

---

### SetTool

Selects active tool.

---

### SetPayload

Sets payload mass and centre of gravity.

**Request**
```
float64 load
float64 x
float64 y
float64 z
```

**Notes**
- Required before enabling robot
- Some firmware returns `-10000` but still applies payload

---

### SetTCP

Sets TCP offset.

---

## I/O Services

---

### SetDO / GetDI

Controller digital I/O.

---

### SetToolDO / GetToolDI

Tool I/O.

---

## Force / Compliance Services

(Availability depends on robot and firmware)

- ForceMode
- SetForceLevel
- EndForce

---

## Important Behaviour Notes

- TCP connection must not be left idle
- Error responses may destabilise TCP
- Do not retry failed motion commands
- Restart bringup on timeout

---

## Related Topics

See:
- `FeedInfo` topic for continuous feedback
- `joint_states` for ROS-native joint data

---

## See Also

- Real Robot CLI Bringup Guide
- Control Model & Keepalive
- Failure Modes
