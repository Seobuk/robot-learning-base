# CLAUDE.md

로봇 학습(robot learning) 프로젝트 베이스. 두 스택을 **각자 독립 환경**으로 나란히 둔다:

- `isaac/` — NVIDIA Isaac Lab / Isaac Sim RL 스택 (GPU 병렬 sim). → `isaac/CLAUDE.md`
- `ros2_ws/` — ROS 2 + MuJoCo 인터페이스 + 실기(real robot). → `ros2_ws/CLAUDE.md`

**가장 구체적인 CLAUDE.md가 이긴다.** 스택 작업 시 해당 폴더의 CLAUDE.md를 우선 따른다.
행동 가이드라인은 `.claude/skills/karpathy-guidelines` (assumptions 명시, 최소 구현, 수술적 변경, 검증 가능한 목표) — ponytail과 함께 항상 적용.

## 절대 규칙 (non-negotiable)

1. **git에 올리지 않는 것**: 체크포인트/가중치, 데이터셋, rosbag, 런 출력(`outputs/ logs/ wandb/ checkpoints/`), colcon `build/ install/ log/`. 이미 `.gitignore`에 있음.
   `git add -A` / `git add .` 금지 — 멀티-GB 바이너리가 히스토리에 박히면 영구 오염. 커밋 전 `git status`로 확인.
   가중치를 공유해야 하면 DVC / HuggingFace Hub / release asset으로 승격(promote), git 아님.
2. **하드웨어 안전**: `ros2_ws/`의 노드는 실제 로봇을 움직일 수 있다. 사람이 명시적으로 "실기"라고 하기 전엔 **기본값은 시뮬레이션**(`use_sim_time:=true`). E-stop이 손 닿는 곳에 있다고 가정.
   테스트 통과시키려고 torque/velocity/current 리밋을 올리지 마라. 캘리브레이션 상수(서보 트림 예: PCA9685, 카메라 extrinsics, IMU offset)는 실물 리그에 맞춘 값 — 반올림/정리 금지.
3. **환경 분리**: Isaac Lab(Python 3.11)과 ROS 2(Jazzy, Python 3.12)를 **한 환경에 같이 설치하지 마라** — 충돌한다. 그래서 서브프로젝트가 분리돼 있다.
4. **의존성 추가**: import 고치겠다고 라이브 환경에서 즉석 `pip install` 금지 → 재현 불가 드리프트. 각 스택의 매니페스트(pyproject/pixi 등)에 추가하고 lockfile 재생성 후 커밋.
5. **생성 디렉토리 수정 금지** (읽기 전용): `build/ install/ log/ outputs/ multirun/ wandb/`. 여기 편집분은 다음 빌드/런에서 사라진다. ROS 코드는 `src/`에서만 편집.

## 재현성

- seed는 config에서 온다. 하나의 seed를 python `random`/numpy/torch(+cuda)/env에 적용하고 값 로깅.
- 런 하나 = 디렉토리 하나. resolved config 스냅샷(`.hydra/config.yaml` 또는 `params/`)이 "실제로 뭐가 돌았나"의 소스 오브 트루스.
- 런마다 git SHA + dirty 여부 + 해석된 config + 주요 버전(torch/cuda/isaac/ros distro)을 기록.
- GPU 완전 결정론은 기대하지 마라(비결정 CUDA 커널, async sim). 단일 seed 숫자를 재현성으로 보고하지 말고 N-seed 분산으로.

## 비밀/경로

- W&B 키, 로봇 IP, 크레덴셜은 커밋된 yaml이 아니라 `.env`(gitignore됨)에서 읽는다. `.env.example`가 필요한 변수 목록.
- 절대경로/사용자 데이터 디렉토리를 하드코딩하지 마라 — config/env에서 루트를 읽어라.

## 스킬 설치

- `.claude/skills/`에 넣으면 이 repo에 travel한다. 전역 적용을 원하면 `~/.claude/skills/`로 복사.
- 로봇 학습 전용 skill은 실제 반복 작업(예: 실험 런처)이 생긴 뒤 추가 — 지금 만드는 건 speculative.

## 환경 세팅 (add-when: 실제로 환경 붙일 때)

- Isaac / ROS 구체 명령은 각 스택 CLAUDE.md 참고.
- 재현성 락파일 도구(pixi/uv)는 환경 확정 시 도입. 솔로/초기 단계엔 gitignore + 체크섬 다운로드 스크립트로 충분(과한 DVC+MLflow 지양).
