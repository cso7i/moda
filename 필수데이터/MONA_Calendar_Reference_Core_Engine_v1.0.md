# MONA Calendar Reference Dataset + Saju Core Engine v1.0

**상태:** Development Seed / Core Engine v0.1 Acceptance PASS  
**목적:** 모나가 인터넷 조회 없이 양력·음력·일진·24절기 경계를 읽고, 명리 전문가가 최종 Rule을 승인하기 전에도 재현 가능한 사주 원국 계산을 테스트할 수 있는 기반을 만든다.

---

## 1. 이번 단계에서 실제 확보한 것

이번 단계에서는 설계 문서만 만든 것이 아니라 **실제 로컬 Reference DB와 실행 가능한 사주 Core Engine**을 생성했다.

### Calendar Reference DB

`mona_calendar_reference_seed_1900_2050_v1.sqlite3`

포함 데이터:

- 양력 날짜: `1900-01-01 ~ 2050-12-31`
- 전체 일수: **55,152일**
- 각 날짜의 한국 음력 연·월·일
- 윤달 여부
- 일진(日干支) / 60갑자 index
- 24절기 정확 시각용 table
- 절기 계산 보조 범위: `1899 ~ 2051`
- 24절기 row: **3,672건**
- Source metadata
- Validation 결과

24절기는 본 계산범위 양끝의 월주 경계를 안정적으로 처리하기 위해 본 범위보다 앞뒤 1년을 추가하였다.

---

## 2. 중요: 현재 DB의 등급

이 DB는 현재:

```text
DEVELOPMENT_SEED
```

이다.

즉 **개발과 전문가 검증을 시작하기에 충분한 데이터팩**이지만 아직 이름을 `KASI 공식 canonical DB`라고 부르지는 않는다.

Production 승격 전에는 다음이 필요하다.

1. KASI 공식 자료와 더 넓은 연도 표본 교차검증
2. 절기 경계 다년 검증
3. 윤달/연말/윤년/역법 경계 검증
4. 명리 전문가의 연·월·일·시주 Rule 승인
5. Rule Version 고정

---

## 3. 양력↔음력 Reference 구축 방식

개발 Seed는 `korean-lunar-calendar 0.3.1`의 내부 한국 음력 월 길이 데이터를 이용해 생성했다.

Upstream 프로젝트는 자신의 변환표가 한국천문연구원(KASI/KARI)의 한국 음양력 기준을 따른다고 명시하고 있다.

다만 모나는 이 라이브러리를 맹신하지 않는다.

그래서 실제 DB에는:

```text
source_id
source_version
validation_result
```

를 함께 저장했다.

Runtime 앱은 이 외부 Python library를 직접 호출하지 않아도 된다.

---

## 4. KASI 음양력 Spot Check

KASI의 2026년 8월 월별음양력 표에 있는 **31일 전체**를 비교했다.

검증 항목:

- 양력 날짜
- 음력 연/월/일
- 윤달 여부
- 일진

결과:

```text
VAL_KASI_202608_MONTHLY_001
31 / 31 PASS
```

대표:

```text
2026-08-10
KASI 음력: 2026-06-28
KASI 일진: 丙辰
MONA Seed: 2026-06-28 / 丙辰
PASS
```

---

## 5. 매우 중요한 구분 — KASI 음력간지 vs 사주 월주

KASI 월별음양력 화면의 `음력간지`에 표시되는 **연·월 간지**를 모나의 사주 년주·월주로 그대로 사용하지 않는다.

예:

```text
2026-08-10
KASI 월별음양력 표시:
丙午년 乙未월 丙辰일
```

하지만 사주의 월주는 음력 초하루가 아니라 **절입(Jieqi) 경계**를 사용하는 후보 Rule이므로, 입추 이후인 이 날짜의 사주 월주 Candidate는:

```text
丙申
```

으로 계산된다.

따라서:

- KASI 음양력 DB → 음력 변환 + 일진 Reference
- MONA Saju Rule → 입춘/절입 기준 년주·월주

로 책임을 분리한다.

---

## 6. 24절기 Reference

24절기 table은 각 절기의 **UTC instant**를 canonical 값으로 저장한다.

Schema:

```text
term_year
term_id
solar_longitude_deg
instant_utc
seoul_local_datetime
source_id
source_version
validation_state
```

Runtime 비교는 UTC instant로 수행한다.

이 방식의 장점:

- 해외 출생 처리 가능
- 역사적 timezone 처리 가능
- 특정 지역의 local time과 절기 instant를 정확히 비교 가능
- KST `+09:00` 상수에 의존하지 않음

