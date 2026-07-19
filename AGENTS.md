# AI 에이전트 가이드

이 레포는 칸반 스타일 작업 대시보드입니다. **모든 작업 데이터의 원본은 `tasks.json` 하나**이며,
`index.html`(GitHub Pages)이 이 파일을 읽고 씁니다. AI도 같은 파일을 읽고 쓰면 됩니다.

## 규칙

1. 작업 데이터를 읽거나 수정할 때는 `tasks.json`만 다룬다. (`TASKS.md`는 과거 초안, 참고용)
2. 수정 시 반드시:
   - 해당 task의 `updated_at`을 오늘 날짜(`YYYY-MM-DD`)로 갱신
   - `meta.updated_at`을 현재 시각(ISO 8601), `meta.updated_by`를 `"claude"` 등 에이전트 이름으로 갱신
3. `id`는 절대 변경하지 않는다. 새 작업 id 형식: `t-YYYYMMDD-<seq>-<임의4자>` (예: `t-20260719-008-a1b2`)
4. `status`를 `done`으로 바꾸면 `completed_at`을 채우고 `progress: 100`으로 설정. done에서 되돌리면 `completed_at: null`.
5. **AI가 실제로 작업을 대신 수행해도 되는 항목은 `ai.actionable === true` 이면서 `ai.enabled === true`인 것만.**
   `enabled`는 사용자가 웹 토글로 켜는 실행 허가 스위치다. `ai.instruction`에 지시 내용이 있다.
6. 커밋 메시지 형식: `chore: update tasks via <agent> (YYYY-MM-DD)`
7. 웹에서 동시에 수정될 수 있으므로, 커밋 전 최신 상태를 pull 하고 충돌 시 필드 단위로 병합한다.

## 스키마

```jsonc
{
  "schema_version": 1,
  "meta": {
    "title": "작업 대시보드",
    "updated_at": "ISO 8601 시각",   // 마지막 저장 시각
    "updated_by": "web | claude | ..."
  },
  "lists": [                          // Task Lists: 프로젝트/카테고리 그룹
    {
      "id": "l-xxx",
      "name": "리스트 이름",
      "color": "#2563eb",             // 카드 점 색상
      "visible": true,                // false면 보드에서 해당 리스트 작업 숨김
      "order": 0
    }
  ],
  "tasks": [
    {
      "id": "t-20260719-001",
      "title": "작업 제목",
      "status": "todo",               // todo | in_progress | done
      "list_id": "l-xxx",             // lists.id 참조, null 가능
      "priority": "high",             // high | medium | low
      "progress": 0,                  // 0~100
      "due": "2026-07-22",            // YYYY-MM-DD 또는 null
      "waiting_for": null,            // 대기 사유 문자열. 있으면 Waiting 칩 표시
      "tags": ["focus"],              // 자유 태그
      "notes": "자유 메모",
      "ai": {
        "actionable": false,          // AI가 처리 가능한 작업인지 (Actionable Lists에 표시)
        "enabled": false,             // 사용자가 AI 실행을 허용했는지 (웹 토글)
        "instruction": ""             // AI에게 전달할 지시
      },
      "created_at": "2026-07-19",
      "updated_at": "2026-07-19",
      "completed_at": null,           // done이 된 날짜
      "order": 0                      // 같은 status 컬럼 내 정렬 순서
    }
  ]
}
```

## 자주 쓰는 작업 예시

- **오늘 할 일 요약**: `status !== "done"` 작업을 `priority`, `due` 순으로 정렬해 보고
- **작업 추가**: `tasks`에 스키마대로 push, `order`는 해당 컬럼 마지막 번호 + 1
- **AI 위임 작업 수행**: `ai.actionable && ai.enabled`인 작업의 `instruction`을 수행하고,
  결과를 `notes`에 요약 추가 + `progress` 갱신 (완료 판단은 사용자에게 확인)
