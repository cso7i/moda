# 모나(MONA) 사주 계산엔진 설계 v1.0

**문서 상태:** Draft for Expert Validation  
**목표:** 모나의 사주·운세·궁합·마음지키미·Color/Gem 기능이 공통으로 사용할 수 있는 결정적(deterministic) 사주 계산 코어 정의  
**핵심 원칙:** 계산은 코드/Rule Engine, 설명과 개인화는 Interpretation/Mona AI

---

## 1. 엔진 설계 목표

모나 사주 계산엔진은 같은 입력과 같은 Rule Version에 대해 항상 같은 결과를 반환해야 한다.

엔진은 다음 네 계층을 분리한다.

```text
Birth Input
    ↓
Calendar / Time Normalization
    ↓
Four Pillars Calculation
    ↓
Saju Factor Calculation
    ↓
Fortune Cycle Calculation
    ↓
Canonical Saju JSON
```

AI는 위 계산 결과를 변경하거나 재계산하지 않는다.

---

## 2. 공식 달력 및 시간 데이터 기준

### 2.1 달력 기준

v1에서는 한국천문연구원(KASI)의 음양력 변환·월력·24절기 자료를 외부 검증 기준(Golden Reference)으로 사용한다.

운영 앱에서 모든 요청마다 외부 API를 호출하는 구조는 사용하지 않는다.

권장 구조:

```text
KASI 공식 자료
    ↓
개발/검증 단계 수집
    ↓
정규화된 Local Reference Table
    ↓
Mona Calendar Engine
```

즉 앱의 계산은 자체 엔진에서 수행하고, KASI 자료는 검증·교차검증 기준으로 사용한다.

### 2.2 v1 검증 범위

우선 검증 대상 출생연도는 다음과 같이 권장한다.

```text
1900-01-01 ~ 2050-12-31
```

이 범위를 먼저 완전 검증한 뒤 필요 시 확장한다.

### 2.3 시간대

시간대는 단순 `UTC+9` 상수로 처리하지 않는다.

```text
timezone_id = Asia/Seoul
```

처럼 IANA Time Zone Database 기반 지역 시간대를 저장한다.

해외 출생자를 고려해 각 출생지역의 timezone_id를 별도로 저장할 수 있도록 한다.

---

## 3. 입력 데이터 모델

```text
BirthInput
├─ profile_id
├─ name_or_nickname
├─ calendar_type        SOLAR | LUNAR
├─ lunar_leap_month     true | false | null
├─ birth_date
├─ birth_time
├─ birth_time_known
├─ country
├─ region
├─ latitude
├─ longitude
├─ timezone_id
├─ calculation_profile
└─ rule_version
```

### 필수 입력

- 생년월일
- 양력/음력
- 출생시간을 아는지 여부

### 조건부 입력

음력 입력이면 윤달 여부가 필요하다.

출생시간을 모르면:

```text
birth_time_known = false
hour_pillar = null
```

로 처리한다.

시주가 필요한 분석 결과는 `PARTIAL` 상태를 반환한다.

---

## 4. 시간 처리 계층

출생시각은 세 값을 구분하여 저장한다.

```text
input_local_time
civil_local_time
adjusted_saju_time
```

### input_local_time

사용자가 입력한 시각.

### civil_local_time

해당 지역의 역사적 표준시를 적용한 시각.

### adjusted_saju_time

선택된 명리 Rule에 따라 실제 사주 계산에 사용되는 시각.

이 구조를 사용하면 향후 진태양시 적용 여부가 달라져도 원본 입력을 보존할 수 있다.

---

## 5. 진태양시 정책

명리학에서는 출생지역의 태양시 보정을 사용하는 방식과 표준시를 그대로 사용하는 방식이 혼재하므로 하나의 방식으로 고정하지 않는다.

v1 엔진은 두 모드를 지원하도록 설계한다.

```text
solar_time_mode

CIVIL_TIME
TRUE_SOLAR_TIME
```

### 권장 초기값

