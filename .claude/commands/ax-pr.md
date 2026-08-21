---
description: 이슈 단위로 PR을 올린다 (8단계)
argument-hint: <project> <issue>
allowed-tools: Bash(git:*), Bash(gh:*), Read, Write
---

`$1` 이슈 `$2`의 PR을 올린다.

## 전제

`stage`가 `documented`여야 한다. 게이트를 건너뛴 게 있으면 무엇이 빠졌는지 알리고 멈춘다.

**브랜치 1개 = 이슈 1개 = PR 1개.** 2번에서 이미 이슈 단위로 갈라졌으므로 여기서 더 쪼갤 것은 없다.

## 푸시

브랜치를 push한다. 커밋되지 않은 변경이 있으면 먼저 알린다.

## 생성

```
gh pr create --repo <slug> --base <base> \
  --title "[<개발항목번호>] <제목>" --body <본문>
```

본문 구성:

- **무엇을** — spec의 목적 3줄 이내
- **수용기준** — AC ID별 체크리스트와 verify 판정
- **검증** — 6-A/B/C/D 각각의 결과 요약. 보류나 UNVERIFIABLE이 있으면 숨기지 말고 적는다
- **문서** — 7번 안내서 링크
- **짝 이슈** — FE·BE 짝이 있으면 상대 PR 링크. 머지 순서를 명시한다
- `Closes #<번호>`

## 마무리

PR URL을 상태 파일에 적고 `stage`를 `pr`로 올린다.

보드 카드 이동은 지금 하지 않는다 — 나중에 붙인다. 사람이 손으로 옮겨야 한다는 것만 알린다.
