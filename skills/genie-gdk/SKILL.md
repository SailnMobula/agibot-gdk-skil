---
name: genie-gdk
description: AGIBOT Genie 02 GDK v2.3.4 API reference for Python, C++, and ROS2 interfaces. Use when writing code that controls the Genie 02 robot.
---

# AGIBOT Genie 02 GDK v2.3.4 - API Reference

Use this skill when writing code that interfaces with the AGIBOT Genie 02 robot using the GDK (GENIE Development Kit). This provides the complete API reference for Python, C++, and ROS2 interfaces.

Full documentation: `docs/genie-02-gdk.md`

## Setup

### Python
```bash
pip install protobuf==5.28.3
source ~/.cache/agibot/app/env.sh  # hybrid
source /home/agi/app/env.sh         # local (on robot)
```
```python
import agibot_gdk
```

### C++
```cpp
#include "gdk/gdk.h"
// Namespace: agibot::gdk
// All classes need 1-2s init: std::this_thread::sleep_for(std::chrono::seconds(2));
// Return type: GDKRes (kSuccess on success)
// Init/Release: GDKInit(), GDKRelease()
```

### ROS2
```bash
# Start forwarding nodes (GDK uses DDS internally, not ROS2 by default)
source ~/.cache/agibot/app/gdk/scripts/ros_env.sh <package>  # gdk_camera, gdk_controller, gdk_imu, gdk_lidar
ros2 launch <package> <package>.launch
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=1
```

### Network
- Robot IP: `10.42.1.101` (fixed)
- Developer IP range: `10.42.1.10` - `10.42.1.99`
- SSH: `ssh agi@10.42.1.101` (password: `1`)

---

## Python API

### Robot (`agibot_gdk.Robot`)
Init wait: ~1s. Replaces old `RobotBody` and `MotionControl` classes.

| Method | Parameters | Returns | Description |
|--------|-----------|---------|-------------|
| `get_joint_states()` | none | `JointStates` | Get all joint states (position, velocity, effort, motor) |
| `get_end_state()` | none | `DualEndState` (.left_end_state, .right_end_state) | Get end-effector states |
| `get_whole_body_status()` | none | `WholeBodyStatus` | Get full robot status (errors, power, end models) |
| `get_motion_control_status()` | none | `MotionControlStatus` | Get frame poses, velocities, wrenches, collisions |
| `get_chassis_power_state()` | none | `ChassisPowerState` | Battery, power board, charging info |
| `get_chest_power_state()` | none | `ChestPowerState` | Upper body power states |
| `move_ee_pos(joint_states)` | `JointStates` | `GDKRes` | Move end-effector (gripper control) |
| `move_head_joint(positions, velocities)` | `list[float]`, `list[float]` | `GDKRes` | Move head joints (3 joints: yaw/pitch/roll) |
| `move_waist_joint(positions, velocities)` | `list[float]`, `list[float]` | `GDKRes` | Move waist joints |
| `move_arm_joint(positions, velocities)` | `list[float]`, `list[float]` | `GDKRes` | Move arm joints |
| `joint_control(req)` | `JointControlReq` | `GDKRes` | Single-call PD joint trajectory control |
| `end_effector_pose_control(pose)` | `EndEffectorPose` | `GDKRes` | Servo-level end-effector pose control (no collision detection) |
| `traj_tracking_control(req)` | `A2DTrajectoryRequest` | `GDKRes` | A2D-style trajectory tracking |
| `motion_plan_request(req)` | `MotionPlanReq` | `GDKRes` | Motion planning with collision checking |
| `set_control_mode(mode)` | `MotionControlMode` | `GDKRes` | Set control mode (impedance, gravity comp, etc.) |
| `set_load(load_info)` | `LoadInfo` | `GDKRes` | Set payload parameters (mass, CoG, inertia) |
| `set_compliance_params(left, right, detail)` | `dict[str,float]`, `dict[str,float]`, `str` | `GDKRes` | Set arm compliance (stiffness, damping, clipping) |
| `force_position_control(targets)` | `dict[str, Pose]` | `GDKRes` | Force/position hybrid control |
| `set_reference_frame_poses(lx,ly,lz,lw, rx,ry,rz,rw)` | 8 floats (left/right quaternions) | `GDKRes` | Set reference frame orientations |

**JointStates attrs:** `.timestamp` (ns), `.nums` (int), `.states` (list of JointState)
**JointState attrs:** `.name`, `.mode`, `.position` (rad), `.velocity` (rad/s), `.effort` (Nm), `.motor_position`, `.motor_velocity`, `.motor_current`, `.error_code`

**JointControlReq:**
```python
req = agibot_gdk.JointControlReq()
req.life_time = 1.0          # seconds
req.joint_names = [...]       # list[str]
req.joint_positions = [...]   # list[float] radians
req.joint_velocities = [...]  # list[float] rad/s
req.detail = ""               # optional detail
```

