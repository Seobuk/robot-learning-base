# robot-learning-base

Claude Code로 **로봇 학습(robot learning) 프로젝트를 시작할 때 쓰는 베이스 템플릿**. 개발 규칙(`CLAUDE.md`)과 작업을 돕는 로컬 스킬 30개가 미리 세팅돼 있어서, 이 폴더 안에서 바로 Isaac Lab이나 ROS 2/MuJoCo 프로젝트를 만들면 된다.

## 이게 뭔가요?

빈 폴더에서 로봇 학습 프로젝트를 시작할 때마다 (1) 폴더 구조 잡고 (2) "체크포인트 커밋하지 마" 같은 규칙 적고 (3) 유용한 Claude 스킬 까는 일을 반복하게 된다. 이 repo는 그걸 한 번 해둔 **출발점**이다:

- **`CLAUDE.md`** — Claude Code가 이 프로젝트에서 지킬 규칙(커밋 금지·하드웨어 안전·환경 분리·재현성). Opus 4.8에 맞춰 튜닝됨.
- **스킬 30개** — Claude Code에서 자동으로 작동하는 도우미(코드 규율, 개발 워크플로우, UI/디자인, 문서 조판, 프로젝트 온보딩).
- **두 스택 골격** — `isaac/`(Isaac Lab RL)와 `ros2_ws/`(ROS 2 + MuJoCo)가 충돌 없이 나란히.

> 로봇 학습을 안 하더라도, 30개 스킬 + 개발 규칙을 갖춘 **범용 Claude Code 작업 베이스**로 그냥 써도 된다.

## 필요한 것