```text
CIVIL_TIME
```

진태양시 적용 여부는 전문가 검토 후 모나의 정식 Rule Profile에서 확정한다.

사용자 화면에서 Rule을 매번 선택하게 하기보다는, 내부 Rule Version으로 관리한다.

---

## 6. 날짜 경계 정책

사주 계산에서 가장 중요한 경계값은 별도 Rule로 관리한다.

```text
year_boundary_rule
month_boundary_rule
day_boundary_rule
hour_boundary_rule
```

### v1 기본 제안

```text
year_boundary_rule  = LICHUN
month_boundary_rule = JIEQI
```

### 자시 날짜 변경

다음 두 방식을 모두 엔진에서 지원한다.

```text
MIDNIGHT_00
EARLY_ZI_23
```

최종 기본값은 명리 전문가 검증 후 확정한다.

이 항목은 출생시각이 23:00~00:59 부근인 사용자의 일주와 시주에 영향을 줄 수 있으므로 테스트 우선순위를 최고로 둔다.

---

## 7. Calendar Normalizer

입력이 음력이면 먼저 대응 양력 일자로 변환한다.

```text
Lunar Birth Date
    ↓
Solar Conversion
    ↓
Normalized Civil DateTime
```

저장 결과:

```text
solar_date
lunar_date
is_leap_month
julian_day
weekday
timezone_id
```

원본 입력과 변환 결과를 모두 보존한다.

---

## 8. 24절기 데이터

24절기는 독립 테이블로 관리한다.

```text
SolarTerm
├─ year
├─ term_index
├─ term_name
├─ event_datetime
├─ timezone_id
├─ source_version
└─ validation_status
```

월주 경계에 사용하는 12개 절입점은 별도로 표시한다.

| 월지 | 시작 절입 |
|---|---|
| 寅 | 입춘 |
| 卯 | 경칩 |
| 辰 | 청명 |
| 巳 | 입하 |
| 午 | 망종 |
| 未 | 소서 |
| 申 | 입추 |
| 酉 | 백로 |
| 戌 | 한로 |
| 亥 | 입동 |
| 子 | 대설 |
| 丑 | 소한 |

이 매핑은 `MONTH_BOUNDARY_JIEQI_V1` Rule Table로 관리한다.

---

## 9. 천간·지지 마스터

### 천간

```text
0 甲 갑  WOOD  YANG
1 乙 을  WOOD  YIN
2 丙 병  FIRE  YANG
3 丁 정  FIRE  YIN
4 戊 무  EARTH YANG
5 己 기  EARTH YIN
6 庚 경  METAL YANG
7 辛 신  METAL YIN
8 壬 임  WATER YANG
9 癸 계  WATER YIN
```

### 지지

```text
0 子 자 WATER
1 丑 축 EARTH
2 寅 인 WOOD
3 卯 묘 WOOD
4 辰 진 EARTH
5 巳 사 FIRE
6 午 오 FIRE
7 未 미 EARTH
8 申 신 METAL
9 酉 유 METAL
10 戌 술 EARTH
11 亥 해 WATER
```

각 master row에는 한글명, 한자, 음양, 오행, 정렬 순서, 버전을 저장한다.

---

## 10. 년주 계산

v1 명리 Rule의 기본 경계는 **입춘**으로 제안한다.

```text
if birth_datetime < lichun_datetime_of_year:
    pillar_year = solar_year - 1
else:
    pillar_year = solar_year
```

그 후 60갑자 index를 계산한다.

```text
stem_index   = cycle_index % 10
branch_index = cycle_index % 12
```

기준 연도와 offset은 하드코딩하지 않고 검증된 Reference Constant로 관리한다.

```text
GANJI_REFERENCE
├─ reference_date_or_year
├─ cycle_index
├─ source
└─ version
```

---

## 11. 월주 계산

월주는 음력 월 번호로 계산하지 않는다.

선택된 `month_boundary_rule`의 절입 시각을 기준으로 월지를 결정한다.

