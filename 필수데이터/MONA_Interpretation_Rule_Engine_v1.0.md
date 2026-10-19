# 모나(MONA) Interpretation Rule Engine v1.0

**문서 상태:** Rule Architecture / Canonical Interpretation Design  
**기준 문서:** `mona_handoff.md`, `MONA_사주계산엔진_설계_v1.0.md`, `MONA_데이터모델_DB_Schema_v1.0.md`, `MONA_다국어_용어표준집_v1.0.md`, `MONA_앱_IA_화면명세_v1.0.md`  
**핵심 목적:** 사주 계산 결과를 재물·직업·사랑·관계·학업·오늘의 행동 같은 사용자 언어로 변환하되, 계산·해석·문장생성의 책임을 분리한다.

---

# 1. Interpretation Engine의 위치

모나는 다음 계층을 엄격하게 분리한다.

```text
Birth Input
    ↓
Saju Calculation Engine
    ↓
Canonical Saju Factors
    ↓
Fortune / Transit Factors
    ↓
Interpretation Rule Engine
    ↓
Interpretation Result
    ↓
Locale Content Resolver
    ↓
Mona AI
    ↓
사용자에게 보여줄 자연스러운 설명
```

### 핵심 원칙

- **Saju Engine**은 계산만 한다.
- **Interpretation Rule Engine**은 계산 결과를 의미 단위로 분류한다.
- **Locale Content Resolver**는 승인된 다국어 문장을 선택한다.
- **Mona AI**는 사용자의 현재 상황과 마음에 맞게 자연스럽게 설명한다.
- AI가 원국, 십신, 대운, 합충 관계를 다시 계산하거나 임의로 바꾸지 않는다.

---

# 2. Rule Engine의 책임

Interpretation Rule Engine은 다음을 수행한다.

1. Canonical Factor를 읽는다.
2. 적용 가능한 Domain을 찾는다.
3. Favorable / Caution / Neutral 방향을 판단한다.
4. 여러 Rule이 동시에 맞으면 우선순위와 충돌해결 규칙으로 정리한다.
5. 사용자 현재 Context를 반영해 설명 강도를 조정한다.
6. 사용자에게 보여줄 `content_key`를 선택한다.
7. 행동 제안 후보를 선택한다.
8. 근거를 `evidence`로 남긴다.
9. 공포성·단정성 표현을 출력 단계 전에 차단한다.
10. 최종 결과를 AI가 재해석하지 못하도록 구조화된 형태로 전달한다.

---

# 3. Interpretation의 4단계

```text
Factor
↓
Meaning
↓
Domain
↓
Guidance
```

예:

```text
정재 활성
↓
안정적인 자원 관리 / 현실적 성과
↓
재물 · 직업
↓
예산을 정리하거나 현실적인 선택을 우선
```

또 다른 예:

```text
편관 강한 영향
↓
압박 / 도전 / 추진력
↓
직업 · 관계
↓
서두르지 말고 힘을 사용할 대상을 선택
```

---

# 4. 입력 Canonical Context

Rule Engine 입력은 번역 문자열이 아니라 canonical ID와 수치/상태를 사용한다.

```json
{
  "chart_id": "saj_xxx",
  "rule_profile": "MONA_SAJU_KR_001",
  "day_master": "stem.geng",
  "five_elements": {
    "element.wood": 0.84,
    "element.fire": 0.45,
    "element.earth": 0.62,
    "element.metal": 0.91,
    "element.water": 0.55
  },
  "ten_gods": {
    "ten_gods.direct_wealth": 0.78,
    "ten_gods.direct_officer": 0.66
  },
  "relations": [
    "interaction.combination"
  ],
  "fortune": {
    "period_type": "DAY",
    "active_factor_ids": [
      "ten_gods.direct_wealth",
      "interaction.combination"
    ]
  },
  "context": {
    "mood_id": "mood.worried",
    "concern_id": "domain.career",
    "relationship_type": null
  }
}
```

---

# 5. Rule의 기본 구조

각 Rule은 다음 필드를 가진다.

```text
rule_id
rule_group
version
status
priority
scope
domain_ids
when
exclude_when
effect
content
action
safety
evidence_template
```

### 예시