### Camera (`agibot_gdk.Camera`)
Init wait: ~3s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_latest_image(type, timeout)` | `CameraType`, `float` ms | `Image` or `None` |
| `get_nearest_image(type, ts, timeout)` | `CameraType`, `int` ns, `float` ms | `Image` or `None` |
| `get_image_shape(type)` | `CameraType` | `(width, height)` tuple |
| `get_image_fps(type)` | `CameraType` | `float` FPS |
| `get_image_latency(type, window_s)` | `CameraType`, `float` s | `LatencyStats` |
| `get_camera_intrinsic(type)` | `CameraType` | `CameraIntrinsic` (.intrinsic [fx,fy,cx,cy], .distortion [k1..k6]) |
| `set_dev_camera_config(path)` | `str` config file path | `GDKRes` |
| `close_camera()` | none | `GDKRes` |

**CameraType enum:**
| Name | Value | Description |
|------|-------|-------------|
| `kCameraUnknown` | 0 | Unknown |
| `kHeadBackFisheye` | 1 | Head back fisheye |
| `kHeadLeftFisheye` | 2 | Head left fisheye |
| `kHeadRightFisheye` | 3 | Head right fisheye |
| `kHeadStereoLeft` | 4 | Head stereo left |
| `kHeadStereoRight` | 5 | Head stereo right |
| `kHandLeftColor` | 6 | Hand left color |
| `kHandRightColor` | 7 | Hand right color |
| `kHeadColor` | 8 | Head RGB |
| `kHeadDepth` | 9 | Head depth |
| `kHandLeftUpperColor` | 10 | Hand left upper color |
| `kHandRightUpperColor` | 11 | Hand right upper color |
| `kHandLeftLowerColor` | 12 | Hand left lower color |
| `kHandRightLowerColor` | 13 | Hand right lower color |
| `kHandLeftUpperDepth` | 14 | Hand left upper depth |
| `kHandRightUpperDepth` | 15 | Hand right upper depth |
| `kHandLeftLowerDepth` | 16 | Hand left lower depth |
| `kHandRightLowerDepth` | 17 | Hand right lower depth |

**Image attrs:** `.timestamp_ns`, `.width`, `.height`, `.encoding` (UNCOMPRESSED/JPEG/PNG), `.color_format` (RGB/BGR/RGBA/BGRA/YUV420/YUV422/YUV444/NV12/NV21/GRAY8/GRAY16/BAYER_*/RS2_FORMAT_Z16), `.bit_depth` (8/16/32), `.data_view` (use `.data` in Python)

### Imu (`agibot_gdk.Imu`)
Init wait: ~2s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_latest_imu(type, timeout)` | `ImuType`, `float` ms | `ImuData` or `None` |
| `get_nearest_imu(type, timestamp, timeout)` | `ImuType`, `int` ns, `float` ms | `ImuData` or `None` |
| `get_imu_fps(type)` | `ImuType` | `float` FPS |
| `get_imu_latency(type, window_s)` | `ImuType`, `float` s | `LatencyStats` |
| `close_imu()` | none | `GDKRes` |

**ImuType:** `kImuUnknown=0`, `kImuFront=1`, `kImuBack=2`, `kImuChassis=3`

**ImuData attrs:** `.timestamp_ns`, `.angular_velocity` (.x,.y,.z rad/s), `.linear_acceleration` (.x,.y,.z m/s²)

### Lidar (`agibot_gdk.Lidar`)
Init wait: ~1s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_latest_pointcloud(type, timeout)` | `LidarType`, `float` ms | `PointCloud` or `None` |
| `get_nearest_pointcloud(type, timestamp, timeout)` | `LidarType`, `int` ns, `float` ms | `PointCloud` or `None` |
| `get_lidar_fps(type)` | `LidarType` | `float` FPS |
| `get_lidar_latency(type, window_s)` | `LidarType`, `float` s | `LatencyStats` |
| `close_lidar()` | none | `GDKRes` |

**LidarType:** `kLidarUnknown=0`, `kLidarFront=1`, `kLidarBack=2`

**PointCloud attrs:** `.timestamp_ns`, `.width`, `.height`, `.point_step`, `.row_step`, `.is_bigendian`, `.is_dense`, `.fields` (list of .name/.offset/.datatype/.count), `.data_view`

### UltrasonicRadar (`agibot_gdk.UltrasonicRadar`)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_latest_ultrasonic_radar()` | none | `UltrasonicRadars` |
| `get_nearest_ultrasonic_radar(timestamp)` | `int` ns | `UltrasonicRadars` |
| `get_ultrasonic_radar_fps()` | none | `float` FPS |
| `get_ultrasonic_radar_latency(window_s)` | `float` s | `LatencyStats` |
| `close()` | none | `GDKRes` |

**UltrasonicRadars attrs:** `.timestamp_ns`, `.ultrasonic_radar_datas` (list of `.id`, `.distance_mm`, `.fault_state`)

### Slam (`agibot_gdk.Slam`)
Init wait: ~1s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `start_mapping()` | none | `GDKRes` |
| `stop_mapping()` | none | `GDKRes` |
| `cancel_mapping()` | none | `GDKRes` |
| `record_spec_loc()` | none | `GDKRes` |
| `get_slam_state()` | none | `uint32_t` state |
| `get_odom_info()` | none | `OdomInfo` |
| `get_curr_pose()` | none | `Pose` |

