# ratslam

This package is a port of the original [ratslam_ros](https://openslam-org.github.io/openratslam.html) repository to ROS 2 (tested on ROS 2 Rolling).

## Dependencies

In addition to standard ROS 2 dependencies, this package **requires the custom `topological_msgs` package**. Make sure it is available and built in your workspace before building `ratslam`.

Main dependencies:
- `rclcpp`
- `topological_msgs` (custom)
- `cv_bridge`
- `sensor_msgs`
- `geometry_msgs`
- `nav_msgs`
- `visualization_msgs`
- `tf2_ros`
- `image_transport`
- `Boost` (component `serialization`)
- `OpenCV`
- `Irrlicht`
- `OpenGL`

## Installation


1. **Clone the `ratslam` and `topological_msgs` repositories into your workspace:**

   ```bash
   cd ~/rolling_ws/src
   git clone https://github.com/OpenRatSLAM2/ratslam.git
   git clone https://github.com/OpenRatSLAM2/topological_msgs.git
   ```

2. **Install system dependencies:**

   ```bash
   sudo apt update
   sudo apt install ros-rolling-cv-bridge ros-rolling-sensor-msgs ros-rolling-geometry-msgs ros-rolling-nav-msgs ros-rolling-visualization-msgs ros-rolling-tf2-ros ros-rolling-image-transport libboost-all-dev libopencv-dev libirrlicht-dev libopengl-dev
   ```

## ROS 2 Humble (Ubuntu 22.04) — 빌드 및 실행

This project was originally tested on ROS 2 Rolling. The steps below explain how to build and run on Ubuntu 22.04 with ROS 2 Humble.

Prerequisites

Install system packages (one-time):
```bash
sudo apt update
sudo apt install -y \
   ros-humble-cv-bridge ros-humble-sensor-msgs ros-humble-geometry-msgs \
   ros-humble-nav-msgs ros-humble-visualization-msgs ros-humble-tf2-ros \
   ros-humble-image-transport \
   libboost-all-dev libopencv-dev libirrlicht-dev libglu1-mesa-dev libgl1-mesa-dev
```

## 로컬 포팅: ROS 2 Humble (Ubuntu 22.04)

이 저장소는 OpenRatSLAM2/ratslam을 로컬 환경(Ubuntu 22.04 + ROS 2 Humble)에서 동작하도록 일부 수정한 내용이 포함되어 있습니다.
아래는 로컬에서 적용한 변경사항, 빌드 방법 및 주의사항입니다.

### 주요 변경사항 (로컬)
- 새 패키지 추가
   - `src/topological_msgs/` (5개 `.msg` 파일 포함)
      - ViewTemplate.msg, TopologicalNode.msg, TopologicalEdge.msg, TopologicalMap.msg, TopologicalAction.msg
   - 목적: `topological_msgs` 원격 리포지토리 없이 로컬에서 메시지 생성
- `ratslam` 패키지 수정
   - `CMakeLists.txt`: `cmake_minimum_required(VERSION 4.0)` → `3.16` (로컬 CMake 호환)
   - `cmake/FindIrrlicht.cmake`: 시스템 표준 include/lib 경로와 여러 라이브러리 이름 후보 추가 (Irrlicht 링크 문제 해결)
- README에 Humble 빌드 지침 추가
- 변경사항은 로컬 브랜치 `humble-adapt`에 커밋되어 있음(권장: 별도 브랜치로 관리)

### 빌드 및 실행 (요약)
1. 시스템 의존성 설치 (한 번만)
```bash
sudo apt update
sudo apt install -y \
   ros-humble-cv-bridge ros-humble-image-transport ros-humble-geometry-msgs \
   ros-humble-nav-msgs ros-humble-visualization-msgs ros-humble-tf2-ros \
   libboost-all-dev libopencv-dev libirrlicht-dev libglu1-mesa-dev libgl1-mesa-dev
```

2. 워크스페이스 빌드
```bash
cd ~/rolling_ws
source /opt/ros/humble/setup.bash
colcon build --packages-select topological_msgs ratslam
source install/setup.bash
ros2 interface show topological_msgs/msg/TopologicalMap
```

3. 실행 예
- 터미널 A:
```bash
source /opt/ros/humble/setup.bash
source ~/rolling_ws/install/setup.bash
ros2 launch ratslam irataus.launch
```
- 터미널 B (bag 재생):
```bash
ros2 bag play /path/to/your.db3 --rate 1.0 --clock --start-paused --topics /irat_red/odom /irat_red/camera/image/compressed
# 스페이스로 재생/일시정지
```

### 주의사항 & 트러블슈팅
- 메시지 빌드 에러(`rosidl_generate_interfaces`)가 발생하면 `topological_msgs/package.xml`에 다음 태그가 있는지 확인:
```xml
<member_of_group>rosidl_interface_packages</member_of_group>
```
- Irrlicht 관련 헤더/링커 에러가 나면 `libirrlicht-dev`가 설치되어 있는지 확인하세요.
- upstream이 `cmake_minimum_required(VERSION 4.0)`를 요구하면 로컬에서 임시로 낮추었으므로, upstream과 동기화할 때 충돌 가능성이 있습니다. 변경은 별도 브랜치(`humble-adapt`)로 관리하세요.
- 데이터 파일(.db3, .zip 등)은 레포에 커밋하지 마세요. `.gitignore`에 추가하고 Google Drive 등 외부 스토리지에 보관하세요.

### 권장 .gitignore (레포 루트에 추가)
```
build/
install/
log/
data/*.db3
data/*.zip
.vscode/
__pycache__/
```

위 내용을 원하시면 제가 README에 적용(이미지·링크 정리 포함)하고 현재 브랜치에 커밋해 드리겠습니다.


Build (example):
```bash
cd ~/rolling_ws
source /opt/ros/humble/setup.bash
# If you made local changes or to ensure a clean build, you can remove build artifacts:
# rm -rf build/ install/ log/
colcon build --packages-select topological_msgs ratslam
```

After build:
```bash
source ~/rolling_ws/install/setup.bash
ros2 interface show topological_msgs/msg/TopologicalMap
```

Troubleshooting
- If `rosidl_generate_interfaces` fails, ensure `topological_msgs/package.xml` contains:
   `<member_of_group>rosidl_interface_packages</member_of_group>`
- If you see `irrlicht/irrlicht.h` not found or linkage errors, install `libirrlicht-dev` (see apt command above) and ensure CMake can find the library.
- If there are CMake version issues (e.g., upstream requires 4.x), you may lower `cmake_minimum_required` locally, but keep this in a branch to avoid upstream conflicts.

If you want, create a branch and commit your local Humble-specific adjustments (CMakeLists, FindIrrlicht changes, adding topological_msgs) so you can continue tracking them separately from upstream.

3. **Build the workspace:**

   ```bash
   cd ~/rolling_ws
   source /opt/ros/rolling/setup.bash
   colcon build --symlink-install
   ```

4. **Source the workspace:**

   ```bash
   source ~/rolling_ws/install/setup.bash
   ```

## Usage
To use this package with your datasets, first convert the dataset to a ROS 2 bag format, if necessary. Access the datasets google drive link https://drive.google.com/drive/folders/1ggAxMzIyadmenUPoAon2rL8gvJvuv75K?usp=drive_link for examples.

First, run your dataset launch file, for example:

```bash
ros2 launch ratslam irataus.launch
```

```bash
ros2 launch ratslam oxford_newcollege.launch
```

```bash
ros2 launch ratslam stlucia.launch
```

In another terminal, run your bag:

```bash
ros2 bag play data/irat_aus_28112011/irat_aus_28112011.db3 --rate 1.0 --clock --start-paused --topics /irat_red/odom /irat_red/camera/image/compressed
```

```bash
ros2 bag play data/oxford_newcollege/oxford_newcollege.db3 --rate 1.0 --clock --start-paused --topics /irat_red/odom /irat_red/camera/image/compressed
```

```bash
ros2 bag play data/stlucia_2007/stlucia_2007.db3 --rate 1.0 --clock --start-paused --topics /irat_red/odom /irat_red/camera/image/compressed
```