```json
{
  "rule_id": "IR_TENGOD_DIRECT_WEALTH_CAREER_001",
  "rule_group": "TEN_GOD",
  "version": "1.0.0",
  "status": "DRAFT",
  "priority": 500,
  "scope": ["DAY", "MONTH", "YEAR"],
  "domain_ids": ["domain.wealth", "domain.career"],
  "when": {
    "all": [
      {"factor": "ten_gods.direct_wealth", "op": ">=", "value": 0.65}
    ]
  },
  "exclude_when": {
    "any": [
      {"flag": "risk.critical", "op": "==", "value": true}
    ]
  },
  "effect": {
    "direction": "FAVORABLE",
    "strength": "MEDIUM"
  },
  "content": {
    "summary_key": "interp.direct_wealth.steady.summary",
    "detail_key": "interp.direct_wealth.steady.detail"
  },
  "action": {
    "action_key": "action.wealth.review_budget"
  }
}
```

---

# 6. Rule DSL v1

v1에서는 복잡한 프로그래밍 언어를 만들지 않고 JSON 기반 제한형 DSL을 사용한다.

지원 operator:

```text
==
!=
>
>=
<
<=
IN
NOT_IN
EXISTS
NOT_EXISTS
```

논리:

```text
all
any
not
```

### 예시

```json
{
  "all": [
    {"factor": "ten_gods.direct_officer", "op": ">=", "value": 0.60},
    {
      "any": [
        {"context": "concern_id", "op": "==", "value": "domain.career"},
        {"scope": "period_type", "op": "IN", "value": ["MONTH", "YEAR"]}
      ]
    }
  ]
}
```

---

# 7. Rule Scope

Rule은 적용 기간을 명확히 한다.

```text
NATAL
DAEWOON
YEAR
MONTH
WEEK
DAY
COMPATIBILITY
NAME
TOJEONG
STYLE
```

예:

- 성격/기질: `NATAL`
- 올해 운세: `YEAR`
- 오늘 운세: `DAY`
- 나의 패션: `STYLE`
- 궁합: `COMPATIBILITY`

---

# 8. Domain 표준

v1 Domain은 다음을 기본으로 한다.

```text
domain.overall
domain.wealth
domain.career
domain.business
domain.love
domain.marriage
domain.relationships
domain.family
domain.study
domain.growth
domain.wellbeing
domain.change
```

사용자 친화 UI에서는 `일`, `돈`, `사랑`, `관계`, `성장`처럼 줄여 표시할 수 있지만 내부 ID는 유지한다.

---

# 9. 방향성 Direction

Rule 결과는 기본적으로 다음 세 가지다.

```text
FAVORABLE
CAUTION
NEUTRAL
```

필요 시 `MIXED`를 Aggregate 단계에서 생성할 수 있다.

### 중요한 원칙

`CAUTION`은 “불운 확정”을 뜻하지 않는다.

사용자 표현에서는:

- 신중
- 속도 조절
- 확인 필요
- 말과 행동을 한 번 더 생각
- 과도함 조절

등으로 번역한다.

---

# 10. Strength

Rule의 해석 강도는 다음으로 제한한다.

```text
LOW
MEDIUM
HIGH
```

사용자에게 0.873과 같은 정밀 숫자를 그대로 보여주지 않는다.

내부 계산 수치는 QA와 Rule 평가에만 사용한다.

---

# 11. 십신 → 기본 의미 Mapping

아래 Mapping은 **초기 해석 골격**이며 최종 사용자 콘텐츠는 전문가 검토 대상이다.

| term_id | 기본 Theme | 주요 Domain |
|---|---|---|
| `ten_gods.peer` | 자기주도, 동료, 독립 | 관계, 성장 |
| `ten_gods.rob_wealth` | 경쟁, 공유 자원, 선택 | 돈, 관계 |
| `ten_gods.eating_god` | 표현, 생산, 창작, 여유 | 성장, 일, 관계 |
| `ten_gods.hurting_officer` | 개성, 문제제기, 독립적 표현 | 일, 관계 |
| `ten_gods.indirect_wealth` | 기회, 유동성, 외부 자원 | 돈, 사업 |
| `ten_gods.direct_wealth` | 안정, 계획, 현실적 성과 | 돈, 일 |
| `ten_gods.seven_killings` | 도전, 압박, 추진 | 일, 변화 |
| `ten_gods.direct_officer` | 책임, 구조, 질서 | 일, 관계 |
| `ten_gods.indirect_resource` | 직관, 관찰, 비정형 학습 | 공부, 성장 |
| `ten_gods.direct_resource` | 학습, 보호, 지원, 준비 | 공부, 성장 |