### Pnc (`agibot_gdk.Pnc`)
Init wait: ~1s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `normal_navi(req)` | `NaviReq` | `GDKRes` |
| `high_precision_navi(req)` | `NaviReq` | `GDKRes` |
| `relative_move(req)` | `NaviReq` | `GDKRes` |
| `cancel_task(task_id)` | `uint32_t` | `GDKRes` |
| `pause_task(task_id)` | `uint32_t` | `GDKRes` |
| `resume_task(task_id)` | `uint32_t` | `GDKRes` |
| `get_task_state()` | none | `PNCTaskState` (.id, .state, .type, .message) |
| `move_chassis(twist)` | `Twist` | `GDKRes` |
| `request_chassis_control(mode)` | `int32_t` | `GDKRes` |

**PNCTaskState.state:** 0=idle, 1=starting, 2=running, 3=pausing, 4=paused, 5=resuming, 6=canceling, 7=canceled, 8=fail, 9=success

### Map (`agibot_gdk.Map`)
Init wait: ~1s

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_map(map_id)` | `int` | `MapInfo` |
| `get_curr_map()` | none | `MapName` (.id, .name, .is_curr_map) |
| `get_all_map()` | none | `list[MapName]` |
| `switch_map(map_id)` | `int` | `GDKRes` |
| `remove_map(map_id)` | `int` | `GDKRes` (irreversible!) |

### TF (`agibot_gdk.TF`)
Transform tree lookups. Init wait: ~1s.

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_all_tf_from_base_link()` | none | `list[TransformStamped]` |
| `get_tf_from_base_link(child_frame_id)` | `str` | `Transform` |
| `get_tf_from_sensor(sensor_type)` | `SensorExtrinsicType` | `Transform` |
| `lookup_transform_latest(target, source)` | `str`, `str` | `Transform` (+ optional timestamp_ns) |
| `lookup_transform(target, source, time_ns)` | `str`, `str`, `int` ns | `Transform` |
| `can_transform(target, source)` | `str`, `str` | `bool` |
| `get_all_frame_names()` | none | `list[str]` |
| `get_latest_timestamp(frame_id)` | `str` | `uint64_t` ns |
| `clear()` | none | none |

**SensorExtrinsicType enum:** `kHeadLeftStereoToHeadRightStereo`, `kLeftHandDepthToLeftHandColor`, `kRightHandDepthToRightHandColor`, `kHeadDepthToHeadColor`, `kHeadLeftStereoToHeadLink3`, `kHeadRightStereoToHeadLink3`, `kHeadLeftFisheyeToHeadLink3`, `kHeadRightFisheyeToHeadLink3`, `kHeadBackFisheyeToHeadLink3`, `kChassisFrontLidarToBaseLink`, `kChassisBackLidarToBaseLink`, `kChassisBackLidarToChassisFrontLidar`, `kChassisMid360ImuToChassisMid360Lidar`, `kChassisImuToBaseLink`, `kLeftHandRGBDToArmLEndLink`, `kRightHandRGBDToArmREndLink`, `kHeadRGBDToHeadLink3`

### Bundle (`agibot_gdk.Bundle`)
Bundled tensor data streaming.

| Method | Parameters | Returns |
|--------|-----------|---------|
| `get_latest_bundle(timeout)` | `float` ms | `BundleData` |
| `get_nearest_bundle(timestamp, timeout)` | `int` ns, `float` ms | `BundleData` |
| `close()` | none | `GDKRes` |

**BundleData attrs:** `.timestamp_ns`, `.bundle_info` (dict[str, BundleTensor])
**BundleTensor attrs:** `.dtype` (BundleDType), `.shape` (list[int]), `.data_view`
**BundleDType:** FLOAT16=1, FLOAT32=2, FLOAT64=3, INT8=4, INT16=5, INT32=6, INT64=7, UINT8=8, UINT16=9, UINT32=10, UINT64=11, STRING=12

### Interaction (`agibot_gdk.Interaction`)
Text-to-speech, audio/video playback, ASR, display control.

| Method | Parameters | Returns |
|--------|-----------|---------|
| `set_language(lang)` | `Language` (kLanguageChinese=0, kLanguageEnglish=1) | `GDKRes` |
| `set_volume(volume)` | `int32_t` | `GDKRes` |
| `set_wakeup_switch(is_on)` | `bool` | `GDKRes` |
| `set_audio_switch(is_on)` | `bool` | `GDKRes` |
| `set_display_switch(is_on)` | `bool` | `GDKRes` |
| `play_tts(text)` | `str` | `GDKRes` |
| `play_audio(audio_path)` | `str` | `GDKRes` |
| `play_video(video_path, loop_count)` | `str`, `int32_t` | `GDKRes` |
| `get_func_status()` | none | `VoiceFuncStatus` |
| `get_asr_text()` | none | `str` |

### ExperimentalAPI (`agibot_gdk.ExperimentalAPI`)
Object detection publishing, skill/taskflow integration.