월지는 입춘부터 寅월로 시작한다.

월간의 첫 천간은 년간에 따라 결정하는 Rule Table을 사용한다.

| 년간 그룹 | 寅월 시작 천간 |
|---|---|
| 甲 / 己 | 丙 |
| 乙 / 庚 | 戊 |
| 丙 / 辛 | 庚 |
| 丁 / 壬 | 壬 |
| 戊 / 癸 | 甲 |

이후 월마다 천간을 한 단계씩 진행시킨다.

---

## 12. 일주 계산

일주는 연속된 60갑자 cycle로 계산한다.

구현은 Julian Day 계열의 연속 일수 값을 이용하되, 임의의 인터넷 예제 상수를 그대로 사용하지 않는다.

절차:

```text
Normalized Date
    ↓
Continuous Day Number
    ↓
Verified Ganji Offset
    ↓
Day Ganji Index
```

`Verified Ganji Offset`은 KASI 월별 음양력의 간지 결과와 충분한 날짜 표본으로 검증한 뒤 고정한다.

일주 검증 표본은 최소 다음 범위를 포함한다.

```text
평년
윤년
세기 경계
음력 윤달이 있는 해
입춘 전후
월말/연말
역사적 시간대 변경 구간
```

---

## 13. 시주 계산

출생시간을 아는 경우에만 계산한다.

시지는 12개 두 시간 구간으로 분리한다.

```text
子 23:00-00:59
丑 01:00-02:59
寅 03:00-04:59
卯 05:00-06:59
辰 07:00-08:59
巳 09:00-10:59
午 11:00-12:59
未 13:00-14:59
申 15:00-16:59
酉 17:00-18:59
戌 19:00-20:59
亥 21:00-22:59
```

23시 이후의 날짜 귀속은 `day_boundary_rule`에 따라 결정한다.

자시의 시작 천간은 일간 그룹으로 관리한다.

| 일간 그룹 | 子시 시작 천간 |
|---|---|
| 甲 / 己 | 甲 |
| 乙 / 庚 | 丙 |
| 丙 / 辛 | 戊 |
| 丁 / 壬 | 庚 |
| 戊 / 癸 | 壬 |

---

## 14. 지장간

지장간은 별도 master table로 관리한다.

```text
子 → 癸
丑 → 己 癸 辛
寅 → 甲 丙 戊
卯 → 乙
辰 → 戊 乙 癸
巳 → 丙 戊 庚
午 → 丁 己
未 → 己 丁 乙
申 → 庚 壬 戊
酉 → 辛
戌 → 戊 辛 丁
亥 → 壬 甲
```

지장간별 가중치는 학파 차이가 있으므로 master mapping과 weight rule을 분리한다.

```text
HiddenStem
HiddenStemWeightRule
```

v1 초기에는 존재 관계와 순서를 먼저 확정하고, 수치 가중치는 전문가 검증 전 사용자 결과에 직접 사용하지 않는다.

---

## 15. 오행 계산

오행 분석은 두 값을 분리한다.

### raw_element_count

팔자 여덟 글자의 단순 개수.

### weighted_element_score

지장간, 월령, 통근, 투간 등 Rule을 반영한 해석용 점수.

```text
FiveElementResult
├─ wood_raw
├─ fire_raw
├─ earth_raw
├─ metal_raw
├─ water_raw
├─ wood_weighted
├─ fire_weighted
├─ earth_weighted
├─ metal_weighted
├─ water_weighted
└─ weight_rule_version
```

단순 글자 개수와 실제 명리 해석용 강도를 같은 값으로 취급하지 않는다.

---

## 16. 십신 계산

십신은 일간(Day Master)을 기준으로 다른 천간의 오행 관계와 음양 관계로 결정한다.

| 일간과 대상 관계 | 같은 음양 | 다른 음양 |
|---|---|---|
| 같은 오행 | 비견 | 겁재 |
| 일간이 생함 | 식신 | 상관 |
| 일간이 극함 | 편재 | 정재 |
| 일간을 극함 | 편관 | 정관 |
| 일간을 생함 | 편인 | 정인 |

