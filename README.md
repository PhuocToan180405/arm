# Robot Arm 4-DOF với Panda Gripper - ROS 2 Humble

Robot arm 4 bậc tự do với gripper Panda cho ROS 2 Humble và Gazebo Classic.

## Mô tả
Robot arm này được chuyển đổi từ mô hình 6-DOF, giảm xuống còn 4 bậc tự do chính và được trang bị gripper 2 ngón từ Panda robot. Hỗ trợ simulation trong Gazebo Classic với ros2_control.

## Yêu cầu hệ thống
- Ubuntu 22.04
- ROS 2 Humble
- Gazebo Classic 11
- Python 3.10+

## Cài đặt

### 1. Cài đặt ROS 2 Humble
```bash
# Cập nhật system
sudo apt update && sudo apt upgrade -y

# Cài đặt ROS 2 Humble Desktop
sudo apt install ros-humble-desktop -y

# Source ROS 2
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 2. Cài đặt Gazebo Classic
```bash
# Cài đặt Gazebo 11
sudo apt install gazebo -y
sudo apt install ros-humble-gazebo-ros-pkgs -y
```

### 3. Cài đặt Dependencies
```bash
# ROS 2 Control
sudo apt install ros-humble-ros2-control -y
sudo apt install ros-humble-ros2-controllers -y
sudo apt install ros-humble-gazebo-ros2-control -y

# Các package khác
sudo apt install ros-humble-xacro -y
sudo apt install ros-humble-joint-state-publisher -y
sudo apt install ros-humble-joint-state-publisher-gui -y
sudo apt install ros-humble-robot-state-publisher -y
sudo apt install ros-humble-controller-manager -y

# Build tools
sudo apt install python3-colcon-common-extensions -y
```

### 4. Clone và Build Workspace
```bash
# Tạo workspace
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# Clone repository
git clone https://github.com/PhuocToan180405/arm.git

# Quay về workspace root
cd ~/ros2_ws

# Build
colcon build --symlink-install

# Source workspace
source ~/ros2_ws/install/setup.bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

## Sử dụng

### Hiển thị trong RViz
```bash
# Terminal 1: Launch RViz
ros2 launch six_dof_arm display_rviz.launch.py
```
Sử dụng GUI để điều khiển các joints và gripper.

### Chạy Simulation trong Gazebo
```bash
# Terminal 1: Launch Gazebo
ros2 launch six_dof_arm gazebo.launch.py
```

### Điều khiển Robot
```bash
# Terminal 2: Điều khiển arm joints
ros2 topic pub /arm_controller/joint_trajectory trajectory_msgs/msg/JointTrajectory "
joint_names: ['joint_1', 'joint_2', 'joint_3', 'joint_4']
points:
  - positions: [0.5, 0.5, 0.5, 0.5]
    time_from_start:
      sec: 2
      nanosec: 0
" --once

# Điều khiển gripper - Đóng
ros2 topic pub /gripper_controller/commands std_msgs/msg/Float64MultiArray "
data: [0.0, 0.0]
" --once

# Điều khiển gripper - Mở
ros2 topic pub /gripper_controller/commands std_msgs/msg/Float64MultiArray "
data: [0.04, 0.04]
" --once
```

### Kiểm tra trạng thái
```bash
# Xem các joints hiện tại
ros2 topic echo /joint_states

# Xem các controllers đang chạy
ros2 control list_controllers

# Xem thông tin robot
ros2 topic echo /robot_description
```

## Cấu trúc Project
```
├── gazebo_ros2_control/        # Plugin Gazebo ros2_control
├── gesture_recognition/         # Nhận dạng cử chỉ
├── six_dof_arm/                # Package chính
│   ├── config/                 # Các file cấu hình
│   │   └── joint_names_six_dof_arm.yaml
│   ├── launch/                 # Launch files
│   │   ├── display_rviz.launch.py
│   │   └── gazebo.launch.py
│   ├── meshes/                 # Mesh files 3D
│   └── urdf/                   # URDF files
│       └── six_dof_arm.urdf
└── six_dof_arm_moveit_config/  # MoveIt configuration

```

## Thông số kỹ thuật
- **Arm Joints**: 4 revolute/continuous joints
  - joint_1: Continuous (xoay vô hạn)
  - joint_2, joint_3, joint_4: Revolute (±1.57 rad)
- **Gripper**: 2 prismatic finger joints
  - Range: 0 - 0.04m
  - Max effort: 20N
  - Max velocity: 0.2 m/s

## Xử lý sự cố

### Gazebo không khởi động
```bash
killall gzserver gzclient
ros2 launch six_dof_arm gazebo.launch.py
```

### Controllers không load
```bash
# Kiểm tra controller manager
ros2 control list_controllers

# Load thủ công
ros2 control load_controller --set-state active joint_state_broadcaster
ros2 control load_controller --set-state active arm_controller
ros2 control load_controller --set-state active gripper_controller
```

### Build errors
```bash
# Clean và rebuild
cd ~/ros2_ws
rm -rf build install log
colcon build --symlink-install
```

## License
Apache 2.0 - xem file [LICENSE](LICENSE)

## Đóng góp
Mọi đóng góp đều được chào đón! Hãy tạo issue hoặc pull request.