| Method | Parameters | Returns |
|--------|-----------|---------|
| `publish_object_list(predn, T_list_4x4, obb_extents, camera_id, frame_id, timestamp_sec, kpt_num, include_keypoints)` | see C++ API | `GDKRes` |
| `publish_object_list_from_poses(T_list_4x4, pose_conf, obb_extents, camera_id, frame_id, timestamp_sec)` | see C++ API | `GDKRes` |
| `publish_skill_response(resp)` | `SkillResponse` | `GDKRes` |
| `publish_skill_state(state)` | `SkillState` | `GDKRes` |
| `get_last_skill_request(skill_name)` | `str` | `SkillRequest` or `None` |

### Clock (`agibot_gdk.Clock`)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `Clock.now_ns()` | none (static) | `uint64_t` nanoseconds |
| `Clock.now_sec()` | none (static) | `float` seconds |

---

## C++ API

Namespace: `agibot::gdk`. All methods return `GDKRes` (`kSuccess` on success) unless noted.

### Initialization
```cpp
GDKRes GDKInit();
GDKRes GDKRelease();
```

### Robot
```cpp
GDKRes GetJointStates(JointStates& joint_states);
GDKRes GetEndState(DualEndState& end_state);
GDKRes GetWholeBodyStatus(WholeBodyStatus& status);
GDKRes GetMotionControlStatus(MotionControlStatus& status);
GDKRes GetChassisPowerState(ChassisPowerState& state);
GDKRes GetChestPowerState(ChestPowerState& state);

GDKRes MoveEEPos(const JointStates& joint_states);
GDKRes MoveHeadJoint(const std::vector<double>& positions, const std::vector<double>& velocities);
GDKRes MoveWaistJoint(const std::vector<double>& positions, const std::vector<double>& velocities);
GDKRes MoveArmJoint(const std::vector<double>& positions, const std::vector<double>& velocities);
GDKRes JointControl(const JointControlReq& req);
GDKRes EndEffectorPoseControl(const EndEffectorPose& end_pose);  // NO collision detection!
GDKRes TrajTrackingControl(const A2DTrajectoryRequest& req);
GDKRes MotionPlanRequest(const MotionPlanReq& req);
GDKRes SetControlMode(const MotionControlMode& mode);
GDKRes SetLoad(const LoadInfo& info);
GDKRes SetComplianceParams(const std::map<std::string, double>& left,
                           const std::map<std::string, double>& right,
                           const std::string& detail = "");
GDKRes ForcePositionControl(const std::map<std::string, Pose>& targets);
GDKRes SetReferenceFramePoses(const double& lx, const double& ly, const double& lz, const double& lw,
                              const double& rx, const double& ry, const double& rz, const double& rw);
```

### Camera
```cpp
GDKRes GetLatestImage(const CameraType& type, const float timeout_ms, std::shared_ptr<Image>& img);
GDKRes GetNearestImage(const CameraType& type, const uint64_t ts_ns, const float timeout_ms, std::shared_ptr<Image>& img);
GDKRes GetImageShape(const CameraType& type, std::tuple<int,int>& shape);
GDKRes GetImageFps(const CameraType& type, float& fps);
GDKRes GetImageLatency(const CameraType& type, float window_s, LatencyStats& stats);
GDKRes GetCameraIntrinsic(const CameraType& type, CameraIntrinsic& intrinsic);
GDKRes SetDevCameraConfig(const std::string& cam_conf_path);
GDKRes CloseCamera();
```

### Imu
```cpp
GDKRes GetLatestImu(const ImuType& type, const float timeout_ms, std::shared_ptr<ImuData>& imu);
GDKRes GetNearestImu(const ImuType& type, const uint64_t ts_ns, const float timeout_ms, std::shared_ptr<ImuData>& imu);
GDKRes GetImuFps(const ImuType& type, float& fps);
GDKRes GetImuLatency(const ImuType& type, float window_s, LatencyStats& latency);
GDKRes CloseImu();
```

### Lidar
```cpp
GDKRes GetLatestPointCloud(const LidarType& type, const float timeout_ms, std::shared_ptr<PointCloud>& pc);
GDKRes GetNearestPointCloud(const LidarType& type, const uint64_t ts_ns, const float timeout_ms, std::shared_ptr<PointCloud>& pc);
GDKRes GetLidarFps(const LidarType& type, float& fps);
GDKRes GetLidarLatency(const LidarType& type, float window_s, LatencyStats& latency);
GDKRes CloseLidar();
```

### UltrasonicRadar
```cpp
GDKRes GetLatestUltrasonicRadar(std::shared_ptr<UltrasonicRadars>& data);
GDKRes GetNearestUltrasonicRadar(const uint64_t ts_ns, std::shared_ptr<UltrasonicRadars>& data);
GDKRes GetUltrasonicRadarFps(float& fps);
GDKRes GetUltrasonicRadarLatency(float window_s, LatencyStats& latency);
GDKRes Close();
```