Friendly English는 Terminology Registry를 따른다.

---

# 12. 오행 → 기본 의미 Mapping

오행은 성격 단정이 아니라 **해석 Theme**로 사용한다.

| Element | Theme 예 |
|---|---|
| Wood | 성장, 시작, 확장 |
| Fire | 표현, 활력, 가시성 |
| Earth | 안정, 정리, 현실성 |
| Metal | 기준, 구조, 결단 |
| Water | 유연성, 정보, 흐름 |

### 금지

> “목이 많으니 당신은 반드시 외향적이다.”

### 권장

> “성장이나 확장에 관심이 커지는 방식으로 나타날 수 있어요.”

---

# 13. 합·충·형·파·해 Mapping

| Interaction | Expert | Friendly Theme |
|---|---|---|
| 合 | 합 | 연결, 결합, 협력 |
| 沖 | 충 | 변화, 긴장, 방향 전환 |
| 刑 | 형 | 마찰, 반복 압박 |
| 破 | 파 | 흐트러짐, 계획 수정 |
| 害 | 해 | 미묘한 불편, 숨은 마찰 |

Rule은 전통명 자체를 긍정/부정으로 고정하지 않는다.

예를 들어 `충`은:

```text
변화가 필요한 상황에서는 기회
안정이 중요한 상황에서는 주의
```

처럼 Context에 따라 해석한다.

---

# 14. Context의 역할

Context는 사주 계산 결과를 바꾸지 않는다.

```text
Same Saju Factor
+
Different User Context
=
Different Explanation / Guidance
```

예:

```text
편관 활성 + 취업 고민
→ 경쟁과 압박 속 준비 전략

편관 활성 + 연애 고민
→ 상대를 몰아붙이지 않는 대화 전략
```

즉 Factor는 동일하지만 Domain과 Guidance가 달라진다.

---

# 15. Mood Context

Mood는 **운세 점수 조작용이 아니라 표현과 행동 제안용**이다.

예:

```text
mood.worried
```

이면:

- 표현 강도를 낮춘다.
- 확정적 문장을 피한다.
- 행동을 작고 현실적인 단계로 제한한다.

금지:

```text
걱정 중이므로 운세 점수 -20
```

---

# 16. Concern Context

사용자가 현재 고른 고민은 Domain 우선순위에 사용한다.

예:

```text
concern_id = domain.career
```

이면 오늘 운세에서:

1. 직업 관련 Rule
2. 전체운
3. 기타 Domain

순으로 설명 우선순위를 줄 수 있다.

다른 Domain의 계산 결과는 삭제하지 않는다.

---

# 17. Life-stage Context

나이는 설명에 사용할 수 있지만 고정관념을 만들지 않는다.

권장:

```text
age_band
employment_state
relationship_state
student_state
family_context
```

예:

- 같은 `study` Rule이라도 학생과 직장인의 학습/자격증 Context를 다르게 설명
- 같은 `career` Rule이라도 취업 준비와 재직 중 이직 고민을 구분

성별·연령 stereotype 문구는 Rule에서 금지한다.

---

# 18. Rule Matching Pipeline

```text
1. Input validation
2. Scope filter
3. Domain filter
4. Condition match
5. Exclusion match
6. Priority sort
7. Conflict resolution
8. Diversity control
9. Safety filter
10. Content resolution
11. Action selection
12. Output trace
```

---

# 19. Priority 규칙

priority는 숫자가 클수록 우선.

권장 영역:

```text
900-999  Safety / critical conflict
800-899  Strong structural rule
700-799  Natal + transit interaction
600-699  Strong single factor
500-599  Normal factor
300-499  Context personalization
100-299  Fallback / generic
```

Context Rule이 계산 Rule을 뒤집지 않도록 priority zone을 분리한다.

---

# 20. 충돌 해결

동일 Domain에서 Favorable과 Caution이 동시에 발생할 수 있다.

절대 하나를 억지로 제거하지 않는다.

예:

```text
FAVORABLE: 정재 활성
CAUTION: 충 활성
```

Aggregate:

```text
direction = MIXED
```

사용자 설명:

> 현실적인 성과를 만들 수 있는 흐름은 있지만, 일정이나 관계 변화에는 조금 유연하게 대응하는 편이 좋아요.

