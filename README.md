# SDD 개발 워크플로우 하네스

프론트엔드 / 백엔드 / 모노레포를 지원하는 SDD(Spec-Driven Development) 기반 개발 워크플로우 하네스.

## 구성

```
.claude/
├── agents/          # 6개 역할
│   ├── planner.md   # 기획서 작성
│   ├── critic.md    # 기획 비평
│   ├── architect.md # plan → SDD 산출물 슬라이싱
│   ├── developer.md # TDD 구현
│   ├── reviewer.md  # 코드 품질·UX 리뷰
│   └── tester.md    # E2E + 보안 스캔
├── skills/          # 4개 스킬
│   ├── init/        # 프로젝트 초기화
│   ├── planning/    # /planning {기능} → plan.md
│   ├── spec/        # /spec {feature} → 슬라이스별 SDD 산출물
│   └── dev/         # /dev {slice} → 개발·리뷰·테스트·커밋
├── hooks/
│   └── pre_edit.sh  # Claude Edit/Write 시 ESLint 즉시 검사
└── settings.json    # PreToolUse 훅 등록

.claudeignore        # node_modules, build, env 등 차단
docs/
├── harness.md       # 훅 설계 참조 (init이 읽어 스택별 훅 설치)
└── workflow.md      # 워크플로우 개요
```

## 사용 흐름

```
/init                      → 프로젝트 타입 선택 · 스캐폴딩 · 훅 설치 · CLAUDE.md 생성
/planning 로그인           → docs/plans/login/plan.md (planner + critic 루프)
/spec login                → docs/specs/changes/{login-*}/ 슬라이스 4개 파일씩
/dev login-web             → 해당 슬라이스 개발·리뷰·테스트·sync·archive·commit
```

## 새 프로젝트에 적용

새 프로젝트 루트에 이 하네스를 복사:

```bash
cp -r /c/devel/harness/.claude NEW_PROJECT/
cp /c/devel/harness/.claudeignore NEW_PROJECT/
mkdir -p NEW_PROJECT/docs
cp /c/devel/harness/docs/harness.md NEW_PROJECT/docs/
cp /c/devel/harness/docs/workflow.md NEW_PROJECT/docs/
cd NEW_PROJECT
# Claude Code에서 /init 실행
```

## 지원 프로젝트 타입

- 단일 프론트엔드 (React / Vue / Next.js)
- 단일 백엔드 (Node / FastAPI / NestJS)
- 풀스택 단일 (Next.js App Router)
- 모노레포 (Turborepo / Nx / pnpm workspace)

## 설계 원칙

1. **역할 경계 칼날같이**: 리뷰어는 코드 품질·UX, 테스터는 시나리오·보안 — 중복 없음
2. **파일은 스킬에서만 읽음**: 에이전트는 변수만 받음. 중복 I/O 제거
3. **린트로 규칙 위임**: `no-console`, `no-explicit-any` 등은 ESLint가 강제
4. **재개 가능**: `/dev`는 tasks.md 체크박스와 `.status`로 중단 지점 복구
5. **모노레포 친화**: 아키텍트가 패키지별로 슬라이스 분리
