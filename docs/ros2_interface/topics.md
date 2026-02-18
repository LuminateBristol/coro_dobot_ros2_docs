# Dobot ROS 2 Topics Reference

This page documents the **ROS 2 topics published by the Dobot ROS 2 V4 driver**
(`cr_robot_ros2`) when connected to a real robot.

These topics provide **continuous feedback** and should be preferred over
polling services wherever possible.

---

## Namespace

Topics are typically published under:

```
/dobot_bringup_ros2/
```

Some standard ROS topics are also published globally.

---

## Primary Dobot Feedback Topic

### FeedInfo

**Topic**
```
/dobot_bringup_ros2/msg/FeedInfo
```

**Type**
```
std_msgs/String
```

**Description**

This is the **main feedback channel** from the Dobot controller.  
It publishes a JSON-encoded payload containing live robot state.

**Typical contents (observed)**

- Robot mode (enabled / disabled)
- Joint positions
- Joint velocities
- TCP Cartesian pose
- Force / torque estimates
- Safety and alarm state
- Power and servo status

**Notes**
- Published continuously while TCP is connected
- Preferred over repeated service calls
- Safe to use as a keepalive signal
- Payload format may change between firmware versions

---

## ROS-Native State Topics

These topics are published to integrate with the standard ROS ecosystem.

---

### joint_states

**Topic**
```
/joint_states
```

**Type**
```
sensor_msgs/JointState
```

**Description**

Standard ROS joint state message derived from Dobot feedback.

Contains:
- joint names
- joint positions (rad)
- joint velocities

**Notes**
- Required for RViz and MoveIt
- Read-only
- Does not accept commands

---

### tf

**Topic**
```
/tf
```

**Type**
```
tf2_msgs/TFMessage
```

**Description**

Dynamic transforms for the robot model.

**Notes**
- Published continuously
- Used for visualization and kinematics

---

### tf_static

**Topic**
```
/tf_static
```

**Type**
```
tf2_msgs/TFMessage
```

**Description**

Static transforms for the robot (base, tool frames).

---

## Optional / Environment-Dependent Topics

These topics may appear depending on launch configuration.

---

### /parameter_events

**Type**
```
rcl_interfaces/msg/ParameterEvent
```

**Description**
Standard ROS 2 parameter update events.

---

### /rosout

**Type**
```
rcl_interfaces/msg/Log
```

**Description**
ROS logging output.

---

## Topic Usage Guidelines

### Prefer topics over services

- Topics are non-blocking
- Topics do not trigger TCP command paths
- Topics are safe for monitoring and keepalive logic

### Do NOT publish to Dobot topics

All Dobot topics are **read-only**.  
Commands must be sent via services only.

---

## Recommended Usage Patterns

- Use `FeedInfo` to:
  - monitor robot mode
  - detect enable/disable
  - implement keepalive timers
- Use `joint_states` for:
  - visualization
  - motion planning
- Avoid polling `RobotMode` unless necessary

---

## Known Limitations

- `FeedInfo` is JSON-encoded (not strongly typed)
- Field names are firmware-dependent
- No official schema is published by Dobot

---

## See Also

- Dobot ROS 2 Services Reference
- Real Robot CLI Bringup Guide
- Control Model & Keepalive
