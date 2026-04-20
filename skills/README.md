# shopport Skills

> Claude Code가 읽고 실행하는 커머스 운영 워크플로우 스킬

## 사용법

### Claude Code에서
```
/skill install shopport
```

또는 수동 설치:
```bash
git clone https://github.com/onlymytho/shopport ~/.claude/skills/shopport
```

### 스킬 목록

| 스킬 | 설명 |
|---|---|
| `source-to-register` | 소싱 검색 → 마진 계산 → 판매 채널 등록 |
| `order-fulfill` | 주문 확인 → 소싱 발주 → 송장 등록 |
| `inventory-sync` | 멀티채널 재고 동기화 |
| `waterfall-source` | 3개 소싱채널 워터폴 검색 |
| `price-monitor` | 소싱가 변동 감지 → 판매가 조정 |

## 전제 조건

- `sppt` CLI 설치
- 최소 1개 판매 채널 + 1개 소싱 채널 설정 (`sppt init`)

## 스킬 구조

각 스킬은 `SKILL.md` 마크다운 파일 하나로 구성됩니다.
Claude Code가 파일을 읽고, CLI 명령어를 조합해 워크플로우를 실행합니다.
