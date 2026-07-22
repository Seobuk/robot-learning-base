# CLAUDE.md

코드 작성/리뷰 행동 지침은 이 repo의 `karpathy-guidelines` 스킬을 따른다 — 핵심: 참조된 파일은 열어 확인한 뒤 답하고, 요청된 최소 변경만, 모호하면 추측 대신 질문. 사소한 작업(오타 수정, 자명한 one-liner)은 재량껏.

파괴적/되돌리기 어려운 작업은 실행 전 확인: 파일·브랜치 삭제, `rm -rf`, 테이블 드롭, `git push --force`, `git reset --hard`, 외부에 보이는 작업(push, PR/이슈 코멘트). 막혔다고 파괴적 우회(`--no-verify`, 낯선 진행 중 파일 폐기)로 풀지 않는다.

## Project-Specific Guidelines — 로봇 학습 베이스

두 스택을 각자 독립 환경으로 나란히 둔다. 스택 작업 시 해당 폴더의 CLAUDE.md를 우선 따른다(더 구체적인 것이 이긴다):
- `isaac/` — Isaac Lab / Isaac Sim RL (Python 3.11). → `isaac/CLAUDE.md`
- `ros2_ws/` — ROS 2 Jazzy + MuJoCo 인터페이스/실기 (Python 3.12). → `ros2_ws/CLAUDE.md`

**git에 올리지 않는 것**: 체크포인트/가중치, 데이터셋, rosbag, 런 출력(`outputs/ logs/ wandb/ checkpoints/`), colcon `build/ install/ log/` — 이미 `.gitignore`에 있음. `git add -A`/`git add .` 대신 `git status`로 확인 후 스테이징(멀티-GB 바이너리가 히스토리에 박히면 영구 오염). 가중치 공유는 DVC/HuggingFace Hub/release asset으로, git 아님.

**하드웨어 안전**: `ros2_ws` 노드는 실제 로봇을 움직일 수 있다. 사람이 "실기"라고 명시하기 전엔 기본값은 시뮬레이션(`use_sim_time:=true`)이고 E-stop이 손 닿는 곳에 있다고 가정. 테스트 통과 목적으로 torque/velocity/current 리밋을 올리지 않는다. 캘리브레이션 상수(서보 트림 예: PCA9685, 카메라 extrinsics, IMU offset)는 실물에 맞춘 값이니 반올림/정리하지 않는다.

**환경 분리**: Isaac Lab(Py 3.11)과 ROS 2(Jazzy, Py 3.12)를 한 환경에 같이 설치하지 않는다 — 충돌한다.

**의존성**: import를 고치려고 라이브 환경에서 즉석 `pip install` 하지 않는다. 스택 매니페스트(pyproject/pixi)에 추가하고 lockfile 재생성 후 커밋.

**생성 디렉토리는 편집하지 않는다**(읽기 전용): `build/ install/ log/ outputs/ multirun/ wandb/`. ROS 코드는 `src/`에서만 편집.

**재현성**: seed는 config에서 온다(python/numpy/torch/env에 적용하고 값 로깅). 런 하나 = 디렉토리 하나, resolved config 스냅샷이 "실제로 뭐가 돌았나"의 소스 오브 트루스. 런마다 git SHA + config + 주요 버전(torch/cuda/isaac/ros distro) 기록. GPU 완전 결정론은 기대하지 말고 N-seed 분산으로 보고.

**비밀/경로**: W&B 키·로봇 IP·크레덴셜은 커밋된 yaml이 아니라 `.env`(gitignore)에서 읽는다(`.env.example` 참고). 절대경로/사용자 데이터 디렉토리 하드코딩 금지 — config/env에서 루트를 읽는다.

**스킬**: `.claude/skills/`의 스킬은 repo와 함께 이동한다(전역 적용은 `~/.claude/skills/`로 복사). 코드 작업 행동 지침은 `karpathy-guidelines` 스킬이 담당 — 로봇 전용 스킬은 실제 반복 작업이 생기면 추가.

**스택 버전 핀**(조사 기준 2026-07, fragile하니 함부로 올리지 말 것): Isaac Lab 2.3.x / Isaac Sim 5.1 / PyTorch 2.7+CUDA 12.8 · ROS 2 Jazzy + MuJoCo 3.x + `mujoco_ros2_control`. 구체 명령은 각 스택 CLAUDE.md.
