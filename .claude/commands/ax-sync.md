---
description: 대상 리포에서 이슈를 가져와 태스크 백로그로 만든다 (1단계)
argument-hint: <project>
allowed-tools: Bash(gh issue list:*), Bash(gh issue view:*), Read, Write
---

프로젝트 `$1`의 이슈를 수집한다.

## 1. 레지스트리 확인

`projects.yml`에서 `$1` 항목을 읽는다. 없으면 중단하고 등록 방법을 안내한다.

## 2. 이슈 조회

FE·BE 리포 각각에 대해:

```
gh issue list --repo <slug> --state open --limit 100 \
  --json number,title,body,milestone,assignees,url
```

`milestone`이 레지스트리의 값과 다른 이슈는 제외한다.

## 3. 파싱

이슈마다 아래를 뽑는다.

- **개발항목 번호** — 제목에 레지스트리 `issue.item_pattern`을 적용한다. `null`이면 번호 추적 없이 이슈번호만 쓴다. **채번하지 않는다** — 팀이 이미 쓰는 번호를 그대로 수입한다
- **분류** — 어느 리포에서 왔는가 (fe / be). 라벨이 비어 있는 팀이 많으므로 라벨에 기대지 않는다
- **짝 이슈** — 레지스트리 `issue.pair_rule`을 따른다. `fe_suffix_F`면 FE 항목번호 끝의 `F`를 떼어 BE 짝을 찾고, 양쪽 목록을 대조해 실제 이슈번호로 해석한다. `none`이면 건너뛴다
- **의존** — 본문 `**선행**:` 줄. 표기가 세 가지로 섞여 있으니 전부 처리한다.
  - `other_repo#11` — 타 리포 이슈 참조
  - `공통-1, 공통-2` — 하이픈 표기
  - `공통-1·2·4, 선택 1·4·5` — 가운뎃점 압축. **`선택 1`은 하이픈이 아니라 공백이다.** `(공통|선택)[-·\s]?(\d+)(·\d+)*` 로 받아 앞 접두어를 뒤 숫자에 전부 배분한다
  - `없음` — 선행 없음
  해석 결과는 실제 이슈번호로 바꿔 저장한다
- **완료 판정** — 본문 `**완료 판정**:` 줄. 3b 수용기준의 씨앗이다

## 4. 상태 기록

이슈마다 `.ax/state/$1/<번호>-<fe|be>.json`을 만든다. 이미 있으면 `stage`와 `gates`는 보존하고 메타만 갱신한다.

```json
{
  "project": "$1", "repo": "", "number": 0, "kind": "fe|be",
  "item": "선택-6", "title": "", "url": "",
  "branch": null, "pair": null, "depends": [],
  "done_hint": "이슈 본문의 완료 판정 원문",
  "stage": "synced",
  "gates": { "spec": null, "plan": null, "verify": null, "doc": null }
}
```

## 5. 출력

표로 보여준다: 개발항목 · 리포#번호 · 분류 · 짝 · 의존 · 현재 단계.

그리고 아래를 짚어준다.

- **완료 판정이 없거나 한 줄뿐인 이슈** — 3b에서 수용기준을 펼쳐야 한다
- **의존이 아직 착수 안 된 이슈** — 선행부터 해야 한다
- **짝을 못 찾은 FE 이슈** — 6-C 화면 검증을 단독으로 통과할 수 없다
