---
name: sppt-product-parse
description: |
  주문의 SourceOffering을 안전하게 해석해 기존 ProductOption 매핑을 제안·적용하고,
  product master와 product mapping 후속 작업까지 연결하는 워크플로우.
  Use when asked to "상품명 분석", "SKU별 정리", "미매핑 상품 확인", "매핑 설정".
---

# product-parse

상품 업무는 다음 세 영역으로 나눠 처리합니다.

| 업무 | 책임 | 대표 명령 |
| --- | --- | --- |
| `product parse` | intake·후보·기존 옵션 매핑 | `sppt product parse` |
| `product master` | 마스터 상품·옵션·분류 관리 | `sppt product master ...` |
| `product mapping` | 미매핑·연결·이력·정책 관리 | `sppt product mapping ...` |

`parse`는 마스터 CRUD나 매핑 이력 관리 명령이 아닙니다. 새 마스터나 분류를 만들어야
하거나 옵션 BOM을 보완해야 하는 경우에는 자동 확정하지 않고 검토 대기 상태로 남긴 뒤,
`product master`에서 확정하고 `product mapping`에서 연결합니다.

## 시작 전 확인

- 주문이 임포트되어 있거나 채널 API 동기화가 완료되어 있어야 합니다.
- 클라우드 인증을 확인합니다.

```bash
sppt auth whoami
```

- 처리할 채널·계정·상태·cursor/watermark 범위를 분명히 합니다. 범위가 불명확하면 전체를
  재처리하지 말고 먼저 대상 건수와 scope를 확인합니다.

## 1. product parse: 자동 intake

### 명령

```bash
# 기본 동작은 preview run입니다. 마스터·판매채널 매핑은 변경하지 않지만
# 실행 기록·snapshot·후보·검토 상태는 저장합니다.
sppt product parse
sppt product parse --dry-run

# 권장 순서: preview 결과를 확인한 뒤 같은 run을 명시해 적용합니다.
sppt product parse status <runId>
sppt product parse --apply --confirm --run-id <runId>

# 새 run 생성부터 적용까지 한 번에 수행하는 사전 승인된 자동 실행
sppt product parse --apply --confirm

# 실행 결과·pending 항목·적용/충돌 상태 확인
sppt product parse status <runId>
```

### 처리 순서

1. 지정한 source/channel account와 state 범위의 SourceOffering을 읽습니다. 기본 state는
   `unprocessed`이며, 미매핑만 처리하려면 `--state unmapped`를 지정합니다.
2. `product mapping rules publish`로 활성화된 compiled `product_rules` bundle을 source,
   policy, master snapshot과 함께 고정합니다.
3. 상품명·옵션명·raw SKU를 정규화해 기존 마스터 옵션 후보를 찾습니다.
4. raw SKU 또는 명시 정책으로 하나의 active ProductOption이 확정되고 유효한 SKU/BOM,
   revision 조건을 만족하는 경우에만 자동 적용 대상으로 삼습니다.
5. 모호한 identity, SKU 부족, BOM 미완성, 분류 충돌은 `product mapping unmapped`의
   durable review case로 남깁니다.
6. 현재 매핑과 같은 대상이면 `unchanged`로 기록하고 매핑을 다시 쓰지 않습니다. 다른
   대상이 더 강하게 검출되면 `mapping_change` 후보로 만들지만 preview 단계에서는 변경하지
   않습니다.
7. 최종 적용이 승인된 매핑 변경은 서버가 하나의 `changeSetId`로 묶어 이력에 기록합니다.

기본 parse와 `--dry-run`은 마스터 상품·옵션·분류와 판매채널 매핑을 변경하지 않습니다. 대신
run, input manifest, 후보, action plan, review case와 SourceOffering의 parse 상태를 저장할 수
있습니다. `--apply --confirm` 없이는 매핑을 적용하지 않습니다. 최종 매핑 전에 확인을 받으려면
반드시 `parse → status → parse --apply --confirm --run-id` 순서를 사용합니다. `--run-id`를
지정하면 해당 run의 input manifest hash, policy hash, source mapping revision을 다시 검증하고
snapshot이나 revision이 달라지면 중단합니다. `--run-id`를 생략한 `--apply --confirm`은 새 run을
만들고 결과를 기다린 뒤 같은 명령 안에서 적용하므로, 별도 review가 끝난 뒤 적용해야 하는 경우에는
사용하지 않습니다.