---

## 7. 절기 계산 검증

Build 단계의 astronomical calculation은 `pyswisseph 2.10.3.2`를 이용했다.

하지만 **MONA runtime dependency는 아니다.**

계산한 절기 값을 SQLite에 고정하고 앱은 SQLite table만 읽는다.

KASI 2028 달력자료에 표시된 24절기의 날짜·시·분 전체와 비교했다.

결과:

```text
VAL_KASI_2028_SOLAR_TERMS_001
24 / 24 PASS
```

KASI 표시는 초를 제공하지 않으므로 KASI의 분 단위 timestamp와 계산 시각의 차이가 60초 이하인지 확인했다.

최대 차이:

```text
30.54825 seconds
```

즉 24개 모두 KASI가 표시한 같은 분 또는 해당 분 timestamp 기준 1분 이내에 위치했다.

---

## 8. Saju Core Engine

파일:

`mona_saju_core_engine_v0_1.py`

현재 엔진은 외부 API를 호출하지 않는다.

Runtime 기본 의존성:

```text
Python standard library
SQLite reference DB
IANA zoneinfo
```

### 현재 구현

- SOLAR 입력
- LUNAR 입력
- 윤달 lookup
- timezone 적용
- 입춘 기준 년주 Candidate
- 절입 기준 월주 Candidate
- 일주
- 시주
- `MIDNIGHT_00`
- `EARLY_ZI_23`
- 출생시간 모름 처리
- 절기 경계일 출생시간 모름 → ambiguity
- 오행 raw count
- 천간 십신
- 지장간 + 지장간 십신
- canonical JSON

---

## 9. 2026-08-10 Sample

입력:

```text
양력 2026-08-10
12:00
Asia/Seoul
CIVIL_TIME
MIDNIGHT_00
```

현재 Candidate Rule 결과:

```text
년주  丙午
월주  丙申
일주  丙辰
시주  甲午
```

음력 정규화:

```text
2026-06-28
```

이 결과는 `mona_saju_core_sample_20260810.json`에 저장했다.

---

## 10. 입춘 경계 테스트

2028년 입춘 instant를 기준으로 1초 전/후를 계산했다.

입춘 전:

```text
년주 丁未
월주 癸丑
```

입춘 후:

```text
년주 戊申
월주 甲寅
```

따라서 날짜만 보고 `2월 4일이면 무조건 새 년주`처럼 처리하지 않는다.

**정확한 절기 instant**를 비교한다.

---

## 11. 출생시간 모름 처리

일반 날짜:

```text
status = PARTIAL_NO_BIRTH_TIME
년주 = 계산 가능
월주 = 계산 가능
일주 = 계산 가능
시주 = null
```

절기 경계일:

```text
status = PARTIAL_BOUNDARY_AMBIGUOUS
```

예를 들어 출생일이 입춘일인데 시간을 모르면:

- 입춘 전 년주/월주
- 입춘 후 년주/월주

가 다를 수 있으므로 임의로 하나를 선택하지 않는다.

사용자에게 출생시간에 따라 결과가 달라질 수 있음을 안내한다.

---

## 12. 자시 날짜 경계

두 후보 Rule을 모두 구현했다.

```text
MIDNIGHT_00
EARLY_ZI_23
```

예:

```text
2026-08-10 23:30
```

`MIDNIGHT_00`

```text
일주 丙辰
시주 戊子
```

`EARLY_ZI_23`

```text
일주 丁巳
시주 庚子
```

어느 것을 MONA 정식 기준으로 사용할지는 전문가 검수 Sheet에서 최종 승인한다.

---

## 13. 진태양시

현재 v0.1에서는:

```text
solar_time_mode = CIVIL_TIME
```

만 실행한다.

`TRUE_SOLAR_TIME`은 interface는 유지하지만 **전문가 Rule 승인 전 활성화하지 않았다.**

즉 진태양시를 어설프게 자동 적용하지 않는다.

---

## 14. 십신

십신 계산은 일간과 대상 천간의:

```text
오행 관계
+
음양 관계
```

로 결정한다.

구현 ID:

```text
ten_gods.peer
ten_gods.rob_wealth
ten_gods.eating_god
ten_gods.hurting_officer
ten_gods.indirect_wealth
ten_gods.direct_wealth
ten_gods.seven_killings
ten_gods.direct_officer
ten_gods.indirect_resource
ten_gods.direct_resource
```

따라서 다국어 UI 문자열은 Core Engine에 들어 있지 않다.

