# AGIBOT Genie 02 GDK - Claude Code Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that provides the complete AGIBOT Genie 02 GDK v2.3.4 API reference. When installed, Claude Code can write robot control code with full knowledge of the Python, C++, and ROS2 interfaces.

## What it does

This skill gives Claude Code access to the full GDK API surface, so it can:

- Write Python or C++ code that controls the Genie 02 robot
- Use correct method signatures, enum values, joint names, and limits
- Set up ROS2 forwarding nodes and DDS topic subscriptions
- Configure motion planning, impedance control, and trajectory tracking
- Access cameras, IMUs, LiDAR, SLAM, navigation, and TF transforms

## API coverage

| Module              | Description                                                              |
| ------------------- | ------------------------------------------------------------------------ |
| **Robot**           | Joint states, arm/head/waist control, motion planning, impedance control |
| **Camera**          | 18 camera types, intrinsics, image capture                               |
| **IMU**             | 3 IMU sensors, angular velocity, linear acceleration                     |
| **Lidar**           | Front/back LiDAR, point cloud retrieval                                  |
| **Slam**            | Mapping, localization, odometry                                          |
| **Pnc**             | Navigation, chassis control, task management                             |
| **Map**             | Map CRUD, switching                                                      |
| **TF**              | Transform tree lookups, sensor extrinsics                                |
| **Bundle**          | Bundled tensor data streaming                                            |
| **Interaction**     | TTS, audio/video playback, ASR                                           |
| **ExperimentalAPI** | Object detection publishing, skill/taskflow integration                  |
| **DDS**             | Low-level pub/sub, service clients                                       |

## Installation

Clone this repo:

```bash
git clone <repo-url>
```

Copy the repo into your project root or user `.claude/`

```bash
# From your project root or your user folder
mkdir -p .claude
cp -r agibot-G2-gdk .claude/
```

The skill is automatically available once the directory exists — no additional configuration needed.

## Usage

Claude Code loads this skill automatically when you ask it to write code for the Genie 02 robot. Examples:

```
> Write a Python script that reads the head camera and saves a JPEG

> Move both arms to a home position using motion planning with collision checking

> Set up impedance control on the left arm with custom stiffness parameters

> Subscribe to the front LiDAR point cloud over ROS2
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- For running generated code: AGIBOT GDK v2.3.4+ installed on the robot or dev machine

## License

This skill packages publicly available API documentation from AGIBOT for use with Claude Code. The GDK itself is provided by AGIBOT under their own license terms.
