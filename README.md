# robot-learning-base

로봇 학습(robot learning) 프로젝트를 올려두는 베이스 monorepo. 두 스택을 **각자 독립 환경**으로 나란히 두고, 코딩·문서 작업용 로컬 스킬을 함께 번들한다.

```
.
├── CLAUDE.md            # 루트 규칙 (Opus 4.8 튜닝 가이드라인 + 로봇학습 프로젝트 규칙)
├── .gitignore           # 체크포인트/데이터셋/rosbag/런 출력/colcon 생성물/무거운 스킬 에셋 제외
├── .env.example         # 필요한 환경변수(W&B 키, 로봇 IP 등) — 실제 .env는 gitignore
├── .claude/skills/      # 로컬 스킬 29개 (repo와 함께 이동)
├── isaac/               # Isaac Lab / Isaac Sim RL 스택  (Python 3.11)  → isaac/CLAUDE.md
├── ros2_ws/             # ROS 2 + MuJoCo 인터페이스 스택 (Jazzy, 3.12)   → ros2_ws/CLAUDE.md
│   └── src/             # colcon 패키지가 여기 (build/install/log는 생성물, 커밋 X)
└── data/                # 데이터셋 blob은 gitignore, DVC/HF 포인터만 커밋
```

## 왜 두 스택이 분리돼 있나

Isaac Lab(Python 3.11)과 ROS 2(Jazzy, Python 3.12)는 **한 환경에 같이 설치하면 충돌**한다. 그래서 서브프로젝트로 나누고, 각 폴더의 `CLAUDE.md`가 그 스택의 정확한 명령어와 관례를 담는다. 가장 구체적인 CLAUDE.md가 이긴다.

---

# 스킬 사용법

`.claude/skills/`의 스킬은 이 repo와 함께 이동한다(전역으로 쓰려면 `~/.claude/skills/`로 복사). 스킬은 **대화 중 요청이 각 `SKILL.md`의 description과 맞으면 자동 로드**된다 — 별도 설치 명령이 필요 없다. 일부는 슬래시로도 부른다(예: `/ponytail-review`).

## 설치된 29개 한눈에

| 갈래 | 스킬 |
|---|---|
| 코딩 규율 (7) | `karpathy-guidelines`, `ponytail` + `-audit`/`-debt`/`-gain`/`-help`/`-review` |
| 개발 워크플로우 · superpowers (14) | `brainstorming`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `using-git-worktrees`, `test-driven-development`, `systematic-debugging`, `verification-before-completion`, `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch`, `using-superpowers`, `writing-skills` |
| UI/UX 디자인 · ui-ux-pro-max (7) | `ui-ux-pro-max`, `design`, `design-system`, `brand`, `ui-styling`, `banner-design`, `slides` |
| 문서 조판 (1) | `kami` |

## kami — 문서/PDF 조판 (EN·KO 전용)

전문 문서를 parchment 배경 + ink-blue 액센트 + 세리프 중심으로 조판한다. **아래처럼 요청하면 자동 트리거**:

- 한국어: `이력서 만들어줘` · `원페이저 만들어줘` · `이 리서치를 장문 문서로 정리해줘` · `발표용 슬라이드 만들어줘` · `앱 랜딩 페이지 만들어줘` · `정식 레터 작성해줘` · `프로젝트 포트폴리오 만들어줘`
- English: `build me a resume` · `make a one-pager` · `turn this into a PDF` · `design a slide deck` · `build a landing page` · `make this presentable`

**템플릿 9종**: One-Pager · Long Doc · Letter · Portfolio · Resume · Slides · Equity Report · Changelog · Landing Page. + 인라인 SVG 다이어그램 14종(architecture/flowchart/bar/line/donut/timeline/…).

**출력 경로**: 기본은 WeasyPrint로 HTML→PDF. 슬라이드는 요청 시 PPTX(python-pptx) 또는 Marp(마크다운) 경로도 지원.

**로컬 의존성** (문서를 실제로 렌더할 때):
```bash
pip install weasyprint          # 필수 — HTML→PDF
pip install Pygments            # 선택 — 코드 블록 하이라이트
pip install python-pptx         # 선택 — 편집 가능한 PPTX 슬라이드
# Marp 슬라이드는 로컬 marp CLI 필요(번들 안 됨)
```

**빌드/검증** (kami가 내부적으로 쓰는 명령):
```bash
K=.claude/skills/kami
python3 $K/scripts/build.py --verify <target>            # 렌더 + 폰트 임베딩 + 페이지수 체크
python3 $K/scripts/build.py --check-placeholders <filled.html>   # {{...}} 잔여 확인
python3 $K/scripts/build.py --check-resume-balance <resume.pdf>  # 이력서 2p 균형 체크
```

**폰트 (EN/KO 전용으로 트림함)**:
- EN: **Charter** (`@font-face`; 없으면 Georgia fallback) · 코드: JetBrains Mono
- KO: **Source Han Serif KR** (`assets/fonts/`, 로컬 전용)
- CN(TsangerJinKai 38M)·JA(YuMincho) 폰트와 타 언어 데모/index는 **제거해 경량화**했다.
- 폰트는 용량 때문에 **git에 없음**. 새 머신에선 `bash .claude/skills/kami/scripts/ensure-fonts.sh`로 받거나, `@font-face`에 박힌 jsDelivr fallback으로 온라인 렌더된다.

**디자인 불변식**(kami invariants): 배경 parchment `#f5f4ed`(순백 금지) · 액센트 ink-blue `#1B365D` 하나만 · 회색은 전부 warm-toned · 페이지당 세리프 1종, weight 500 고정(볼드 없음). 전체 스펙은 `.claude/skills/kami/CHEATSHEET.md`.

## 그 외 스킬 (상황별 자동 트리거)

- **코딩 규율**: 코드를 쓰거나 리뷰할 때 `karpathy-guidelines`·`ponytail`이 배경으로 작동(최소 구현·수술적 변경). 명시 호출: `/ponytail-review`(오버엔지니어링 리뷰), `/ponytail-audit`(repo 전체 감사), `/ponytail-debt`(`ponytail:` 주석 부채 수집).
- **개발 워크플로우(superpowers)**: `brainstorming`(창작/기능 작업 **전**), `writing-plans`→`executing-plans`(멀티스텝 계획·실행), `test-driven-development`(구현 전 테스트), `systematic-debugging`(버그/실패 시), `using-git-worktrees`(격리 작업), `requesting-code-review`/`receiving-code-review`(리뷰).
- **UI/UX(ui-ux-pro-max)**: 웹/모바일 UI·디자인 요청 시. `design`(브랜드·토큰·로고 종합), `ui-styling`(shadcn/ui + Tailwind), `design-system`(토큰·컴포넌트), `slides`(HTML 프레젠테이션), `banner-design`.

---

## 시작하기 (로봇 스택)

각 스택의 `CLAUDE.md`부터 읽기. 새 프로젝트는 거기 명령어대로:
- Isaac: `./isaaclab.sh --new` 제너레이터로 `isaac/` 안에 태스크 생성.
- ROS 2: `ros2_ws/src/`에 패키지 만들고 `colcon build --symlink-install`.

## 스택 참고 (조사 기준일: 2026-07)

Isaac Lab 2.3.x / Isaac Sim 5.1 / PyTorch 2.7+CUDA 12.8 · ROS 2 Jazzy + MuJoCo 3.x + `mujoco_ros2_control`.
버전 조합은 fragile하니 함부로 올리지 말 것.
