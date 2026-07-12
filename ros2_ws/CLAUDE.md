# ros2_ws/ — ROS 2 + MuJoCo 인터페이스 스택

로봇/시뮬레이션 **인터페이스**(실시간, 소싱된 오버레이). 이 워크스페이스는 URDF/MJCF/메시와 ros2_control 인터페이스를 소유한다.
**학습 코드(torch/gymnasium/RL)는 여기 넣지 않는다** — 별도 pip/uv 파이썬 패키지로. colcon 패키지에 RL 의존성 추가 금지.

## 스택 핀

- ROS 2 **Jazzy Jalisco** (Ubuntu 24.04, Python 3.12, Tier-1, EOL 2029-05) 기본. 보수적 대안 Humble(22.04, EOL 2027-05). Kilted/비-LTS는 학습 스택엔 너무 최신.
- MuJoCo **3.x** (`pip install mujoco` — 네이티브 lib 번들, 학습쪽 별도 C 빌드 불필요).
- distro와 Ubuntu 짝 고정: Jazzy=24.04, Humble=22.04. 크로스-distro apt 설치 시도 금지.

## 빌드 / 실행

- 빌드: `colcon build --symlink-install` (Python 빠른 반복). 노드 실행 전 `source install/setup.bash`.
- **코드는 `src/`에서만 편집.** `install/`(심링크/생성물) 편집분은 다음 빌드에서 사라진다.
- 빌드 타입: 제어/description 패키지는 `ament_cmake`, 순수 노드(브리지, RL 인터페이스)는 `ament_python`. `package.xml`/`CMakeLists`와 일치시키고 섞지 마라.
- **conda 금지**(rosdep/ament 깨짐). 학습용 venv/컨테이너만 별도.

## MuJoCo 통합

- 공식 `ros-controls/mujoco_ros2_control`(ros2_control `SystemInterface` 플러그인, FT/IMU/카메라/lidar 플러그인, URDF→MJCF 처리) 사용. 대안: `dfki-ric/mujoco_ros2_control`.
- 커스텀 브리지는 MJX/GPU 배치 sim이 필요할 때만. 이미 있는 SystemInterface를 재구현하지 마라.

## 자산 단일 출처

- MJCF/URDF/메시는 집이 **하나**: `my_robot_description`. 학습 env는 같은 MJCF를 패키지 상대경로로 로드 — RL용 두 번째 모델 복사본 만들지 마라(drift).
- 주의: URDF ≠ MJCF. MuJoCo는 contact/solver/actuator gear/geom group/damping 등 MJCF 고유 튜닝이 필요 — 순진한 URDF→MJCF import는 돌긴 해도 transfer 나쁨.

## launch / param / 시계

- launch는 `.launch.py`(composable, substitution/condition). `sim.launch.py`와 `real.launch.py`가 **동일 컨트롤러**를 include하고 ros2_control 하드웨어 플러그인만 교체 → 정책이 sim→real 그대로 배포.
- param은 각 패키지 `config/`의 YAML. sim-only/real-only는 launch arg로 분리.
- MuJoCo/Gazebo 아래선 항상 `use_sim_time:=true` (TF/컨트롤러 타이밍 일치).

## sim-to-real

- 1일차부터 domain randomization(질량/마찰/PD gain) + actuator dynamics + 센서 노이즈 + action/observation latency. 없으면 zero-shot 실패.
- DR 범위 검증: ~1000 랜덤 에피소드 돌려 물리 실패(NaN/폭발) 비율 <~1% 유지.
- 캘리브레이션 경로 남겨라: 실제 actuator/센서는 모델과 다르다(gain/offset/latency) — sim 값 하드코딩 말고 `config/`에 노출.

## gitignore됨

`build/ install/ log/`, rosbag(`*.bag *.db3 *.mcap`), 체크포인트. 절대 커밋 금지 — 머신 종속 생성물/거대 바이너리.