Terminology Registry에서 표시한다.

---

## 15. 오행

현재는 **raw visible element count**만 확정 계산한다.

```text
천간 4글자
+
지지 4글자
=
최대 8 visible elements
```

지장간 가중치, 월령, 통근, 투간을 반영하는 weighted element score는 아직 계산하지 않는다.

이 부분은 전문가 검증 후 별도 Rule Version으로 승격한다.

---

## 16. 지장간

표준 후보 mapping은 Engine에 포함했지만:

```text
weight = not applied
```

이다.

현재 반환하는 것은:

- 숨은 천간
- 해당 오행
- 일간 기준 십신

까지다.

지장간 가중치는 전문가 검토 전 점수에 사용하지 않는다.

---

## 17. 아직 구현하지 않은 핵심 영역

현재 Core v0.1이 임의로 확정하지 않은 항목:

- 진태양시
- 신강/신약 weighted judgement
- 격국
- 용신
- 희신
- 기신
- 신살 최종 채택
- 대운 순/역
- 대운 시작나이
- 대운 세부 Rule
- Interpretation

이들은 전문가 승인 후 독립 Rule Layer로 추가한다.

---

## 18. Self Test

파일:

`test_mona_saju_core_v0_1.py`

검증:

```text
2026-08-10 sample
SOLAR/LUNAR 동일 결과
2028 입춘 1초 전/후
출생시간 모름
절기 경계 ambiguity
MIDNIGHT_00
EARLY_ZI_23
2050 마지막 날짜 edge
```

결과:

```text
SELFTEST_MONA_SAJU_CORE_V0_1_PASS
```

---

## 19. 생성 파일

### 개발 DB

`mona_calendar_reference_seed_1900_2050_v1.sqlite3`

### 데이터 Export

`mona_calendar_days_1900_2050_v1.csv.gz`

### 24절기 Export

`mona_solar_terms_1900_2050_v1.csv`

실제 DB에는 경계 cushion을 위해 1899/2051 절기도 포함한다.

### 검증

`mona_calendar_validation_v1.json`

### Manifest / SHA-256

`mona_calendar_reference_manifest_v1.json`

### 재생성 Script

`build_mona_calendar_reference_v1.py`

### Core Engine

`mona_saju_core_engine_v0_1.py`

### Test

`test_mona_saju_core_v0_1.py`

### Sample Output

`mona_saju_core_sample_20260810.json`

---

## 20. Production 승격 절차

현재:

```text
DEVELOPMENT_SEED
```

다음:

```text
KASI_BROAD_VALIDATION
↓
EXPERT_RULE_APPROVAL
↓
EDGE_CASE_ACCEPTANCE
↓
MONA_CALENDAR_REFERENCE_001
↓
MONA_SAJU_RULE_001
```

Production에서는 filename/version이 아니라 DB의 manifest와 Rule Version을 canonical 기준으로 사용한다.

---

## 21. 다음 작업

이제 명리 전문가가 데이터를 일일이 입력할 필요는 없다.

다음 단계는 **전문가 Rule Decision Pack**을 실제 Core Engine에 연결하는 것이다.

우선 검토 순서:

1. `year_boundary_rule = LICHUN` 승인 여부
2. `month_boundary_rule = JIEQI` 승인 여부
3. `MIDNIGHT_00 vs EARLY_ZI_23`
4. `CIVIL_TIME vs TRUE_SOLAR_TIME`
5. 지장간 mapping / weight
6. 신강약 Rule
7. 대운 순역
8. 대운 시작나이
9. 용신 체계
10. 신살 채택

전문가가 1~4를 먼저 결정하면 **사주 원국 계산 Rule v1**을 상당 부분 고정할 수 있다.

---

# Acceptance

```text
CALENDAR_REFERENCE_BUILD_PASS
CALENDAR_DAY_55152_PASS
SOLAR_TERM_3672_PASS
KASI_202608_31_OF_31_PASS
KASI_2028_SOLAR_TERM_24_OF_24_PASS
SOLAR_LUNAR_ROUNDTRIP_SAMPLE_PASS
LICHUN_BOUNDARY_PASS
UNKNOWN_TIME_BOUNDARY_PASS
DAY_BOUNDARY_DUAL_RULE_PASS
SELFTEST_MONA_SAJU_CORE_V0_1_PASS
```

---

**MONA Calendar Reference Dataset + Saju Core Engine v1.0**  
**Current runtime engine: v0.1.0**  
**Status: Development Seed / Expert Rule Approval Pending**