---

# 21. Conflict Resolution 순서

```text
1. Safety Rule
2. Invalidation / exclusion
3. Strong structure
4. Natal-transit relation
5. Ten Gods
6. Five Elements
7. Context preference
8. Generic fallback
```

---

# 22. 중복 억제

여러 Rule이 같은 의미를 반복하면 하나로 묶는다.

예:

```text
정재 활성 → 계획적인 돈 관리
Earth 강화 → 현실적 정리
```

둘 다 같은 `resources_planning` Theme으로 연결된다면:

```text
theme_id = theme.resources_planning
```

하나의 설명으로 합친다.

---

# 23. Theme Layer

전문용어와 사용자 문장을 연결하는 중간 계층.

예:

```text
theme.growth
theme.expression
theme.structure
theme.resources
theme.opportunity
theme.challenge
theme.relationship
theme.transition
theme.focus
theme.rest
theme.communication
theme.boundary
```

영어에서는 Tarot-style UX lexicon을 활용하기 좋은 영역이다.

---

# 24. Interpretation Content 역할

콘텐츠는 역할별로 분리한다.

```text
SUMMARY
DETAIL
EVIDENCE
GUIDANCE
ACTION
CAUTION
REFLECTION
```

예:

```text
content_key = interp.direct_officer.career.summary
role = SUMMARY
```

---

# 25. Content Key 규칙

```text
interp.<factor/theme>.<domain>.<role>
```

예:

```text
interp.direct_wealth.wealth.summary
interp.direct_wealth.career.detail
interp.seven_killings.career.guidance
interp.clash.relationships.caution
```

Action:

```text
action.<domain>.<action_name>
```

예:

```text
action.wealth.review_budget
action.career.write_three_options
action.relationships.delay_reply
```

---

# 26. Locale Content

한 Rule은 locale에 의존하지 않는다.

```text
Rule
↓
content_key
↓
ko-KR content
en content
ja-JP content
zh-CN content
fr-FR content
```

Rule JSON 안에 한국어 문장을 넣지 않는다.

---

# 27. Friendly / Expert Content

상세 화면에서는 두 레벨 제공 가능.

### Friendly

> 오늘은 책임과 정리가 중요한 흐름이에요.

### Expert

> 정관의 영향이 상대적으로 강하게 들어오고 있습니다.

영어:

### Friendly

> Structure and responsibility are stronger themes today.

### Expert

> Direct Officer (Zheng Guan) is more active in today’s reading.

---

# 28. 행동 제안 Rule

운세 설명은 가능한 경우 작은 행동으로 끝난다.

Action은 다음 조건을 만족해야 한다.

- 오늘 할 수 있음
- 비용이 거의 들지 않음
- 과도한 위험이 없음
- 의료/법률/투자 결정을 대신하지 않음
- 너무 추상적이지 않음
- 사용자의 고민과 연결 가능

예:

```text
action.career.list_priorities
action.wealth.review_budget
action.relationships.read_before_send
action.study.focus_25min
action.growth.take_one_small_step
```

---

# 29. 행동 제안 금지 예

사용하지 않는다.

```text
오늘 반드시 퇴사하세요.
이 사람과 헤어지세요.
지금 주식을 사세요.
치료를 중단하세요.
대출을 받으세요.
오늘 계약하면 반드시 성공합니다.
```

---

# 30. 긍정 / 주의 문구 원칙

### Favorable

좋은 흐름을 “확정된 성공”으로 표현하지 않는다.

금지:

> 오늘 계약하면 성공합니다.

권장:

> 현실적인 조건을 잘 정리하면 협의가 조금 더 수월하게 진행될 가능성이 있어요.

### Caution

불운을 “확정된 사고”로 표현하지 않는다.

금지:

> 사고수가 있습니다.

권장:

> 오늘은 서두를수록 실수가 생기기 쉬운 흐름으로 볼 수 있어요. 이동이나 중요한 결정은 한 번 더 확인해보세요.

---

# 31. Fear-Selling Guard

다음 표현은 자동 QA / Content Lint 대상.

```text
반드시
100%
무조건
대박
파산
이혼 확정
사고가 난다
죽는다
병에 걸린다
저주
귀신이 붙었다
결제하면 막을 수 있다
부적을 사야 한다
```

전통 용어가 포함된 경우에는 기술 설명 문맥인지 확인한다.

---