### TF
```cpp
GDKRes GetAllTfFromBaseLink(std::vector<TransformStamped>& transforms);
GDKRes GetTfFromBaseLink(const std::string& child_frame_id, Transform& transform);
GDKRes GetTfFromSensor(const SensorExtrinsicType& type, Transform& transform);
GDKRes LookupTransformLatest(const std::string& target, const std::string& source, Transform& transform, uint64_t* timestamp_ns = nullptr);
GDKRes LookupTransform(const std::string& target, const std::string& source, uint64_t time_ns, Transform& transform);
bool CanTransform(const std::string& target, const std::string& source);
std::vector<std::string> GetAllFrameNames();
GDKRes GetLatestTimestamp(const std::string& frame_id, uint64_t& timestamp_ns);
void Clear();
```

### Bundle
```cpp
GDKRes GetLatestBundle(const float timeout_ms, std::shared_ptr<BundleData>& bundle);
GDKRes GetNearestBundle(const uint64_t ts_ns, const float timeout_ms, std::shared_ptr<BundleData>& bundle);
GDKRes Close();
```

### Interaction
```cpp
GDKRes SetLanguage(const Language& language);
GDKRes SetVolume(const int32_t& volume);
GDKRes SetWakeupSwitch(const bool& is_on);
GDKRes SetAudioSwitch(const bool& is_on);
GDKRes SetDisplaySwitch(const bool& is_on);
GDKRes PlayTts(const std::string& text);
GDKRes PlayAudio(const std::string& audio_path);
GDKRes PlayVideo(const std::string& video_path, const int32_t& loop_count);
GDKRes GetFuncStatus(VoiceFuncStatus& func_status);
GDKRes GetAsrText(std::string& asr_text);
```

### ExperimentalAPI
```cpp
GDKRes PublishObjectList(const std::vector<std::vector<double>>& predn,
                         const std::vector<const double*>& T_list_4x4,
                         const std::array<double, 3>& obb_extents,
                         int camera_id, const std::string& frame_id,
                         double timestamp_sec, int kpt_num, bool include_keypoints);
GDKRes PublishObjectListFromPoses(const std::vector<const double*>& T_list_4x4,
                                  const std::vector<double>& pose_conf,
                                  const std::array<double, 3>& obb_extents,
                                  int camera_id, const std::string& frame_id,
                                  double timestamp_sec);
GDKRes PublishSkillResponse(const SkillResponse& resp);
GDKRes PublishSkillState(const SkillState& state);
bool GetLastSkillRequest(const std::string& skill_name, SkillRequest& out);
```

### Clock
```cpp
static uint64_t Clock::NowNs();
static double Clock::NowSec();
```

### Slam
```cpp
GDKRes StartMapping();
GDKRes StopMapping();
GDKRes CancelMapping();
GDKRes RecordSpecLoc();
GDKRes GetOdomInfo(OdomInfo& odom_info);
GDKRes GetSlamState(uint32_t& state);
GDKRes GetCurrPose(Pose& pose);
```

### Pnc (Navigation)
```cpp
GDKRes NormalNavi(const NaviReq& req);
GDKRes HighPrecisionNavi(const NaviReq& req);
GDKRes RelativeMove(const NaviReq& req);
GDKRes CancelTask(uint32_t task_id);
GDKRes PauseTask(uint32_t task_id);
GDKRes ResumeTask(uint32_t task_id);
GDKRes GetTaskState(PNCTaskState& state);
GDKRes MoveChassis(const Twist& twist);
GDKRes RequestChassisControl(int32_t control_mode);
```

### Map
```cpp
GDKRes GetMap(const uint8_t map_id, MapInfo& info);
GDKRes GetCurrMap(MapName& name);
GDKRes GetAllMap(std::vector<MapName>& names);
GDKRes SwitchMap(const uint8_t map_id);
GDKRes RemoveMap(const uint8_t map_id);
```

---

## Key Structs

