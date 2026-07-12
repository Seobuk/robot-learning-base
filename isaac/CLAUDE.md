# isaac/ — Isaac Lab / Isaac Sim RL 스택

Isaac Lab 기반 강화학습. 커스텀 태스크는 **여기서** Isaac Lab 제너레이터로 만든다.

## 스택 핀 (현재 기준, 함부로 올리지 마라)

- Python **3.11** (필수 — venv Python이 Isaac Sim의 Python과 정확히 일치해야 import됨)
- Isaac Sim **5.1**, Isaac Lab **2.3.x** (Isaac Lab 2.2 = Isaac Sim 5.0)
- PyTorch **2.7** + CUDA **12.8** — CUDA 매칭 휠로 설치:
  `pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128`
- Linux GLIBC 2.35+ (Ubuntu 22.04+). Windows는 long-path 지원 켜기.

## 새 태스크 시작

- **제너레이터 사용**: `./isaaclab.sh --new` (Linux) / `isaaclab.bat --new` (Windows) → "external project" 선택, workflow(manager-based vs direct)와 RL 라이브러리(rsl_rl / skrl / rl_games / sb3) 선택.
- 구 `isaac-sim/IsaacLabExtensionTemplate` fork 금지 — 2025-09-26 archived/deprecated.
- manager-based(모듈형 obs/reward/termination cfg, 대부분 RL에 권장) vs direct(단일 `DirectRLEnv`, 오버헤드 적고 제어 세밀) 중 택1.

## 설치 / 실행

- 프로젝트는 editable pip 패키지: `python -m pip install -e source/<project_name>`
- 새 env는 태스크 `__init__.py`에서 `gym.register(...)` 해야 train/list_envs가 찾는다. 모듈 이름/위치 바꾸면 `pip install -e` 재실행.
- 태스크 목록: `python scripts/list_envs.py`
- 학습: `python scripts/<rl_lib>/train.py --task=<Task-Name> --headless` (학습 노드는 항상 `--headless`; GUI는 debug/play용)
- 재생/평가: `python scripts/<rl_lib>/play.py --task=<Task-Name> --checkpoint /path/to/model.*`

## 설정 / 로깅

- Hydra CLI override로 튜닝(스크립트 편집 X): env 파라미터 `env.<a>.<b>=value`, 알고리즘 `agent.<...>=value`.
  주의: 일부 cfg는 `__post_init__`에서 파생값을 미리 계산 — base 파라미터를 override해도 재계산 안 됨. 파생 필드를 직접 set 하거나 cfg를 편집.
- 태스크 하이퍼파라미터는 태스크의 `agents/` 폴더(rsl_rl=python `*_cfg.py`, skrl/rl_games/sb3=YAML), `gym.register(... kwargs=...)`로 연결.
- seed: rsl_rl/skrl/sb3 = `agent.seed=...`, rl_games = `agent.params.seed=...` (라이브러리마다 키 다름 — 복붙 주의).
- 로거 기본 TensorBoard(`tensorboard --logdir logs/<rl_lib>/<task>`), wandb/neptune는 opt-in.
- 런 디렉토리 `logs/<rl_lib>/<task>/<timestamp>/`, `params/`에 resolved `env.yaml`+`agent.yaml` 스냅샷.

## gitignore됨 (루트 .gitignore가 커버)

`logs/ outputs/ wandb/ *.egg-info/ __pycache__/ *.pt/*.pth/*.onnx/*.zip`, USD/Omniverse 캐시. 거대한 USD/asset 바이너리는 커밋하지 말고 Nucleus/asset-root 경로 참조.

## 흔한 실수

- Python 3.10을 Isaac Sim 5.x에 쓰기 → #1 설치 실패.
- torch를 기본 인덱스로 설치 → CPU-only/ABI 불일치. 반드시 CUDA 매칭 휠.
- editable 설치 잊고 모듈 옮긴 뒤 `gym.register` ID가 안 잡히는데 원인 못 찾기 → `pip install -e` 재실행.