계산 함수 예:

```text
get_ten_god(day_master, target_stem)
```

지장간에 대해서도 동일한 함수를 사용한다.

---

## 17. 합·충·형·파·해 Engine

관계 계산은 별도 Rule Table 기반으로 만든다.

```text
RelationRule
├─ relation_type
├─ source_branch_or_stem
├─ target_branch_or_stem
├─ condition
├─ priority
├─ interpretation_id
└─ version
```

계산 결과는 단순 boolean이 아니라 근거를 남긴다.

예:

```text
{
  "relation": "CHONG",
  "left": "子",
  "right": "午",
  "source_positions": ["DAY_BRANCH", "YEAR_BRANCH"]
}
```

후속 궁합엔진과 운세엔진이 같은 결과를 재사용한다.

---

## 18. 신강·신약 Engine

신강·신약은 단순 오행 개수로 결정하지 않는다.

v1 설계에서는 다음 component를 독립적으로 산출한다.

```text
StrengthComponents
├─ seasonal_power
├─ root_support
├─ visible_support
├─ resource_support
├─ peer_support
├─ output_drain
├─ wealth_drain
├─ officer_control
└─ special_structure_flags
```

그 후 `STRENGTH_RULE_VERSION`이 component를 조합한다.

전문가 검수 전에는 임의의 0~100 점수를 사용자에게 노출하지 않는다.

결과는 내부적으로 다음 상태를 지원한다.

```text
STRONG
SLIGHTLY_STRONG
BALANCED
SLIGHTLY_WEAK
WEAK
SPECIAL
UNDETERMINED
```

---

## 19. 격국

격국은 P1 심화 기능으로 구현한다.

엔진 구조:

```text
PatternDetector
    ↓
Candidate Patterns
    ↓
Conflict Resolution
    ↓
Selected Pattern
```

결과에는 반드시 다음을 남긴다.

```text
pattern_name
evidence
rule_version
confidence_state
```

`confidence_state`는 AI 확률이 아니라 Rule 충족 정도를 나타내는 상태값으로 사용한다.

---

## 20. 용신·희신·기신

이 영역은 해석 방식의 차이가 크므로 v1에서 계산 interface는 만들되, 전문가 검증 전에는 정식 사용자 결과의 필수값으로 사용하지 않는다.

```text
YongsinEngine
├─ method
├─ selected_elements
├─ supporting_evidence
├─ conflict_flags
└─ rule_version
```

지원 가능한 method 구조 예:

```text
BALANCING
SEASONAL
BRIDGING
SPECIAL_STRUCTURE
```

최종 채택 방법과 우선순위는 명리 전문가 검토 사항으로 남긴다.

---

## 21. 신살 Engine

신살은 P2이다.

모든 신살을 한 번에 구현하지 않는다.

```text
ShinsalRule
├─ shinsal_id
├─ required_inputs
├─ condition
├─ display_level
├─ caution_level
└─ version
```

불안을 과도하게 유발하는 신살은 계산 가능하더라도 사용자 화면에서 숨길 수 있도록 `display_level`을 둔다.

---

## 22. 대운 Engine

대운은 다음을 분리한다.

```text
Daewoon
├─ direction
├─ start_datetime_basis
├─ start_age
├─ first_pillar
├─ periods
└─ rule_version
```

### 대운 순행/역행

성별 또는 명리 계산 구분값과 년간 음양을 사용하는 방식 등 학파별 차이가 있으므로 `DAEWOON_DIRECTION_RULE`로 분리한다.

### 대운 시작 나이

인접 절입점과 출생시각의 시간 차이를 이용하는 방식을 지원하되 환산 공식은 독립 Rule로 둔다.

```text
DAEWOON_START_RULE
```

전문가 검증 전에 하나의 공식을 절대값처럼 고정하지 않는다.

---

## 23. 세운·월운·일운

운세 계산에서 새로운 원국을 만들지 않는다.