```cpp
// Geometry
struct Vector3 { double x, y, z; };
struct Quaternion { double x, y, z, w; };
struct Pose { Vector3 position; Quaternion orientation; };
struct Transform { Vector3 translation; Quaternion rotation; };
struct TransformStamped { std::string frame_id, child_frame_id; Transform transform; uint64_t timestamp_ns; };
struct Twist { Vector3 linear; Vector3 angular; };  // m/s, rad/s
struct Wrench { Vector3 force; Vector3 torque; };    // N, N.m

// Joint control
struct JointControlReq {
    double life_time{0.0};
    std::vector<std::string> joint_names{};
    std::vector<double> joint_positions{};   // radians
    std::vector<double> joint_velocities{};  // rad/s
    std::string detail{};
};

// End-effector pose control
struct EndEffectorPose {
    double life_time;
    int32_t group;  // EndEffectorControlGroup enum value
    Pose left_end_effector_pose;
    Pose right_end_effector_pose;
};

// Joint states
struct JointState {
    std::string name; uint32_t mode;
    double position, velocity, effort;           // rad, rad/s, Nm
    double motor_position, motor_velocity, motor_current;
    uint32_t error_code;
};
struct JointStates { size_t nums; std::vector<JointState> states; uint64_t timestamp; };

// Motor state
struct MotorState {
    uint32_t id; bool enable;
    double position, velocity, effort;  // rad, rad/s, Nm
    float current, voltage, temperature; // A, V, C
    uint32_t status, err_code;
};

// End-effector state
struct EndState { bool controlled; uint32_t type; std::vector<std::string> names; std::vector<MotorState> end_states; };
struct DualEndState { EndState left_end_state; EndState right_end_state; };

// Robot status
struct WholeBodyStatus {
    uint32_t right_arm_error, left_arm_error;
    bool right_arm_control, left_arm_control, right_arm_estop, left_arm_estop;
    uint32_t right_end_error, left_end_error;
    std::string right_end_model, left_end_model;
    uint32_t waist_error, lift_error, neck_error, chassis_error;
    uint64_t timestamp;
};

struct MotionControlStatus {
    std::vector<std::string> frame_names;
    std::vector<Pose> frame_poses;
    std::vector<std::string> collision_pairs_1, collision_pairs_2;
    uint8_t mode, error_code; std::string error_msg;
    std::vector<Twist> twists;
    std::vector<Wrench> wrenches;
};

// Sensor data
struct ImuData { Vector3 angular_velocity; Vector3 linear_acceleration; uint64_t timestamp_ns; };
struct LatencyStats { double max_latency_ms, avg_latency_ms, p99_latency_ms, p999_latency_ms, p9999_latency_ms; };
struct CameraIntrinsic { std::vector<double> intrinsic; std::vector<double> distortion; };

// Navigation
struct NaviReq { Pose target; uint64_t timestamp_ns; };
struct OdomInfo {
    PoseWithCovariance pose; TwistWithCovariance twist;
    bool is_stationary, is_sliping; int32_t loc_confidence, loc_state;
    Vector3 velocity, velocity_body, acceleration, ang_vel, orientation_euler;
};

// Compliance
struct ComplianceParam {
    double translational_stiffness, translational_damping;
    double rotational_stiffness, rotational_damping;
    double nullspace_stiffness, joint1_nullspace_stiffness;
    double translational_clip_neg_x/y/z, translational_clip_x/y/z;
    double rotational_clip_neg_x/y/z, rotational_clip_x/y/z;
    double translational_ki, rotational_ki;
};

// Load
struct LoadInfo { double mass; std::array<double,3> f_x_center_load; std::array<double,9> load_inertia; };

// Skills/Taskflow
struct SkillRequest { uint64_t timestamp_ns; std::string skill_name; uint32_t command; /* 0=START,1=CANCEL,2=PAUSE,3=RESUME,4=SWITCH,5=RESET */ std::string command_source, params_json, uuid, detail; double timeout; };
struct SkillResponse { std::string frame_id; uint64_t timestamp_ns; uint32_t result; /* 0=SUCCESS,1=FAIL */ std::string uuid, reason; };
struct SkillState { uint64_t timestamp_ns; std::string name; uint32_t state; /* 0=UNKNOWN,1=RUNNING,2=SUCCESS,3=FAILURE */ uint32_t seq; std::string detail; };
```

---

## Control Modes

### MotionControlMode
```cpp
struct MotionControlMode {
    enum class InputSource : uint8_t {
        INPUT_AUTO = 0,     // System/task management
        INPUT_TELEOP = 1,   // VR/MoCap/gamepad
        INPUT_HMI = 2,      // On-body HMI/tablet
        INPUT_END_BTN = 3,  // End-effector button
        INPUT_VLA = 4,      // VLA/large model
        INPUT_RL = 5,       // Reinforcement learning
        INPUT_GDK = 6       // SDK/external API (default)
    } input_source;

    enum class Target : uint8_t {
        TARGET_ARMS = 0,       // Both arms
        TARGET_LEFT_ARM = 1,   // Left arm only
        TARGET_RIGHT_ARM = 2   // Right arm only
    } target;

    enum class ControlMode : uint8_t {
        CTRL_AUTO_POS = 0,             // Auto position control
        CTRL_JOINT_POSITION = 1,       // Joint PD/trajectory
        CTRL_CARTESIAN_IMPEDANCE = 2,  // Cartesian impedance
        CTRL_JOINT_IMPEDANCE = 3,      // Joint impedance
        CTRL_GRAVITY_COMP_TEACH = 4    // Gravity compensation / teach mode
    } control_mode;

    enum class SafeMode : uint8_t {
        SAFE_NORMAL = 0,   // Normal
        SAFE_REDUCED = 1,  // Reduced speed
        SAFE_STOP = 2      // Safety stop
    } safe_mode;

    uint8_t priority{10};  // 0-255, higher = more priority
};
```

### EndEffectorControlGroup
```cpp
enum class EndEffectorControlGroup {
    kUnknown = 0,
    kLeftArm = 4, kRightArm = 8, kBothArms = 12,
    kLeftArmWaistLift = 20, kRightArmWaistLift = 24, kBothArmsWaistLift = 28,
    kLeftArmWaistPitch = 36, kRightArmWaistPitch = 40, kBothArmsWaistPitch = 44,
    kLeftArmWaist = 52, kRightArmWaist = 56, kBothArmsWaist = 60,
};
```

---

## Motion Planning