# 32. Health / Legal / Finance 경계

사주 콘텐츠가 의료·법률·금융 판단을 대신하지 않는다.

예:

### 금전

> 소비와 계약 조건을 꼼꼼히 확인해보세요.

가능.

> 오늘 특정 종목을 매수하세요.

금지.

### 건강생활

> 일정이 빡빡하다면 휴식 시간을 확보해보세요.

가능.

> 이 시기에 간 질환이 생깁니다.

금지.

---

# 33. Today Reading 구성

오늘 운세는 최소 다음 구조를 갖는다.

```text
overall_summary
top_theme
domain_results
caution
today_action
lucky_lifestyle
evidence
```

예:

```json
{
  "top_theme_id": "theme.structure",
  "overall_direction": "MIXED",
  "domain_results": [
    {
      "domain_id": "domain.career",
      "direction": "FAVORABLE",
      "content_key": "interp.direct_officer.career.summary"
    }
  ],
  "action_key": "action.career.list_priorities"
}
```

---

# 34. Domain Score

점수는 Rule Engine의 직접 진실값이 아니다.

점수를 제공하려면 별도 Scoring Layer를 둔다.

```text
Rule Match
↓
Weighted Evidence
↓
Domain Score
↓
UI Score
```

### 권장 제한

```text
20 ~ 90
```

0이나 100 같은 극단값은 기본적으로 사용하지 않는다.

### 해석

점수는:

> 해당 시점의 여러 factor를 상대적으로 요약한 참고 지표

로 설명한다.

---

# 35. Scoring과 Interpretation 분리

점수가 70이라고 해서 반드시 문장이 “좋다”일 필요는 없다.

예:

```text
score = 72
direction = MIXED
```

가능.

왜냐하면 기회와 주의 factor가 동시에 있을 수 있기 때문이다.

---

# 36. 대운 / 세운 / 월운 / 일운 공통 구조

```text
Natal Factors
+
Transit Factors
+
Natal-Transit Relations
↓
Interpretation Rules
```

기간이 달라도 Rule Engine의 구조는 공유한다.

차이는:

```text
scope
weight
content granularity
```

이다.

---

# 37. 기간별 Weight 예

정식 수치는 전문가 검증 전 확정하지 않는다.

구조만 정의한다.

```text
DAEWOON    long_term_weight
YEAR       annual_weight
MONTH      monthly_weight
DAY        daily_weight
```

오늘 운세에서 대운 영향은 Context로 사용하되 일운 문장을 압도하지 않도록 제한한다.

---

# 38. 궁합 Rule Engine

궁합은 두 Chart 결과를 입력으로 받는다.

```text
Chart A
+
Chart B
+
Relationship Type
↓
Compatibility Factors
↓
Compatibility Interpretation Rules
```

Domain:

```text
compat.emotion
compat.communication
compat.lifestyle
compat.values
compat.conflict
compat.strength
```

결과:

> 좋음 / 나쁨

이 아니라:

- 잘 맞는 부분
- 조율이 필요한 부분
- 관계에서 활용할 장점
- 갈등이 생겼을 때 참고할 점

으로 출력한다.

---

# 39. 이름풀이 Rule

성명학/이름 Rule은 사주 Rule과 독립 모듈로 둔다.

```text
Name Factors
+
Saju Factors (optional)
↓
Name Interpretation
```

이름만 보는 결과와 사주+이름 결과를 구분한다.

---

# 40. 토정비결 Rule

토정비결은 독립 `rule_group = TOJEONG`으로 관리한다.

사주 Rule을 그대로 재사용해서 “토정비결”이라고 이름만 바꾸지 않는다.

---

# 41. 별자리 Rule

별자리는 `rule_group = ZODIAC`.

사주와 계산근거가 다르므로 결과를 혼합 계산하지 않는다.

화면에서는 가벼운 참고 콘텐츠로 명확히 구분한다.

---

# 42. 직관형 운세 Rule

직관형 콘텐츠는 다음 UX Theme을 사용할 수 있다.

```text
Theme
Energy
Influence
Opportunity
Challenge
Guidance
Focus
Reflection
```

실제 영적 능력을 주장하지 않는다.

---

# 43. 타로와의 어휘 공유

타로에서 사용되는 중립적 UX 어휘는 다음 영역에서 재사용 가능하다.

```text
Theme
Influence
Opportunity
Challenge
Guidance
Focus
Reflection
Balance
Flow
Timing
```