기본 원국은 불변이며 해당 기간의 간지를 `transit`으로 추가한다.

```text
Natal Chart
+
Year Transit
+
Month Transit
+
Day Transit
```

이를 통해 원국과 운의 상호작용을 분석한다.

### 세운

년주 Rule과 같은 연 경계 규칙을 사용한다.

### 월운

월주와 같은 절입 경계를 사용한다.

### 일운

일주 계산 함수를 그대로 재사용한다.

---

## 24. Fortune Factor Output

운세엔진은 처음부터 “오늘 78점”을 직접 만들지 않는다.

먼저 근거 Factor를 생성한다.

```text
FortuneFactors
├─ favorable_relations
├─ unfavorable_relations
├─ ten_god_activation
├─ element_balance_effect
├─ natal_transit_relations
├─ risk_flags
├─ opportunity_flags
└─ evidence
```

이 Factor를 Interpretation Engine이 재물·직업·사랑·관계 등 사용자 영역으로 변환한다.

점수화는 별도 `FORTUNE_SCORING_VERSION`에서 관리한다.

---

## 25. Canonical Output JSON

계산엔진의 최종 결과는 UI 문장이 아니라 구조화된 JSON이다.

예:

```json
{
  "engine_version": "1.0.0",
  "rule_profile": "MONA_SAJU_KR_001",
  "input": {
    "calendar_type": "SOLAR",
    "birth_time_known": true,
    "timezone_id": "Asia/Seoul"
  },
  "normalized": {
    "solar_date": "1990-01-01",
    "lunar_date": null,
    "adjusted_saju_datetime": "1990-01-01T12:00:00+09:00"
  },
  "chart": {
    "year": {},
    "month": {},
    "day": {},
    "hour": {}
  },
  "five_elements": {},
  "ten_gods": {},
  "relations": [],
  "strength": {},
  "fortune": {
    "daewoon": [],
    "year": null,
    "month": null,
    "day": null
  },
  "quality": {
    "birth_time_known": true,
    "partial_chart": false,
    "warnings": []
  }
}
```

UI, 모나 AI, 궁합, Color/Gem은 이 canonical output만 사용한다.

---

## 26. 출생시간 모름 처리

출생시간을 모르는 사용자는 앱에서 배제하지 않는다.

```text
년주 = 계산
월주 = 계산
일주 = 계산
시주 = UNKNOWN
```

시주 의존 요소는 다음 상태를 반환한다.

```text
NOT_AVAILABLE_WITHOUT_BIRTH_TIME
```

화면에서는 다음처럼 설명한다.

> 출생시간을 몰라도 기본 사주와 운세를 볼 수 있어요. 다만 시주가 필요한 일부 상세 분석은 제외됩니다.

임의의 정오나 자정을 입력해 시주를 만들어내지 않는다.

---

## 27. Rule Version 구조

```text
RuleProfile
├─ profile_id
├─ calendar_rule_version
├─ timezone_rule_version
├─ solar_time_rule_version
├─ year_boundary_version
├─ month_boundary_version
├─ day_boundary_version
├─ pillar_rule_version
├─ hidden_stem_version
├─ ten_god_version
├─ relation_version
├─ strength_version
├─ pattern_version
├─ yongsin_version
├─ daewoon_version
└─ fortune_version
```

예:

```text
MONA_SAJU_KR_001
```

Rule이 바뀌어도 과거 결과는 당시 version으로 재현 가능해야 한다.

---

## 28. 계산 Trace

모든 중요 계산에는 Trace를 남긴다.

예:

```text
YEAR_PILLAR
input = 2026-02-03 22:00
lichun = ...
boundary = LICHUN
selected_cycle_year = ...
result = ...
```

Trace는 사용자에게 전부 보여주지 않지만 QA, 전문가 검수, 오류 분석에 사용한다.

---

## 29. 검증 전략

사주 엔진은 일반적인 unit test만으로 충분하지 않다.

### Golden Test