### PlanningGroup
```cpp
enum class PlanningGroup : uint8_t {
    HEAD_YAW = 0, HEAD_PITCH = 1,
    LEFT_ARM = 2, RIGHT_ARM = 3,
    LEFT_TOOL = 4, RIGHT_TOOL = 5,
    WAIST_LIFT = 6, WAIST_PITCH = 7,
    CHASSIS = 8, HEAD_ROLL = 9, WAIST = 10
};
```

### PlanningTarget
```cpp
struct PlanningTarget {
    uint8_t group_id;      // PlanningGroup value
    uint8_t target_type;   // PlanningTargetType: POSE=0, JOINT=1, PASSIVE=2

    // For POSE target:
    std::string target_frame, base_frame;
    Pose target_pose;
    std::vector<double> target_velocity;

    // For JOINT target:
    std::vector<std::string> target_joint_names;
    std::vector<double> target_joint_positions;
    std::vector<double> target_joint_velocity;
};
```

### MotionPlanReq
```cpp
struct MotionPlanReq {
    uint8_t input_type;   // PlanInputType: INPUT_VR=51, INPUT_MOCAP=52, INPUT_HMI=53, INPUT_GDK=54 (default)
    uint8_t type;         // MotionPlanType: PLAN_JOINT_SPACE=0, PLAN_CARTESIAN_SPACE=1, INTERP_JOINT_SPACE=2, INTERP_CARTESIAN_SPACE=3
    std::vector<PlanningTarget> targets;

    uint8_t time_parametization;  // TimeParam: SCALED_TIME=0, ACCURATE_TIME=1
    double time{0.5};             // Motion time (seconds)
    double timeout{10.0};         // Planning timeout

    bool enable_env_collision{true};
    bool enable_self_collision{true};
    bool enable_com_check{false};

    std::string uuid;
    uint64_t seed{0};
    std::string root_log_path;
    uint8_t log_level;  // PlanLogLevel: VERBOSE=0, DEBUG=1, RELEASE=2, NONE=3
};
```

---

## A2D Trajectory Tracking

```cpp
struct A2DArmAction {
    std::string control_type;  // "ABS_POSE", "DELTA_POSE", "END_SPEED", "ABS_JOINT", etc.
    std::vector<double> action_data;  // 7 for pose, 6 for speed, N for joints
};
struct A2DHeadJointsAction { std::string control_type; std::vector<double> action_data; };
struct A2DWaistJointsAction { std::string control_type; std::vector<double> action_data; };
struct A2DEndEffectorAction { std::string control_type; std::vector<double> action_data; };

struct A2DTrajectoryAction {
    A2DArmAction left_arm, right_arm;
    A2DHeadJointsAction head;
    A2DWaistJointsAction waist;
    A2DEndEffectorAction left_effector, right_effector;
};

struct A2DTrajectoryRequest {
    std::string robot_link{"base_link"};
    double trajectory_reference_time{1.0};
    std::vector<A2DTrajectoryAction> actions;
};
```

---

## Joint Names & Limits

From `gdk/config/t2_soft_limit.json`. All values in radians.

| Config Key | Joint Name | Min (rad) | Max (rad) | Group |
|-----------|-----------|-----------|-----------|-------|
| `idx01_body_joint1` | `body_joint1` | -1.082 | 0.000 | Waist/Lift |
| `idx02_body_joint2` | `body_joint2` | 0.000 | 2.653 | Waist/Lift |
| `idx03_body_joint3` | `body_joint3` | -1.920 | 1.571 | Waist/Lift |
| `idx04_body_joint4` | `body_joint4` | -0.436 | 0.436 | Waist/Lift |
| `idx05_body_joint5` | `body_joint5` | -3.046 | 3.046 | Waist/Lift |
| `idx11_head_joint1` | `head_joint1` | -1.571 | 1.571 | Head (yaw) |
| `idx12_head_joint2` | `head_joint2` | -0.349 | 0.349 | Head (pitch) |
| `idx13_head_joint3` | `head_joint3` | -0.535 | 0.535 | Head (roll) |
| `idx21_arm_l_joint1` | `arm_l_joint1` | -3.072 | 3.072 | Left arm |
| `idx22_arm_l_joint2` | `arm_l_joint2` | -2.060 | 2.060 | Left arm |
| `idx23_arm_l_joint3` | `arm_l_joint3` | -3.072 | 3.072 | Left arm |
| `idx24_arm_l_joint4` | `arm_l_joint4` | -2.496 | 1.012 | Left arm |
| `idx25_arm_l_joint5` | `arm_l_joint5` | -3.072 | 3.072 | Left arm |
| `idx26_arm_l_joint6` | `arm_l_joint6` | -1.012 | 1.012 | Left arm |
| `idx27_arm_l_joint7` | `arm_l_joint7` | -1.536 | 1.536 | Left arm |
| `idx61_arm_r_joint1` | `arm_r_joint1` | -3.072 | 3.072 | Right arm |
| `idx62_arm_r_joint2` | `arm_r_joint2` | -2.060 | 2.060 | Right arm |
| `idx63_arm_r_joint3` | `arm_r_joint3` | -3.072 | 3.072 | Right arm |
| `idx64_arm_r_joint4` | `arm_r_joint4` | -2.496 | 1.012 | Right arm |
| `idx65_arm_r_joint5` | `arm_r_joint5` | -3.072 | 3.072 | Right arm |
| `idx66_arm_r_joint6` | `arm_r_joint6` | -1.012 | 1.012 | Right arm |
| `idx67_arm_r_joint7` | `arm_r_joint7` | -1.536 | 1.536 | Right arm |