하지만 다음은 금지한다.

```text
정관 = The Emperor
편관 = The Tower
```

즉 **타로 카드와 명리 Factor를 1:1 대응시키지 않는다.**

공유하는 것은 어휘 스타일이지 점술 체계가 아니다.

---

# 44. 나의 패션 Rule 연결

Interpretation Rule의 결과를 직접 Color/Gem으로 고정하지 않는다.

```text
Interpretation Theme
+
Five Element Context
+
Daily Fortune
+
Situation
↓
Style Rule Engine
```

예:

```text
theme.structure
+
element.metal supportive
+
situation.interview
↓
color.navy
```

이 mapping 역시 별도 Rule Version을 가진다.

---

# 45. Mona AI Input Schema

Rule Engine은 AI에게 다음 형태로 전달한다.

```json
{
  "locale": "ko-KR",
  "profile_context": {
    "age_band": "30s",
    "employment_state": "EMPLOYED"
  },
  "current_context": {
    "mood_id": "mood.worried",
    "concern_id": "domain.career"
  },
  "reading": {
    "scope": "DAY",
    "overall_direction": "MIXED",
    "themes": [
      "theme.structure",
      "theme.transition"
    ],
    "domain_results": [
      {
        "domain_id": "domain.career",
        "direction": "FAVORABLE",
        "strength": "MEDIUM",
        "summary_content_key": "interp.direct_officer.career.summary",
        "evidence_term_ids": [
          "ten_gods.direct_officer"
        ]
      }
    ],
    "action_key": "action.career.list_priorities"
  }
}
```

AI는 이 구조 밖의 새로운 명리 결론을 추가하지 않는다.

---

# 46. Mona AI Output Contract

AI 출력은 최종적으로 다음 역할만 한다.

```text
summary_text
supportive_explanation
practical_guidance
reflection_question(optional)
```

예:

```json
{
  "summary_text": "오늘은 책임과 정리가 중요한 날로 볼 수 있어요.",
  "supportive_explanation": "해야 할 일을 한꺼번에 처리하기보다 우선순위를 정하면 흐름을 활용하기 좋아요.",
  "practical_guidance": "오늘 꼭 끝낼 일 세 가지만 적어보세요.",
  "reflection_question": "지금 가장 먼저 정리하고 싶은 일은 무엇인가요?"
}
```

---

# 47. AI 금지 동작

AI는 다음을 하면 안 된다.

- 계산 결과와 다른 사주팔자 제시
- 등록되지 않은 십신 판정
- 대운 시작 나이 임의 변경
- Rule Engine에 없는 중대한 불운 추가
- 사용자가 묻지 않은 건강·사망·범죄 예측
- 전문가 용어를 임의로 바꿔 새로운 개념 생성
- 다른 점술 체계를 임의 결합해 결론 강화

---

# 48. Content Review Workflow

```text
Rule Author
↓
Domain Expert Review
↓
Content Writer
↓
Linguistic Review
↓
Safety Review
↓
APPROVED
```

다국어는:

```text
ko-KR approved
↓
locale translation
↓
linguistic review
↓
domain terminology review
↓
APPROVED
```

영어에서 의미가 애매하면 한국어 직역보다 `term_id`의 정의를 기준으로 번역한다.

---

# 49. Rule 상태

```text
DRAFT
EXPERT_REVIEW
SHADOW
APPROVED
DEPRECATED
```

### SHADOW

실제 사용자 결과에는 사용하지 않고 테스트 데이터에서만 계산한다.

신강약/용신처럼 검증이 필요한 영역에서 유용하다.

---

# 50. Rule Versioning

예:

```text
Interpretation Engine = 1.0.0
Rule Pack             = MONA_INTERP_KR_001
Content               = 1.0.0
Terminology           = 1.0.0
```

Rule 수정 시 기존 결과의 재현성을 깨지 않는다.

---

# 51. Rule Trace

각 결과에는 적용 Rule을 기록한다.

```json
{
  "rule_id": "IR_TENGOD_DIRECT_OFFICER_CAREER_001",
  "matched": true,
  "priority": 600,
  "domain": "domain.career",
  "direction": "FAVORABLE",
  "evidence": [
    {
      "factor_id": "ten_gods.direct_officer",
      "value": 0.71
    }
  ]
}
```

사용자 화면에는 필요 시 간략한 근거만 보여준다.