한국천문연구원 자료와 비교한다.

검증:

- 양력 ↔ 음력
- 윤달
- 간지
- 24절기
- 날짜 경계

### Boundary Test

특히 다음을 집중 테스트한다.

```text
입춘 직전 / 직후
절입 직전 / 직후
22:59
23:00
23:59
00:00
00:01
00:59
01:00
윤년 2월 29일
음력 윤달
12월 31일 / 1월 1일
```

### Historical Time Test

지역과 연도에 따른 역사적 표준시 변화가 적용되는지 확인한다.

### Deterministic Test

같은:

```text
input + rule_version
```

에 대해 모든 환경에서 byte-level로 같은 canonical 결과가 생성되는지 확인한다.

---

## 30. 최소 검증 케이스 세트

정식 개발 시작 전에 최소 다음 유형의 테스트 케이스를 확보한다.

1. 일반 양력 + 출생시간 있음
2. 일반 음력 + 출생시간 있음
3. 음력 윤달
4. 출생시간 모름
5. 입춘 당일 입춘 전
6. 입춘 당일 입춘 후
7. 월 절입 전/후
8. 23시 경계
9. 00시 경계
10. 해외 출생 / 다른 timezone
11. 한국 역사적 표준시 구간
12. 윤년 2월 29일
13. 대운 순행 케이스
14. 대운 역행 케이스
15. 전문가가 수작업으로 확인한 사주 원국

각 케이스는 예상 결과와 Rule Version을 같이 저장한다.

---

## 31. 오류 및 Quality Flag

결과에는 상태를 함께 반환한다.

```text
VALID
PARTIAL
RULE_AMBIGUOUS
OUT_OF_VALIDATED_RANGE
INVALID_INPUT
SOURCE_VALIDATION_FAILED
```

예:

출생시간 모름:

```text
status = PARTIAL
```

검증 범위를 벗어난 날짜:

```text
status = OUT_OF_VALIDATED_RANGE
```

결과를 억지로 생성해 정상 데이터처럼 보여주지 않는다.

---

## 32. 사주엔진과 Interpretation Engine의 경계

### Saju Engine이 반환

- 팔자
- 오행
- 십신
- 지장간
- 합충형파해
- 신강약 component
- 격국 candidate
- 운의 간지
- 원국과 운의 관계
- 근거

### Interpretation Engine이 담당

- 성격
- 재물
- 직업
- 연애
- 결혼
- 학업
- 관계
- 오늘의 조언
- 좋은 흐름
- 조심할 흐름

### Mona AI가 담당

- 사용자의 현재 고민 이해
- 운세 결과를 자연스럽게 설명
- 과거 히스토리 연결
- 사용자의 마음에 맞는 표현
- 현실적인 작은 행동 제안

---

## 33. Color / Gem 연결

Color와 Gem은 사주 계산엔진이 직접 추천하지 않는다.

```text
Saju Engine
    ↓
Five Element / Fortune Factors
    ↓
Fashion Rule Engine
    ↓
Color Recommendation
Gem Recommendation
```

따라서 색·보석 매핑을 바꾸더라도 사주 원국에는 영향을 주지 않는다.

---

## 34. 궁합 연결

궁합도 두 사주의 계산 결과를 다시 계산하지 않는다.

```text
Person A Canonical Chart
+
Person B Canonical Chart
    ↓
Compatibility Engine
```

각 개인의 원국은 동일한 `rule_profile`로 계산되어야 한다.

Rule Profile이 다르면 궁합 계산을 차단하거나 동일 버전으로 재계산한다.

---

## 35. 모나 v1 개발 패키지 구조 제안

플랫폼에 독립적인 논리 구조:

```text
mona_saju/
├─ calendar/
│  ├─ converter
│  ├─ solar_terms
│  └─ timezone
├─ masters/
│  ├─ stems
│  ├─ branches
│  ├─ hidden_stems
│  └─ relations
├─ pillars/
│  ├─ year
│  ├─ month
│  ├─ day
│  └─ hour
├─ factors/
│  ├─ five_elements
│  ├─ ten_gods
│  ├─ relations
│  ├─ strength
│  ├─ pattern
│  ├─ yongsin
│  └─ shinsal
├─ fortune/
│  ├─ daewoon
│  ├─ yearly
│  ├─ monthly
│  └─ daily
├─ rules/
├─ validation/
├─ tests/
└─ api/
```

실제 프로그래밍 언어와 프레임워크를 결정해도 이 논리 구조는 유지한다.

---

## 36. v1에서 우선 구현할 범위

### Engine Core

- 입력 정규화
- 양력/음력 변환
- timezone 처리
- 절기
- 년주
- 월주
- 일주
- 시주
- 천간/지지 master
- 지장간
- 오행
- 십신
- 기본 관계
- 대운 interface
- 세운/월운/일운
- canonical JSON
- Rule Version
- Trace
- 자동 테스트

### Expert Review 이후 확정

- 자시 날짜 변경 기본값
- 진태양시 기본 적용 여부
- 신강약 가중치
- 격국 우선순위
- 용신 산출법
- 대운 순역 규칙
- 대운 시작 나이 환산법
- 채택 신살 목록

---

## 37. 개발 승인 Gate

사주 엔진은 다음 조건을 만족한 후에만 UI와 본격 연결한다.

```text
CALENDAR_VALIDATION_PASS
SOLAR_TERM_BOUNDARY_PASS
YEAR_PILLAR_PASS
MONTH_PILLAR_PASS
DAY_PILLAR_PASS
HOUR_PILLAR_PASS
LUNAR_LEAP_MONTH_PASS
UNKNOWN_TIME_PASS
TIMEZONE_PASS
RULE_VERSION_PASS
DETERMINISTIC_REPLAY_PASS
```

하나라도 실패하면 계산엔진은 `development_only` 상태를 유지한다.

---

## 38. 다음 작업

다음 단계는 **모나 앱 IA + 화면별 상세 명세 v1**이다.

사주 엔진과 화면을 직접 결합하지 않고 다음 흐름으로 설계한다.

```text
Saju Engine
    ↓
Application Service
    ↓
Screen View Model
    ↓
UI
```

따라서 다음 설계에서는 다음 화면을 정의한다.

```text
Splash / Onboarding
프로필
생년월일 입력
오늘
사주
상세 사주
대운
운세
궁합
모나 상담
재미
타로
관상
꿈
부적
나의 패션
히스토리
설정
```

---

# 39. v1 핵심 결정 요약

| 항목 | v1 방향 |
|---|---|
| 공식 달력 검증 | KASI 기준 |
| 앱 계산 방식 | Local deterministic engine |
| 시간대 | IANA timezone |
| 사주 연 경계 | 입춘 기본 제안 |
| 월 경계 | 절입 기준 |
| 음력 입력 | 양력으로 정규화 후 계산 |
| 출생시간 모름 | 시주 NULL, 부분 분석 |
| 진태양시 | Rule로 분리 |
| 자시 날짜 경계 | 23시/0시 방식 모두 지원 후 전문가 확정 |
| 오행 | raw와 weighted 분리 |
| 십신 | 일간 기준 결정적 계산 |
| 신강약 | component 기반, 전문가 가중치 |
| 용신 | interface 우선, 전문가 검증 후 사용자 노출 |
| 대운 | 순역·시작나이 Rule 분리 |
| AI 역할 | 계산 금지, 해석·개인화 담당 |
| 재현성 | input + Rule Version으로 완전 재현 |
| 검증 | Golden + Boundary + Deterministic Test |

---

## 참고 기준

- 한국천문연구원(KASI) 음양력 변환 및 월력/역서 자료
- 한국천문연구원 24절기/특일 정보
- 한국천문연구원 연간 역서·월력요항
- IANA Time Zone Database

---

**MONA Saju Calculation Engine Design v1.0**  
**Next: MONA App IA & Screen Specification v1**
