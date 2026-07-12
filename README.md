# robot-learning-base

로봇 학습(robot learning) 프로젝트를 올려두는 베이스 monorepo. 두 스택을 **각자 독립 환경**으로 나란히 둔다.

```
.
├── CLAUDE.md            # 루트 규칙 (커밋 금지 목록, 하드웨어 안전, 환경 분리, 재현성)
├── .gitignore           # 체크포인트/데이터셋/rosbag/런 출력/colcon 생성물 전부 제외
├── .env.example         # 필요한 환경변수(W&B 키, 로봇 IP 등) — 실제 .env는 gitignore
├── .claude/skills/
│   └── karpathy-guidelines/   # 설치된 행동 가이드라인 스킬
├── isaac/               # Isaac Lab / Isaac Sim RL 스택  (Python 3.11)  → isaac/CLAUDE.md
├── ros2_ws/             # ROS 2 + MuJoCo 인터페이스 스택 (Jazzy, 3.12)   → ros2_ws/CLAUDE.md
│   └── src/             # colcon 패키지가 여기 (build/install/log는 생성물, 커밋 X)
└── data/                # 데이터셋 blob은 gitignore, DVC/HF 포인터만 커밋
```

## 왜 두 스택이 분리돼 있나

Isaac Lab(Python 3.11)과 ROS 2(Jazzy, Python 3.12)는 **한 환경에 같이 설치하면 충돌**한다. 그래서 서브프로젝트로 나누고, 각 폴더의 `CLAUDE.md`가 그 스택의 정확한 명령어와 관례를 담는다. 가장 구체적인 CLAUDE.md가 이긴다.

## 스킬

`.claude/skills/`의 스킬은 이 repo와 함께 이동한다. 전역으로 쓰려면 `~/.claude/skills/`로 복사.
현재 설치: **karpathy-guidelines** (assumptions 명시 · 최소 구현 · 수술적 변경 · 검증 가능한 목표).

## 시작하기

각 스택의 `CLAUDE.md`부터 읽기. 새 프로젝트는 거기 명령어대로:
- Isaac: `./isaaclab.sh --new` 제너레이터로 `isaac/` 안에 태스크 생성.
- ROS 2: `ros2_ws/src/`에 패키지 만들고 `colcon build --symlink-install`.

## 스택 참고 (조사 기준일: 2026-07)

Isaac Lab 2.3.x / Isaac Sim 5.1 / PyTorch 2.7+CUDA 12.8 · ROS 2 Jazzy + MuJoCo 3.x + `mujoco_ros2_control`.
버전 조합은 fragile하니 함부로 올리지 말 것.