신규 마스터 상품·ProductOption·classification node 생성, 마스터 field 수정, BOM 구성
변경은 parse가 자동 적용하지 않습니다. 이 결과는 pending review로 남기고 다음 순서로
처리합니다.

```bash
sppt product master create --name "알텐바흐 316Ti 멀티쿠커"
sppt product master option create <productId> \
  --name "20cm" --option-type variant --sku-id <skuId>
sppt product master classification create --kind category --name "냄비"
sppt product mapping map <sourceOfferingId> --product-option-id <masterOptionId>
```

`status`는 run 상태와 pending 사유를 확인하는 읽기 전용 명령입니다. parse 결과를 임의의
후보 선택으로 확정하지 말고, 신규 master·classification·BOM이 필요한지 먼저 판단합니다.

### 이미 매핑된 항목의 처리

- 현재 매핑 대상과 parse 후보가 같으면 `unchanged`로 처리합니다. 기존 decision과 매핑을
  덮어쓰지 않습니다.
- 현재 매핑과 다른 대상이 더 강하게 검출되면 `mapping_change`로 표시합니다. preview에서는
  기존 매핑을 유지하며, `status`에서 대상과 근거를 확인한 뒤 `--apply --confirm --run-id`로
  명시적으로 승인해야 합니다.
- 적용 조건은 신규 매핑과 동일하게 canonical identity, 동일한 mapping revision, active
  ProductOption, 실행 가능한 SKU/BOM입니다. 조건이 맞지 않거나 판단이 모호하면 기존 매핑을
  유지하고 conflict 또는 review case로 남깁니다.
- 재매핑이 적용되면 기존 decision을 삭제하지 않고 supersedes 관계의 새 decision과 새
  `changeSetId`를 기록합니다. 잘못된 재매핑은 `product mapping rollback`으로 되돌립니다.

## 2. product master: 확정 데이터 관리

마스터 상품과 옵션은 이 영역에서 명시적으로 생성·수정합니다. 생성·수정에는 optimistic
revision과 idempotency를 사용합니다. 옵션 create/update CLI는 현재 product revision과 옵션
revision을 서버에서 읽어 내부적으로 configuration preview를 만든 뒤 즉시 적용합니다. 별도
review 후 confirm하는 최종 매핑 명령과는 다른 동작입니다.

```bash
sppt product master list --query "멀티쿠커"
sppt product master create --name "알텐바흐 316Ti 멀티쿠커" --category-id <categoryId>
sppt product master update <productId> --revision <n> --name "알텐바흐 316Ti 멀티쿠커 2.0"

sppt product master option list <productId>
sppt product master option create <productId> \
  --name "20cm" --option-type variant --sku-id <skuId>
sppt product master option update <productId> <optionId> \
  --name "20cm 본품"
```

### 분류 관리

분류는 별도 최상위 업무가 아니라 `product master` 아래에서 관리합니다. 지원 kind는
`brand`, `category`, `productLine`입니다.

```bash
sppt product master classification list --kind category
sppt product master classification create --kind category --name "냄비"
sppt product master classification update --kind category <id> --revision <n> \
  --name "냄비·웍"
sppt product master classification archive --kind category <id> --revision <n>
sppt product master classification restore --kind category <id> --revision <n>
```

상품 분류 연결이 빠져 있으면 매핑과 별개로 분류 연결 필요 review issue로 남깁니다. parse는
분류 후보만 pending으로 제안하며, 분류 디렉터리와 상품 연결은 master에서 확정합니다.

### 옵션과 BOM

단품은 direct SKU를 사용하고, 세트·프로모션은 옵션의 BOM 구성으로 관리합니다. 구성 변경은
현재 active 옵션을 보존한 전체 configuration preview를 확인한 뒤 적용합니다. parse가
BOM을 추측하거나 불완전한 BOM을 자동 확정하게 두지 않습니다.

## 3. product mapping: 연결·이력·정책

### 미매핑과 직접 연결