QA에서는 전체 Trace 사용.

---

# 52. Explainability

사용자가 “왜?”를 눌렀을 때:

### Friendly

> 오늘은 책임과 구조를 뜻하는 흐름이 상대적으로 강조돼요.

### Expert

> 오늘 운에서 정관 관련 Factor가 활성화되었습니다.

원국/운세 계산의 실제 canonical evidence와 연결한다.

---

# 53. Fallback Rule

특정 Rule이 충분히 맞지 않더라도 결과를 억지 생성하지 않는다.

```text
NO_STRONG_SIGNAL
```

일반 문구:

> 오늘은 특정한 흐름 하나보다 균형 있게 하루를 보내는 것이 중요해 보여요.

Fallback은 priority 100 이하.

---

# 54. No-Reading 상태

입력 데이터가 충분하지 않거나 계산 품질이 낮다면:

```text
PARTIAL
RULE_AMBIGUOUS
OUT_OF_VALIDATED_RANGE
```

사용자에게 제한을 알려준다.

AI가 빈 공간을 상상해서 채우지 않는다.

---

# 55. Interpretation Result Schema

```json
{
  "interpretation_version": "1.0.0",
  "rule_pack": "MONA_INTERP_KR_001",
  "scope": "DAY",
  "status": "VALID",
  "overall": {
    "direction": "MIXED",
    "score": 72,
    "theme_ids": [
      "theme.structure",
      "theme.transition"
    ]
  },
  "domains": [
    {
      "domain_id": "domain.career",
      "direction": "FAVORABLE",
      "strength": "MEDIUM",
      "summary_content_key": "interp.direct_officer.career.summary",
      "detail_content_key": "interp.direct_officer.career.detail",
      "evidence_term_ids": [
        "ten_gods.direct_officer"
      ]
    }
  ],
  "actions": [
    {
      "action_key": "action.career.list_priorities",
      "priority": 1
    }
  ],
  "trace_ids": [
    "trace_xxx"
  ]
}
```

---

# 56. 초기 Starter Rules

v1 개발에서 먼저 작성할 Rule 묶음:

### Ten Gods

- 비견
- 겁재
- 식신
- 상관
- 편재
- 정재
- 편관
- 정관
- 편인
- 정인

각각:

```text
wealth
career
love
relationships
study/growth
```

중 관련 Domain을 연결.

### Relations

- 합
- 충
- 형
- 파
- 해

### Five Elements

- Wood
- Fire
- Earth
- Metal
- Water

### Transit

- natal + year
- natal + month
- natal + day

---

# 57. Rule 수량 계획

무작정 수천 개를 만들지 않는다.

초기 목표:

```text
Core Factor Rules        30~50
Domain Combination Rules 30~50
Context Rules            20~30
Action Rules             30~50
Safety/Fallback Rules    10~20
```

총 약 120~200개의 검증된 Rule로 MVP를 시작하는 것을 권장한다.

이후 실제 사용 데이터를 보며 확장한다.

---

# 58. Rule Test 구조

각 Rule은 최소 다음 테스트를 가진다.

```text
MATCH
NO_MATCH
BOUNDARY
CONFLICT
EXCLUSION
LOCALE_CONTENT_EXISTS
SAFETY_PASS
TRACE_PASS
```

---

# 59. Golden Interpretation Test

명리 전문가가 수작업으로 만든 기준 사례를 저장한다.

예:

```text
case_id
input_chart
active_transit
expected_domains
expected_direction
expected_theme_ids
forbidden_outputs
expert_notes
```

문장 자체가 완전히 동일할 필요는 없지만 의미 범위는 유지해야 한다.

---

# 60. 다국어 Rule Test

같은 Interpretation Result에 대해 locale만 변경한다.

검증:

```text
same domain
same direction
same evidence
same action meaning
different localized text
```

언어 변경으로 운세 점수나 계산 결과가 달라지면 FAIL.

---

# 61. Terminology Lint 연결

콘텐츠에 다음이 나오면 검수한다.

예:

```text
ten_gods.direct_officer
```

영어 content에 승인되지 않은 `Official Star`, `Career God` 등이 등장하면 FAIL/Review.

---

# 62. Safety Lint 연결

콘텐츠/AI 결과에 공포성·확정적 표현이 있으면 Review.

특히:

```text
death
disease diagnosis
accident certainty
divorce certainty
guaranteed wealth
guaranteed success
supernatural certainty
```

---

# 63. Data Model 연결

현재 DB Schema와 연결:

```text
interpretation_rule
interpretation_content
reading_session
reading_factor
history_item
```

추가 권장 테이블:

```text
interpretation_trace
action_rule
theme_registry
```

다음 DB migration에서 추가한다.

---

# 64. 추가 DB Schema 제안

```text
theme_registry
theme_label
action_rule
action_content
interpretation_trace
```

`theme_registry`도 term_id처럼 immutable ID 사용.

---

# 65. Rule Engine 모듈 구조

```text
interpretation/
├─ matcher/
├─ aggregator/
├─ conflict/
├─ scoring/
├─ safety/
├─ content_resolver/
├─ action_selector/
├─ trace/
├─ rules/
│  ├─ ten_gods/
│  ├─ elements/
│  ├─ relations/
│  ├─ transit/
│  ├─ context/
│  └─ fallback/
└─ tests/
```

---

# 66. 실행 순서

```text
Load Canonical Chart
↓
Load Active Fortune Context
↓
Build InterpretationContext
↓
Match Rules
↓
Resolve Conflict
↓
Aggregate Themes
↓
Calculate Optional Score
↓
Select Content Keys
↓
Select Action
↓
Safety Validate
↓
Create InterpretationResult
↓
Resolve Locale Content
↓
Send Controlled Context to Mona AI
```

---

# 67. MVP 우선 구현

### 반드시 구현

- Rule DSL
- Rule loader
- Rule matcher
- domain mapping
- favorable/caution/neutral
- priority
- conflict resolution
- theme aggregation
- content key resolution
- action selector
- trace
- safety filter
- locale independence
- unit tests

### 전문가 검토 후 강화

- 신강약
- 격국
- 용신
- 신살
- 복합 Rule Weight
- 세밀한 score calibration

---

# 68. Acceptance Gate

```text
RULE_DSL_PARSE_PASS
RULE_MATCH_PASS
RULE_NO_MATCH_PASS
PRIORITY_PASS
CONFLICT_RESOLUTION_PASS
DOMAIN_MAPPING_PASS
THEME_DEDUP_PASS
ACTION_SELECTOR_PASS
SAFETY_LINT_PASS
TERMINOLOGY_LINT_PASS
5_LOCALE_CONTENT_PASS
TRACE_PASS
DETERMINISTIC_REPLAY_PASS
AI_CONTEXT_BOUNDARY_PASS
```

---

# 69. 다음 단계

다음 설계는 **MONA AI Context Engine + Persona/Prompt Contract v1**이다.

다음 항목을 정의한다.

1. Mona가 어떤 사용자 정보를 언제 읽는지
2. 최근 Mood/Concern 선택 기준
3. History retrieval 범위
4. 관계 프로필 연결
5. 운세 Context 주입 방식
6. 개인 정보 최소화
7. 대화 장기 기억 요약
8. 다국어 Persona
9. Friendly/Expert 용어 제어
10. 안전한 상담 응답 규칙
11. Prompt injection 및 임의 명리 재계산 방지
12. AI output schema
13. 비용/토큰 절감용 Context 압축
14. 상담 저장/삭제 정책

---

# 70. v1 핵심 결정 요약

- Interpretation Rule Engine은 계산과 AI 사이의 독립 계층
- JSON 기반 제한형 Rule DSL
- canonical factor ID만 사용
- Domain / Direction / Strength / Theme 구조
- Context는 계산 결과를 바꾸지 않고 설명과 우선순위에만 사용
- Mood는 점수를 깎거나 올리는 입력으로 사용하지 않음
- favorable/caution 충돌은 `MIXED`로 보존
- 중복 Rule은 Theme layer에서 통합
- 모든 문장은 locale별 content key로 관리
- Expert / Friendly 문장 분리 가능
- 행동 제안은 작고 현실적인 수준
- 공포성·확정성·의학/법률/투자 지시 금지
- AI는 Rule 결과를 설명하지만 새로운 명리 결론을 만들지 않음
- 초기 120~200개 수준의 검증된 Rule부터 시작
- 다음 단계는 Mona AI Context Engine + Persona/Prompt Contract

---

**MONA Interpretation Rule Engine v1.0**  
**Next: MONA AI Context Engine + Persona/Prompt Contract v1**
