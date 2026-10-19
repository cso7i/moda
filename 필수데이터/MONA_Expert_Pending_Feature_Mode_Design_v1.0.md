# MONA Expert-Pending Feature Mode 설계 v1.0

## 1. 결정

명리 전문가 실제 승인 전에도 MONA 핵심 기능을 **개발/내부 테스트용으로 사용할 수 있게 한다.**
전문가 검수가 끝나지 않은 계산/해석 기능은 화면에 반드시 다음 중 하나를 표시한다.

```text
미승인
전문가 검수 전
```

현재 개발 Rule Profile:

```text
MONA_SAJU_RULE_001_DEV_ASSUMED
production_eligible = false
```

정책:

```text
DEV / INTERNAL_ALPHA = 미승인 기능 사용 가능
PRODUCTION = 실제 전문가 승인 전 차단
```

---

## 2. 핵심 정책 변경

기존:

```text
Expert Pending = 기능 비활성
```

변경:

```text
Expert Pending = 개발/내부 Alpha에서 사용 가능 + 명확한 미승인 표시
```

단:

```text
Rule/Data 자체가 부족함 = NOT_AVAILABLE
```

즉 `미승인`과 `미구현`을 반드시 구분한다.

---

## 3. 상태 모델

### ApprovalStatus

```text
APPROVED
VALIDATED_REFERENCE
EXPERT_PENDING
RESEARCH_ONLY
NOT_AVAILABLE
```

### ImplementationStatus

```text
READY
PARTIAL
NEXT
PLANNED
```

두 상태를 분리한다.

예:

```json
{
  "feature_id": "saju.four_pillars",
  "implementation_status": "PARTIAL",
  "approval_status": "EXPERT_PENDING"
}
```

의미:

- 기능은 개발/테스트에서 사용 가능
- 전문가 최종 승인 전
- Production 공개 불가

---

## 4. Build Channel

```text
DEV
INTERNAL_ALPHA
PRODUCTION
```

### DEV

- EXPERT_PENDING 사용 가능
- RESEARCH_ONLY 사용 가능
- 개발 상태 표시

### INTERNAL_ALPHA

- EXPERT_PENDING 사용 가능
- RESEARCH_ONLY 기본 숨김
- 미승인 결과에 Badge/Banner 표시

### PRODUCTION

- APPROVED / VALIDATED_REFERENCE만 공개
- EXPERT_PENDING 차단
- RESEARCH_ONLY 차단
- `production_eligible=false` Rule Profile 차단

---

## 5. 공통 UI

### Badge

```text
[미승인]
```

권장 부가 표현:

```text
전문가 검수 전
개발 기준
```

### Result Banner

```text
전문가 검수 전 결과

현재 결과는 개발 중인 명리 기준으로 계산되었습니다.
전문가 검수에 따라 일부 계산 기준이나 해석이 변경될 수 있습니다.
```

경고 Alert보다 부드러운 Info Banner로 표시한다.

### NOT_AVAILABLE

```text
전문가 검수 및 계산 Rule 확정 후 제공 예정
```

---

## 6. 즉시 체험 가능하게 만들 기능

### Profile / Birth Input

```text
사용 가능
approval = APPROVED(Product)
```

필드:

```text
display_name
calendar_type
birth_date
lunar_leap_month
birth_time_known
birth_time
country
region
timezone_id
birth_place
```

### 양력/음력 변환

```text
implementation = READY
approval = VALIDATED_REFERENCE
```

KASI spot validation 범위 내 개발 사용.

### 사주 4주

```text
implementation = PARTIAL
approval = EXPERT_PENDING
```

표시:

```text
년주
월주
일주
시주
음력
계산 상태
```

### 오행 Raw

```text
implementation = PARTIAL
approval = EXPERT_PENDING
```

현재는 천간/지지 기본 오행 Count만 제공.

반드시 안내:

```text
현재는 기본 오행 구성만 보여드립니다.
지장간 가중치와 신강약 분석은 전문가 검수 후 반영됩니다.
```

### 십신

```text
implementation = PARTIAL
approval = EXPERT_PENDING
```

현재 Core Engine의 Day Master 기준 천간/지장간 십신 사용.

### 지장간 기본

```text
implementation = PARTIAL
approval = EXPERT_PENDING
```

mapping만 사용하고 weight는 미사용.

### 기본 합충형파해

```text
implementation = NEXT
approval = EXPERT_PENDING
```

Master Data candidate mapping으로 개발 가능.

### Today Preview

```text
implementation = NEXT
approval = EXPERT_PENDING
```

입력:

```text
Natal Chart
+
Today Ganji
+
basic Ten Gods relation
+
basic relation candidate
+
starter Interpretation Rules
```

금지 dependency:

```text
신강약
격국
용신
대운 시작나이
신살 종합판정
```

출력:

```text
오늘의 Theme
Focus
Caution
작은 Action
```

상단:

```text
오늘의 미리보기 [미승인]
```

---

## 7. 아직 제공하지 않을 기능

다음은 `(미승인)`으로 열지 않고 `NOT_AVAILABLE` 유지한다.

```text
신강/신약
격국
용신
희신
기신
대운 순행/역행
대운 시작나이
합화 성립 최종판정
신살 종합판정
```

이유:

`전문가 승인이 없어서`가 아니라 **현재 Rule 자체가 충분히 구현되지 않았기 때문**이다.

---

## 8. Mona Chat

### Phase 1 — Local Preview

AI 연결 전에도:

```text
Interpretation Result
+
Mood
+
Concern
+
승인된/Seed Content
```

로 간단한 로컬 Preview를 제공할 수 있다.

표시:

```text
[미승인]
```

### Phase 2 — Internal AI Alpha

Server Gateway 연결 후 사용.

AI는 canonical Context에 있는 Factor만 설명한다.

AI가 다음을 추정하면 안 된다.

```text
용신
신강약
신살
대운
```

Context에 없으면 출력 금지.

---

## 9. Feature Registry

중앙 JSON에서 기능 상태를 관리한다.

예:

```json
{
  "feature_id": "saju.four_pillars",
  "implementation_status": "PARTIAL",
  "approval_status": "EXPERT_PENDING",
  "visible_in": ["DEV", "INTERNAL_ALPHA"],
  "enabled_in": ["DEV", "INTERNAL_ALPHA"]
}
```

코드 곳곳에 `if (debug)` / `if (!approved)`를 흩어놓지 않는다.

---

## 10. Flutter 신규 구조

```text
lib/core/feature/
├─ build_channel.dart
├─ feature_status.dart
├─ feature_definition.dart
├─ feature_registry.dart
└─ feature_policy.dart

lib/presentation/widgets/
├─ approval_badge.dart
├─ expert_pending_banner.dart
└─ feature_gate.dart
```

---

## 11. 권장 Dart Model

```dart
enum ApprovalStatus {
  approved,
  validatedReference,
  expertPending,
  researchOnly,
  notAvailable,
}

enum ImplementationStatus {
  ready,
  partial,
  next,
  planned,
}

enum BuildChannel {
  dev,
  internalAlpha,
  production,
}
```

FeatureDefinition:

```dart
class FeatureDefinition {
  final String featureId;
  final ImplementationStatus implementationStatus;
  final ApprovalStatus approvalStatus;
  final Set<BuildChannel> visibleIn;
  final Set<BuildChannel> enabledIn;
  final List<String> expertTaskIds;
}
```

---

## 12. FeaturePolicy

```dart
FeatureAccess evaluate(
  FeatureDefinition feature,
  BuildChannel channel,
)
```

반환:

```text
visible
enabled
showApprovalBadge
showPendingBanner
releaseBlocked
```

---

## 13. Production Gate

기존 Rule Profile Guard 유지:

```text
ruleProfile.production_eligible == true
```

추가 Gate:

```text
Public-visible EXPERT_PENDING feature count == 0
```

실패 marker:

```text
MONA_PRODUCTION_FEATURE_APPROVAL_GATE_FAIL
```

---

## 14. 운세 Hub 목표 화면

```text
사주팔자       [미승인]   >
오행           [미승인]   >
십신           [미승인]   >
관계           [미승인]   >
오늘 운세      [미승인]   >

대운           준비 중
용신           준비 중
```

현재처럼 placeholder만 보이지 않도록 한다.

---

## 15. 사주 결과 화면

상단:

```text
사주팔자 [미승인]
```

Banner:

```text
전문가 검수 전 결과
```

본문:

```text
년주
월주
일주
시주
```

세부:

```text
음력
Rule Profile
Engine Version
계산상태
```

Internal Alpha에서는 Rule ID를 항상 보여줘도 좋다.

---

## 16. 오행 화면

현재 `rawElementCounts` 사용.

예:

```text
목  2
화  3
토  2
금  1
수  0
```

그래프 제공 가능.

하지만 제목 또는 설명:

```text
기본 오행 구성 [미승인]
```

하단:

```text
현재는 천간·지지의 기본 오행 구성입니다.
지장간 가중치, 월령, 신강약은 아직 반영하지 않습니다.
```

---

## 17. 십신 화면

기본 구조:

```text
Friendly Mode (기본)
Expert Mode (선택)
```

예:

```text
Friendly:
책임·질서

Expert:
정관
Direct Officer
```

Term Registry만 사용한다.

AI/코드가 임의 동의어를 만들지 않는다.

---

## 18. Today Preview

화면:

```text
오늘의 모나 [미승인]

Theme
정리와 책임

Focus
중요한 일의 우선순위를 정리해 보세요.

Caution
급하게 답하기보다 한 번 더 확인해 보세요.

오늘의 작은 행동
우선순위 3개 적기
```

`점수`는 초기에는 없어도 된다.

---

