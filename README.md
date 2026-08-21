# AX_PIPELINE

AX팀 스펙 기반 개발(SDD) 파이프라인.

이 리포에는 **도구만** 있다. 제품 코드도, 특정 프로젝트 설정도 없다.
`projects.yml`에 대상을 등록하면 그 프로젝트에 붙는다.

흐름 정의는 **[PIPELINE.md](PIPELINE.md)** 를 본다.

## 무엇을 하는가

이슈 하나가 개발항목 하나이고, 그 번호가 spec · plan · 검증 · 문서 · PR까지 그대로 이어진다.

```
개발항목번호 → 이슈 → 브랜치 → spec.md → plan.md
    → verify.md → 문서 태그 → PR 제목
```

문서만 보고도 어느 스펙에서 나온 화면인지 역추적된다.

## 준비

**1. 프로젝트를 등록한다** — `projects.yml`의 `projects:` 아래에 항목을 추가한다. 파일 안에 스키마와 예시가 있다.

등록 전에 실물을 확인한다. 둘 다 자주 틀린다.

```bash
gh repo view <owner>/<repo> --json defaultBranchRef   # base가 main인가 master인가
git branch -r                                          # 브랜치 명명 관례가 이미 있는가
```

**2. `gh` 인증을 확인한다** — `gh auth status`. `repo` 스코프가 필요하다.

## 사용

```
/ax-sync   <project>          1.  이슈 수집
/ax-branch <project> be#N     2.  브랜치
/ax-prd    <project>          3a. Total PRD
/ax-spec   <project> be#N     3b. 스펙        ← 게이트
/ax-plan   <project> be#N     4.  계획        ← 게이트
/ax-build  <project> be#N     5.  구현
/ax-verify <project> be#N     6.  검증        ← 게이트
/ax-doc    <project> be#N     7.  문서화      ← 게이트
/ax-pr     <project> be#N     8.  PR
```

6번 검증은 네 갈래로 갈린다.

| | 대상 | 방법 |
|---|---|---|
| 6-A | 전체 | 빌드 · 테스트 · 린트 |
| 6-B | 전체 | `claude -p` 헤드리스로 spec 대조 |
| 6-C | FE | Chrome 확장으로 화면 직접 조작 + 캡처 |
| 6-D | BE · 화면 없는 항목 | API 계약 검증 |

6-C가 찍은 캡처가 7번 문서의 자료가 된다. 문서용으로 다시 찍지 않는다.

## 구성

| 경로 | 내용 |
|---|---|
| `PIPELINE.md` | 흐름 정의 |
| `projects.yml` | 대상 프로젝트 레지스트리 |
| `.claude/commands/` | 단계별 커맨드 9개 |
| `.claude/skills/ax-doc-format/` | 7단계 문서 규격 + HTML 템플릿 |
| `templates/` | spec · plan · verify · total-prd |
| `.ax/state/` | 이슈별 진행 상태 (git 제외) |
| `workspace/` | 대상 리포 클론 위치 (git 제외) |

## 원칙

- **plan 승인 전에는 코드를 쓰지 않는다.** `/ax-build`는 미승인이면 멈춘다
- **검증은 코드를 쓴 컨텍스트가 하지 않는다.** 6-B는 별도 프로세스를 띄운다
- **개발항목 번호를 채번하지 않는다.** 팀이 쓰던 번호를 그대로 수입한다
- **기존 관례를 이긴다고 생각하지 않는다.** 브랜치 명명·기본 브랜치·CI 게이트가 이미 있으면 그것을 따른다
- **진행 상태의 진실은 `.ax/state`다.** 보드가 아니다

## 아직 안 붙인 것

- **프로젝트 보드 연동** — 카드 상태 자동 이동. 나중에 커맨드 끝에 `gh project item-edit` 한 줄로 붙는다
- **6-A 커맨드 자동 탐지** — 지금은 `projects.yml`의 `verify.build/test/lint`에 직접 적어야 한다