---

## DDS Topics (from sensors.h CAMERA_DDS_TOPIC map)

### Camera DDS Topics
| CameraType | DDS Topic |
|-----------|-----------|
| `kHeadBackFisheye` | `/camera/head_back_fisheye` |
| `kHeadLeftFisheye` | `/camera/head_left_fisheye` |
| `kHeadRightFisheye` | `/camera/head_right_fisheye` |
| `kHeadStereoLeft` | `/camera/head_stereo_left` |
| `kHeadStereoRight` | `/camera/head_stereo_right` |
| `kHandLeftColor` | `/camera/hand_left_color` |
| `kHandRightColor` | `/camera/hand_right_color` |
| `kHeadColor` | `/camera/head_color` |
| `kHeadDepth` | `/camera/head_depth` |
| `kHandLeftUpperColor` | `/camera/hand_left_upper_color` |
| `kHandRightUpperColor` | `/camera/hand_right_upper_color` |
| `kHandLeftLowerColor` | `/camera/hand_left_lower_color` |
| `kHandRightLowerColor` | `/camera/hand_right_lower_color` |
| `kHandLeftUpperDepth` | `/camera/hand_left_upper_depth` |
| `kHandRightUpperDepth` | `/camera/hand_right_upper_depth` |
| `kHandLeftLowerDepth` | `/camera/hand_left_lower_depth` |
| `kHandRightLowerDepth` | `/camera/hand_right_lower_depth` |

### IMU DDS Topics
| ImuType | DDS Topic |
|---------|-----------|
| `kImuFront` | `/imu/livox_front` |
| `kImuBack` | `/imu/livox_back` |
| `kImuChassis` | `/imu/chassis` |

### Lidar DDS Topics
| LidarType | DDS Topic |
|-----------|-----------|
| `kLidarFront` | `/lidar/livox_front` |
| `kLidarBack` | `/lidar/livox_back` |

---

## DDS API (Low-Level)

For custom pub/sub on protobuf-based DDS topics.

```cpp
#include "common/dds.h"

// QoS configuration
struct QoS {
    QoSHistory history = QoSHistory::kKeepLast;     // kKeepLast=0, kKeepAll=1
    int depth = 5;
    QoSReliability reliability = QoSReliability::kReliable;  // kBestEffort=0, kReliable=1
    QoSDurability durability = QoSDurability::kTransientLocal;  // kVolatile=0, kTransientLocal=1, kTransientLocalAll=2
};

// Publisher
std::shared_ptr<Publisher> CreatePublisher(const std::string& topic_name, const std::string& topic_type, const QoS& qos = QoS());
bool Publisher::Publish(const std::shared_ptr<google::protobuf::Message>& message, const std::shared_ptr<PubMsgInfo>& pub_info = nullptr);

// Subscriber
using SubscriberCallback = std::function<void(const std::shared_ptr<google::protobuf::Message>&, const std::shared_ptr<SubMsgInfo>&)>;
std::shared_ptr<Subscriber> CreateSubscriber(const std::string& topic_name, const std::string& topic_type, const SubscriberCallback& callback, const QoS& qos = QoS());

// Client (service call)
std::shared_ptr<Client> CreateClient(const std::string& service_name, const std::string& request_type, const std::string& response_type, const QoS& qos = QoS());
bool Client::WaitForService(int timeout_ms = -1);
std::shared_ptr<google::protobuf::Message> Client::Call(const std::shared_ptr<google::protobuf::Message>& request, int timeout_ms = -1);
std::shared_future<...> Client::AsyncCall(const std::shared_ptr<google::protobuf::Message>& request, callback = nullptr);

// Service (server)
using ServiceCallback = std::function<std::shared_ptr<google::protobuf::Message>(const std::shared_ptr<google::protobuf::Message>&)>;
std::shared_ptr<Service> CreateService(const std::string& service_name, const std::string& request_type, const std::string& response_type, const ServiceCallback& callback, const QoS& qos = QoS());
```

---

## GDKRes Return Codes

| Value | Name | Description |
|-------|------|-------------|
| 0 | `kSuccess` | Success |
| 1 | `kInvalidInput` | Invalid input parameter |
| 2 | `kInvalidOutput` | Invalid output parameter |
| 3 | `kRuntimeError` | Runtime error |
| 4 | `kUnknown` | Unknown error |

---

## ROS2 Forwarding Packages

| Package | Launch Command |
|---------|---------------|
| `gdk_camera` | `ros2 launch gdk_camera camera.launch` |
| `gdk_controller` | `ros2 launch gdk_controller controller.launch` |
| `gdk_imu` | `ros2 launch gdk_imu imu.launch` |
| `gdk_lidar` | `ros2 launch gdk_lidar lidar.launch` |