```bash
sppt product mapping unmapped

# 첫 호출은 preview만 반환하며 변경하지 않습니다.
sppt product mapping map <sourceOfferingId> --product-option-id <masterOptionId>

# preview 결과와 대상 revision을 확인한 뒤 같은 요청에 confirm을 붙입니다.
sppt product mapping map <sourceOfferingId> \
  --product-option-id <masterOptionId> --confirm

# 해제도 동일하게 preview 후 confirm합니다.
sppt product mapping unmap <sourceOfferingId>
sppt product mapping unmap <sourceOfferingId> --confirm
```

map/unmap은 단건이어도 preview token과 대상 mapping revision을 검증합니다. `--confirm` 없는
호출은 preview만 반환하고 변경하지 않으며, 확인 시점에 source가 바뀌었거나 preview binding이
없으면 다시 preview를 만들고 중단합니다. 대상 ProductOption이 없으면 먼저 `product master`에서
생성·확정합니다.

### history와 rollback

```bash
sppt product mapping history --from "2026-09-02 10:00" --to "2026-09-02 10:10"

# changeSetId를 명시하는 경우에도 먼저 영향 범위를 확인합니다.
sppt product mapping rollback --changeset <changeSetId>
sppt product mapping rollback --changeset <changeSetId> --confirm

# timestamp는 history에서 대상을 고르는 편의 selector입니다.
sppt product mapping rollback --at "2026-09-02 10:05"
sppt product mapping rollback --at "2026-09-02 10:05" --confirm
```

사람이 새 `changeSetId`를 만들거나 배정하지 않습니다. rollback에서는 서버가 만든 기존 ID를
선택할 수 있지만, map, unmap, parse 자동 적용, rollback 실행 자체의 `changeSetId`는 서버가
자동 생성합니다. `--at`은 해당
시점의 history를 찾는 selector일 뿐이며, 실제 rollback 단위는 서버가 선택한 구체적인
change set입니다. 시간만으로 대상이 모호하거나 현재 revision이 달라지면 자동 rollback하지
않고 preview 또는 conflict를 반환합니다. rollback은 기존 이력을 삭제하지 않고 보정 decision과
새 change set을 추가합니다.

### product_rules 정책 authoring

`product_rules`는 사람이 작성·검토하는 Markdown authoring policy입니다. 아래 lifecycle로만
관리하고, parse는 publish된 정책 bundle을 snapshot에 고정해 사용합니다.

```bash
sppt product mapping rules show
sppt product mapping rules set ./product-rules.md
sppt product mapping rules publish
```

- `show`로 현재 draft와 active 정책을 확인합니다.
- `set`은 Markdown 원문을 draft로 저장합니다.
- `publish`는 compile·검증에 성공한 draft만 active 정책으로 교체합니다.
- 작성 가능한 규칙은 normalization/noise, alias, explicit SKU/source mapping, option·quantity
  pattern, classification suggestion 중심으로 제한합니다.
- 자유로운 설명은 검토 참고용 context로 보존할 수 있지만 canonical mapping을 단독으로
  결정하지 않습니다.
- 정책을 publish해도 기존 매핑을 자동 재작성하지 않습니다. 새 parse 실행과 별도 적용이
  필요합니다.

`set`의 Markdown은 정책 원문을 제출하는 입력이며 상품 마스터나 매핑 데이터를 직접 가져오는
임포트 경로가 아닙니다. 정책이 없거나 compile되지 않으면 이름 유사도만으로 확정하지 말고
`policy_not_applied` 또는 pending으로 남깁니다.

## 안전 원칙

1. parse는 항상 dry-run으로 시작하고, 실제 변경은 `--apply --confirm`을 함께 지정합니다.
2. map/unmap은 preview 결과·대상·현재 revision을 확인한 뒤 `--confirm`합니다.
3. raw SKU, canonical identity, 명시 정책으로 대상이 하나로 확정되지 않으면 선택하지 않습니다.
4. 신규 master, classification, option, BOM 변경은 pending review로 남기고 master에서 먼저
   확정합니다.
5. input manifest hash, policy hash, source mapping revision이 확인 시점과 다르면 재실행하거나
   충돌로 보고합니다.
6. 자동화가 생성한 `changeSetId`와 history를 보존하고, 기존 decision을 덮어쓰지 않습니다.
7. 적용 후 `sppt product parse status <runId>`와 `sppt product mapping unmapped`로 pending,
   conflict, projection 상태를 다시 확인합니다.