## 19. 결과 저장

각 결과에 다음 metadata 포함:

```text
approval_status
rule_profile_id
engine_version
interpretation_version
content_version
```

이유:

전문가 승인 후 기존 결과를 재계산할 수 있어야 함.

기존 미승인 결과를 조용히 덮어쓰지 않는다.

History:

```text
2026-08-11 사주 [미승인]
```

향후:

```text
새 승인 기준으로 다시 계산하기
```

지원 가능.

---

## 20. 다국어 Resource Key

코드 literal 금지.

```text
status.expert_pending.badge
status.expert_pending.banner_title
status.expert_pending.banner_body
status.not_available.expert_review
status.research_only.badge
```

5 locale에 모두 추가한다.

---

## 21. Analytics

가능:

```text
feature_id
approval_status
rule_profile_id
build_channel
screen_id
success/fail
```

금지:

```text
생년월일
상담 본문
정밀 사주 내용
얼굴 이미지
```

---

## 22. Codex 구현 순서

### Phase 1 — Feature Gate

1. Feature Registry asset 추가
2. FeatureRegistry loader
3. BuildChannel
4. FeaturePolicy
5. ApprovalBadge
6. ExpertPendingBanner
7. FeatureGate
8. Production Approval Gate

Acceptance:

```text
FEATURE_REGISTRY_LOAD_PASS
DEV_PENDING_VISIBLE_PASS
INTERNAL_ALPHA_PENDING_VISIBLE_PASS
PRODUCTION_PENDING_BLOCK_PASS
APPROVAL_BADGE_PASS
EXPERT_PENDING_BANNER_PASS
RELEASE_APPROVAL_GATE_PASS
```

### Phase 2 — 체험 가능한 Core

1. Profile/Birth Input
2. 실제 Saju 계산
3. Four Pillars 화면
4. Raw Five Elements 화면
5. Ten Gods 화면
6. Hidden Stems 기본
7. History 저장

Acceptance:

```text
PROFILE_INPUT_PASS
PROFILE_TO_SAJU_PASS
FOUR_PILLARS_SCREEN_PASS
RAW_ELEMENTS_SCREEN_PASS
TEN_GODS_SCREEN_PASS
RESULT_SAVE_WITH_APPROVAL_STATUS_PASS
```

### Phase 3 — Today Preview

1. 오늘 Day Ganji
2. Natal vs Today basic Factor
3. Starter Interpretation Rule
4. Theme/Focus/Caution/Action
5. 미승인 표시

Acceptance:

```text
TODAY_PREVIEW_PASS
TODAY_PENDING_LABEL_PASS
NO_YONGSIN_DEPENDENCY_PASS
NO_DAEWOON_DEPENDENCY_PASS
```

---

## 23. Feature Matrix

| Feature | Implementation | Approval | DEV | Internal Alpha | Production |
|---|---|---|---:|---:|---:|
| Profile/Birth Input | NEXT | APPROVED | ✅ | ✅ | ✅ |
| 양력/음력 | READY | VALIDATED_REFERENCE | ✅ | ✅ | ✅ |
| 사주 4주 | PARTIAL | EXPERT_PENDING | ✅ | ✅ | ❌ |
| 오행 Raw | PARTIAL | EXPERT_PENDING | ✅ | ✅ | ❌ |
| 십신 | PARTIAL | EXPERT_PENDING | ✅ | ✅ | ❌ |
| 지장간 기본 | PARTIAL | EXPERT_PENDING | ✅ | ✅ | ❌ |
| 합충형파해 기본 | NEXT | EXPERT_PENDING | ✅ | ✅ | ❌ |
| Today Preview | NEXT | EXPERT_PENDING | ✅ | ✅ | ❌ |
| 신강약 | PLANNED | NOT_AVAILABLE | ❌ | ❌ | ❌ |
| 격국 | PLANNED | NOT_AVAILABLE | ❌ | ❌ | ❌ |
| 용신 | PLANNED | NOT_AVAILABLE | ❌ | ❌ | ❌ |
| 대운 시작 | PLANNED | NOT_AVAILABLE | ❌ | ❌ | ❌ |
| 신살 종합 | PLANNED | NOT_AVAILABLE | ❌ | ❌ | ❌ |

---

## 24. 최종 정책

MONA는 전문가 검수가 끝날 때까지 모든 기능을 잠그지 않는다.

개발/내부 Alpha에서 실제로 사용할 수 있게 할 기능:

```text
Profile/Birth Input
사주 4주
오행 Raw
십신
지장간 기본
기본 관계
Today Preview
History
```

미승인 기능에는 항상:

```text
[미승인]
전문가 검수 전
```

을 표시한다.

Rule/Data 자체가 부족한 기능은 `준비 중`으로 유지한다.

Public Production은 실제 전문가 승인 후에만 가능하다.

이 정책을 MONA MVP의 canonical Expert-Pending 개발 정책으로 사용한다.