- **[Claude Code](https://claude.com/claude-code)** — 스킬과 `CLAUDE.md`가 여기서 작동한다. · **git**
- 실제로 학습을 *돌릴* 때만 필요: Isaac은 NVIDIA GPU + Isaac Sim, ROS 2는 Ubuntu(Jazzy = 24.04). 자세한 건 각 스택 `CLAUDE.md`에. **지금 시작하는 데는 필요 없다.**

## 빠르게 시작하기

**1. repo 받기** — clone 하거나 GitHub "Use this template"로 새 repo를 만든다:

```bash
git clone https://github.com/Seobuk/robot-learning-base.git my-robot-project
cd my-robot-project
```

**2. Claude Code로 이 폴더 열기** — 여는 순간 `.claude/skills/`의 스킬 30개와 루트 `CLAUDE.md` 규칙이 자동 적용된다. 따로 설치할 게 없다.

**3. 스택 고르기** — 만들 프로젝트에 맞게 (자세한 명령은 해당 폴더의 `CLAUDE.md`):

- **Isaac Lab RL**: `isaac/` 안에서 `./isaaclab.sh --new` 제너레이터로 태스크 생성 → `isaac/CLAUDE.md`
- **ROS 2 + MuJoCo**: `ros2_ws/src/`에 패키지 만들고 `colcon build --symlink-install` → `ros2_ws/CLAUDE.md`

**4. 환경변수** — `.env.example`를 `.env`로 복사하고 값(W&B 키·로봇 IP 등)을 채운다. `.env`는 git에 올라가지 않는다.

## 꼭 알아둘 규칙 3가지

이 프로젝트는 Claude가 아래를 지키도록 설정돼 있다 (전체는 `CLAUDE.md`):

1. **커밋하지 않는 것** — 체크포인트/가중치, 데이터셋, rosbag, 런 출력. 이미 `.gitignore`에 있다. `git add -A` 대신 `git status`로 확인하고 스테이징.
2. **기본값은 시뮬레이션** — `ros2_ws`의 노드는 실제 로봇을 움직일 수 있다. "실기"라고 명시하기 전엔 시뮬레이션(`use_sim_time:=true`). 테스트 통과하려고 torque 리밋·캘리브레이션 상수를 임의로 바꾸지 않는다.
3. **Isaac ↔ ROS 2 환경 분리** — Isaac Lab(Python 3.11)과 ROS 2(Jazzy, 3.12)는 한 환경에 같이 설치하면 충돌한다. 그래서 폴더가 나뉘어 있다.

## 폴더 둘러보기

```
.
├── CLAUDE.md            # 루트 규칙 (Opus 4.8 튜닝 가이드라인 + 로봇학습 규칙)
├── .gitignore           # 체크포인트/데이터셋/rosbag/런 출력/colcon 생성물 제외
├── .env.example         # 필요한 환경변수 목록 — 실제 .env는 gitignore
├── .claude/skills/      # 로컬 스킬 30개 (repo와 함께 이동)
├── isaac/               # Isaac Lab / Isaac Sim RL 스택  (Python 3.11)  → isaac/CLAUDE.md
├── ros2_ws/             # ROS 2 + MuJoCo 인터페이스 스택 (Jazzy, 3.12)   → ros2_ws/CLAUDE.md
│   └── src/             # colcon 패키지가 여기 (build/install/log는 생성물, 커밋 X)
└── data/                # 데이터셋 blob은 gitignore, DVC/HF 포인터만 커밋
```

---

# 번들된 스킬 30개

`.claude/skills/`의 스킬은 이 repo와 함께 이동한다(전역으로 쓰려면 `~/.claude/skills/`로 복사). **대화 중 요청이 각 `SKILL.md`의 설명과 맞으면 자동 로드**되므로 따로 설치·호출할 필요가 없다. 일부는 슬래시로도 부른다(예: `/ponytail-review`).

| 갈래 | 스킬 |
|---|---|
| 코딩 규율 (7) | `karpathy-guidelines`, `ponytail` + `-audit`/`-debt`/`-gain`/`-help`/`-review` |
| 개발 워크플로우 · superpowers (14) | `brainstorming`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `using-git-worktrees`, `test-driven-development`, `systematic-debugging`, `verification-before-completion`, `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch`, `using-superpowers`, `writing-skills` |
| UI/UX 디자인 · ui-ux-pro-max (7) | `ui-ux-pro-max`, `design`, `design-system`, `brand`, `ui-styling`, `banner-design`, `slides` |
| 문서 조판 (1) | `kami` |
| 프로젝트 온보딩 (1) | `setup` |

## 언제 무엇이 뜨나 (상황별)

- **코딩 규율** — 코드를 쓰거나 리뷰할 때 `karpathy-guidelines`·`ponytail`이 배경으로 작동(최소 구현·수술적 변경). 직접 호출: `/ponytail-review`(오버엔지니어링 리뷰), `/ponytail-audit`(repo 감사).
- **개발 워크플로우(superpowers)** — `brainstorming`(작업 **전** 탐색), `writing-plans`→`executing-plans`(계획·실행), `test-driven-development`(구현 전 테스트), `systematic-debugging`(버그·실패 시), `using-git-worktrees`(격리 작업), `requesting-code-review`/`receiving-code-review`(리뷰).
- **UI/UX(ui-ux-pro-max)** — 웹/모바일 UI·디자인 요청 시. `design`, `ui-styling`(shadcn/ui + Tailwind), `slides`(HTML 프레젠테이션) 등.
- **프로젝트 온보딩(setup)** — 기존 코드를 넣고 `이 코드 세팅해줘`처럼 요청하면 트리거. 의존성 설치 → 코드 분석 → **개발환경 가이드 + 메모리 MD + 코드에 맞는 CLAUDE.md** 생성. 설치·파일 생성/덮어쓰기 등 **변경 전엔 항상 먼저 물어본다**(기존 CLAUDE.md는 clobber 없이 merge).

## kami — 문서/PDF 조판 (EN·KO)

전문 문서를 parchment 배경 + ink-blue 액센트 + 세리프 중심으로 조판한다. 아래처럼 요청하면 자동 트리거:

- 한국어: `이력서 만들어줘` · `원페이저 만들어줘` · `이 리서치를 장문 문서로 정리해줘` · `발표용 슬라이드 만들어줘` · `앱 랜딩 페이지 만들어줘`
- English: `build me a resume` · `make a one-pager` · `turn this into a PDF` · `design a slide deck`

**템플릿 9종**(One-Pager·Long Doc·Letter·Portfolio·Resume·Slides·Equity Report·Changelog·Landing Page) + SVG 다이어그램 14종. 출력은 기본 WeasyPrint(HTML→PDF), 슬라이드는 PPTX·Marp도 가능.

문서를 실제로 렌더할 때 로컬 의존성:

```bash
pip install weasyprint     # 필수 — HTML→PDF
pip install Pygments       # 선택 — 코드 하이라이트
pip install python-pptx    # 선택 — 편집 가능한 PPTX
```

폰트는 EN(**Charter**)·KO(**Source Han Serif KR**) 전용으로 트림했고, 용량 때문에 git에는 없다 — 새 머신에선 `bash .claude/skills/kami/scripts/ensure-fonts.sh`로 받거나 `@font-face`의 jsDelivr fallback으로 온라인 렌더된다. 전체 스펙: `.claude/skills/kami/CHEATSHEET.md`.

---

# 더 알아보기

## 왜 두 스택이 분리돼 있나

Isaac Lab(Python 3.11)과 ROS 2(Jazzy, Python 3.12)는 한 환경에 같이 설치하면 충돌한다. 그래서 서브프로젝트로 나누고, 각 폴더의 `CLAUDE.md`가 그 스택의 정확한 명령어와 관례를 담는다. **가장 구체적인 `CLAUDE.md`가 이긴다.**

## 스택 버전 참고 (조사 기준일: 2026-07)

Isaac Lab 2.3.x / Isaac Sim 5.1 / PyTorch 2.7 + CUDA 12.8 · ROS 2 Jazzy + MuJoCo 3.x + `mujoco_ros2_control`.
버전 조합은 fragile하니 함부로 올리지 말 것.
