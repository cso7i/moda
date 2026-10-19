# 모나(MONA) 앱 IA + 화면별 상세 명세 v1.0

**문서 상태:** Product IA / Screen Specification Draft  
**기준 문서:** `MONA_사주계산엔진_설계_v1.0.md`, `MONA_다국어_용어표준집_v1.0.md`, `mona_handoff.md`  
**기본 locale:** `ko-KR`  
**기본 준비 locale:** `en`, `ja-JP`, `zh-CN`, `fr-FR`  
**후속 확장:** `zh-TW`, `zh-HK`

---

# 1. 문서 목적

본 문서는 모나(MONA)의 전체 정보구조(IA), 화면 목록, 화면 간 이동, 입력/출력, 상태, 예외처리, 다국어 처리 원칙을 실제 개발 가능한 수준으로 정의한다.

핵심 목표는 다음과 같다.

1. 사용자에게는 단순하고 따뜻한 흐름을 제공한다.
2. 내부적으로는 사주 계산엔진, Interpretation Engine, Mona AI Context Engine을 분리한다.
3. 모든 화면은 중앙 Terminology Registry와 locale resource를 사용한다.
4. 운세 결과는 공포를 유발하지 않고 행동 제안으로 연결한다.
5. 관상은 본인 사진만 허용하며 꾸민 본인 사진은 허용한다.
6. 타로·관상·꿈·부적·패션은 `재미` 영역에 둔다.
7. 히스토리는 장기적인 사용자 경험의 핵심으로 유지한다.

---

# 2. 전체 IA

```text
APP
├─ Splash
├─ Onboarding
├─ Profile Setup
│  ├─ Basic Info
│  ├─ Birth Info
│  ├─ Birth Time
│  ├─ Birth Place
│  └─ Consent / Privacy
│
├─ Main Navigation
│  ├─ TODAY
│  │  ├─ Today Home
│  │  ├─ Mood Check
│  │  ├─ Concern Check
│  │  ├─ Today Reading
│  │  ├─ Reading Detail
│  │  └─ Ask Mona
│  │
│  ├─ FORTUNE
│  │  ├─ Fortune Hub
│  │  ├─ Saju
│  │  │  ├─ Four Pillars
│  │  │  ├─ Five Elements
│  │  │  ├─ Ten Gods
│  │  │  ├─ Relations
│  │  │  ├─ Structure
│  │  │  └─ Daewoon Timeline
│  │  ├─ Period Fortune
│  │  │  ├─ Daily
│  │  │  ├─ Weekly
│  │  │  ├─ Monthly
│  │  │  ├─ Yearly
│  │  │  └─ Life Path
│  │  ├─ Compatibility
│  │  ├─ Tojeong Bigyeol
│  │  ├─ Name Reading
│  │  ├─ Zodiac
│  │  └─ Intuitive Reading
│  │
│  ├─ MONA
│  │  ├─ Mona Home
│  │  ├─ Chat Session
│  │  ├─ Context Selector
│  │  ├─ Past Context
│  │  └─ Session Summary
│  │
│  ├─ FUN
│  │  ├─ Fun Hub
│  │  ├─ Tarot
│  │  ├─ Face Reading
│  │  ├─ Dream Reading
│  │  ├─ Lucky Talisman
│  │  ├─ My Style
│  │  │  ├─ Lucky Color
│  │  │  ├─ Gem
│  │  │  └─ Situation Style
│  │  └─ Lucky Draw
│  │
│  └─ ME
│     ├─ My Profile
│     ├─ Relationship Profiles
│     ├─ History
│     ├─ Favorites
│     ├─ Language
│     ├─ Privacy
│     └─ Settings
```

---

# 3. 하단 네비게이션

하단 메뉴는 5개로 고정한다.

| ID | 메뉴 | 역할 |
|---|---|---|
| `nav.today` | 오늘 | 매일 진입하는 메인 홈 |
| `nav.fortune` | 운세 | 사주와 전문 운세 |
| `nav.mona` | 모나 | AI 마음지키미 |
| `nav.fun` | 재미 | 타로, 관상, 꿈, 부적, 패션 |
| `nav.me` | 나 | 프로필, 히스토리, 설정 |

### 디자인 원칙

- 중앙 `모나` 버튼은 시각적으로 강조할 수 있다.
- 선택된 탭은 텍스트 + 아이콘으로 명확히 표시한다.
- 아이콘만으로 의미를 전달하지 않는다.
- 다국어에서 길이가 늘어날 수 있으므로 하단 라벨은 locale별 short label을 사용한다.

---

# 4. 공통 화면 구성

모든 화면은 가능한 한 아래 구성요소를 재사용한다.

```text
App Bar
Content
Primary Action
Secondary Action
Bottom Navigation
```

### 공통 컴포넌트

- `MonaGreetingCard`
- `FortuneScoreCard`
- `DomainScoreCard`
- `TodayActionCard`
- `CautionCard`
- `ElementBar`
- `PillarGrid`
- `TenGodChip`
- `TimelineCard`
- `HistoryCard`
- `MonaMessageBubble`
- `ContextChip`
- `FunContentCard`
- `PrivacyNoticeCard`
- `LocaleAwareTermLabel`

---

# 5. Screen ID 규칙

화면 ID는 다음처럼 관리한다.

```text
ONB-xxx   Onboarding
PRO-xxx   Profile
TOD-xxx   Today
FOR-xxx   Fortune
SAJ-xxx   Saju
COM-xxx   Compatibility
MON-xxx   Mona
FUN-xxx   Fun
HIS-xxx   History
SET-xxx   Settings
```

화면명을 코드에 직접 사용하지 않고 Screen ID를 기준으로 analytics, route, QA를 연결한다.

---

# 6. ONB-001 Splash

## 목적

앱 진입 시 브랜드 표시 및 초기 상태 확인.

## 표시

- MONA 로고
- 짧은 브랜드 문구

예:

> 오늘의 마음을 지켜주는 모나

## 처리

- locale resource load
- terminology registry version 확인
- profile 존재 여부 확인
- 첫 실행 여부 확인

## 이동

```text
첫 실행 → ONB-002
기존 사용자 → TOD-001
```

---

# 7. ONB-002 Onboarding 1 — 모나 소개

## 핵심 문장

> 행운은 조금 더 즐겁게, 불운은 조금 더 조심스럽게.

## 설명

- 모나는 운세를 통해 오늘의 마음과 선택을 돌아보도록 돕는다.
- 미래를 단정하지 않는다.
- 중요한 현실적 결정을 대신하지 않는다.

## Action

- `시작하기`
- `이미 사용 중이에요` (계정 기반 동기화 도입 시)

---

# 8. ONB-003 Onboarding 2 — 개인정보 안내

설명:

- 생년월일과 출생정보가 사주 계산에 사용됨
- 상담과 마음 기록은 사용자 경험 개인화에 사용
- 관상은 본인 사진만 가능
- 원본 얼굴 사진은 가능한 한 분석 후 보관하지 않음

Action:

- `확인했어요`
- `개인정보 자세히 보기`

---

# 9. PRO-001 프로필 시작

## 질문

> 누구의 사주를 볼까요?

초기 가입에서는 기본값:

- `나`

향후 관계 프로필에서 추가:

- 배우자
- 연인
- 가족
- 친구
- 기타

---

# 10. PRO-002 기본 정보

## 입력

- 이름 또는 닉네임
- 명리 계산에 필요한 구분 정보

## 원칙

표현상 성별 고정관념을 만들지 않는다.

필요한 계산 규칙이 성별 구분값을 요구하는 경우:

```text
calculation_gender_code
```

처럼 내부 필드로 분리한다.

---

# 11. PRO-003 생년월일

## 입력

- 양력 / 음력
- 년 / 월 / 일
- 음력 선택 시 윤달 여부

## Validation

- 존재하지 않는 날짜 차단
- 엔진 검증 범위 밖이면 안내
- 음력 윤달 조합 검증

## 저장

display text가 아니라 canonical date + calendar enum 저장.

---

# 12. PRO-004 출생시간

선택:

- 시간을 알아요
- 시간을 몰라요

### 시간을 알아요

- 시
- 분

### 시간을 몰라요

`birth_time_known=false`

화면 안내:

> 출생시간을 몰라도 기본 사주와 운세는 볼 수 있어요. 다만 시주가 필요한 일부 상세 분석은 제외됩니다.

---

# 13. PRO-005 출생지역

## 입력

- 국가
- 도시/지역

자동 저장:

- latitude
- longitude
- timezone_id

향후 진태양시 적용 시 별도 계산에 사용.

---

# 14. PRO-006 프로필 확인

요약:

- 이름
- 생년월일
- 양/음력
- 출생시간
- 출생지역

Action:

- `사주 만들기`
- `수정`

`사주 만들기` 선택 시 Saju Engine 호출.

---

# 15. PRO-007 계산 상태

## State

```text
CALCULATING
SUCCESS
PARTIAL
ERROR
```

### PARTIAL

예:

- 출생시간 모름

### ERROR

예:

- 날짜 변환 실패
- 검증 범위 밖
- Rule mismatch

오류가 있어도 AI가 임의 계산하여 대체하지 않는다.

---

# 16. TOD-001 오늘 홈

모나의 가장 중요한 메인 화면.

## 상단

시간대에 따라:

- 좋은 아침이에요
- 좋은 오후예요
- 편안한 저녁이에요

사용자 이름을 과도하게 반복하지 않는다.

## 섹션 1 — 오늘 마음

`MoodCheckCard`

아직 기록하지 않았으면:

> 오늘 마음은 어때요?

기록했으면:

> 오늘은 ‘조금 지침’으로 기록했어요.

## 섹션 2 — 오늘의 모나

- 종합 흐름
- 오늘 한마디
- 주요 영역 카드
- 오늘의 행동

## 섹션 3 — 오늘의 행운

- Color
- 숫자
- 시간
- 아이템

## 섹션 4 — 빠른 메뉴

- 사주
- 이번 주
- 궁합
- 나의 패션
- 타로
- 꿈해몽

---

# 17. TOD-002 마음 체크

## 기본 감정

- 괜찮아요
- 편안해요
- 기대돼요
- 그냥 그래요
- 지쳤어요
- 걱정돼요
- 화나요
- 답답해요
- 외로워요
- 이야기하고 싶어요

## 기능

- 하나 또는 복수 선택 가능 여부는 UX 테스트 후 결정
- intensity 1~5는 v2 옵션
- 자유 메모는 선택

## 저장

```text
mood_id
mood_term_id
created_at
note
```

---

# 18. TOD-003 고민 선택

질문:

> 오늘 가장 마음에 걸리는 것은 무엇인가요?

카테고리:

- 공부
- 시험
- 진로
- 취업
- 직장
- 사업
- 돈
- 연애
- 결혼
- 가족
- 친구
- 인간관계
- 미래
- 생활 컨디션
- 외로움
- 기타

다국어에서는 고정 `concern_id`를 사용한다.

---

# 19. TOD-004 오늘 운세 결과

## 헤더

`오늘의 모나`

### 종합

- Score 또는 Level
- 한 줄 메시지

예:

> 오늘은 답을 빨리 찾기보다 방향을 다시 확인해 보는 편이 좋아요.

## 영역

- 사랑
- 일
- 돈
- 성장
- 관계

## 안내

점수는 절대적 예측이 아니라 내부 factor의 상대적 표현임을 설정/도움말에서 설명한다.

---

# 20. TOD-005 오늘 상세

## 섹션

### 오늘의 흐름

- favorable factors
- caution factors

### 명리 근거

Friendly label 우선.

Expert mode를 펼치면:

- 십신
- 오행
- 합충
- transit

### 작은 행동

예:

> 중요한 답장은 보내기 전에 한 번 더 읽어보세요.

### CTA

`모나에게 물어보기`

---

# 21. FOR-001 운세 허브

## 상단

> 무엇을 보고 싶나요?

## 주요 카드

### 사주
나의 기본 구조

### 오늘 / 주 / 월 / 년 / 평생

기간별 흐름

### 궁합

두 사람의 관계

### 토정비결

전통 연간 운세

### 이름풀이

이름과 사주

### 별자리

가벼운 별자리 콘텐츠

### 직관형 운세

질문형 감성 콘텐츠

---

# 22. SAJ-001 사주 홈

## 상단

프로필 selector

- 나
- 등록된 관계 프로필

## Summary

- 사주팔자
- Day Master
- 오행 요약
- 주요 성향
- 대운 현재 위치

Action:

- `자세히 보기`

---

# 23. SAJ-002 사주팔자

## PillarGrid

```text
시주 | 일주 | 월주 | 년주
천간
지지
```

각 칸 터치 시:

- 한자
- 읽는 법
- 오행
- 음양
- 십신
- 도움말

## 출생시간 모름

시주 칸:

`출생시간 필요`

---

# 24. SAJ-003 오행

## 시각화

- Wood
- Fire
- Earth
- Metal
- Water

두 값 구분:

- 기본 구성
- 해석용 균형

UI에서 숫자를 과도하게 절대화하지 않는다.

## 상세

- 강한 요소
- 부족한 요소
- 균형 설명
- Color/Gem 연결 링크

---

# 25. SAJ-004 십신

## Friendly view

10개 관계를 친화적으로 설명.

예:

- 비견 → 동료·자기주도
- 정재 → 안정형 재물
- 정관 → 책임·질서

## Expert view

전통명 + 원어/한자.

다국어 표준집의 Expert/Friendly layer를 사용한다.

---

# 26. SAJ-005 관계 구조

표시:

- 합
- 충
- 형
- 파
- 해

Friendly UI:

- 연결
- 변화
- 긴장
- 흐트러짐
- 미묘한 마찰

전문가 상세 펼치기에서 전통 용어 노출.

---

# 27. SAJ-006 사주 구조

P1.

- 신강/신약
- 격국
- 용신
- 희신
- 기신

전문가 검증이 완료되지 않은 항목은 사용자에게 정식 노출하지 않는다.

State:

```text
AVAILABLE
EXPERT_REVIEW_PENDING
NOT_AVAILABLE
```

---

# 28. SAJ-007 대운 타임라인

## 형태

가로 스크롤 또는 세로 timeline.

예:

```text
7~16
17~26
27~36
37~46
...
```

각 구간:

- 간지
- 핵심 흐름
- 주요 Theme
- Opportunity
- Challenge

현재 구간을 강조한다.

---

# 29. FOR-010 주간 운세

## 구성

- 한 주 전체 흐름
- 좋은 날
- 신중할 날
- 사랑
- 일
- 돈
- 관계
- 이번 주 행동

일별 상세로 진입 가능.

---

# 30. FOR-011 월간 운세

- 월 전체 흐름
- 상순/중순/하순 또는 주요 전환일
- 돈
- 일
- 사랑
- 관계
- 성장
- 일정 참고

---

# 31. FOR-012 올해 운세

- 올해 총운
- 상반기 / 하반기
- 재물
- 직업
- 사업
- 연애
- 결혼
- 가족
- 성장
- 중요한 시기

CTA:

`올해를 모나와 이야기하기`

---

# 32. FOR-013 평생 / 인생 흐름

대운 Timeline과 연결.

과도한 확정 표현 금지.

예:

> 이 시기에는 새로운 방향을 탐색하려는 마음이 커질 수 있어요.

---

# 33. COM-001 궁합 시작

두 프로필 선택.

```text
Profile A
Profile B
Relationship Type
```

관계:

- 연인
- 썸
- 배우자
- 가족
- 친구
- 직장
- 사업

---

# 34. COM-002 상대 등록

상대 프로필이 없으면 `Relationship Profile` 생성.

기본 정보는 본인 프로필과 동일 구조.

---

# 35. COM-003 궁합 결과

## 영역

- 종합
- 정서
- 대화
- 생활
- 사랑
- 가치관
- 갈등
- 서로의 장점
- 조심할 부분

## 원칙

`좋은 궁합 / 나쁜 궁합`으로 단정하지 않는다.

CTA:

`이 관계를 모나와 이야기하기`

---

# 36. FOR-020 토정비결

## 올해 선택

기본: 현재 연도

## 결과

- 총운
- 월별
- 재물
- 일
- 관계
- 가정
- 좋은 흐름
- 조심할 흐름

해외 locale에서는 `Tojeong Bigyeol` 고유명사 + 설명 사용.

---

# 37. FOR-030 이름풀이

## 입력

- 한글 이름
- 한자 선택

## 결과

- 이름 의미
- 획수
- 음오행
- 자원오행
- 수리
- 사주와의 관계

이름풀이와 사주팔자를 동일한 개념으로 표현하지 않는다.

---

# 38. FOR-040 별자리

가벼운 콘텐츠.

- 오늘
- 이번 주
- 이번 달
- 사랑
- 일
- 돈
- 행운

다른 운세 결과와 충돌할 경우 “어느 것이 진짜인가”를 만들지 않도록 별자리 영역은 재미/가벼운 참고 성격을 명확히 한다.

---

# 39. FOR-050 직관형 운세

실제 영적 능력을 주장하지 않는다.

## 화면

> 마음속 질문을 하나 떠올려보세요.

입력:

- 질문
- 분야

결과:

- 현재 Theme
- Influence
- Opportunity
- Challenge
- Guidance

타로식 중립 어휘를 활용하되 사주 전문 계산과 혼동하지 않는다.

---

# 40. MON-001 모나 홈

## 상단

> 오늘은 어떤 이야기를 하고 싶나요?

### Quick prompts

- 오늘 마음이 복잡해
- 직장 때문에 힘들어
- 연애가 고민이야
- 돈 걱정이 있어
- 미래가 막막해
- 그냥 이야기하고 싶어

## Context 표시

작은 chip:

- 오늘 운세 사용 중
- 현재 마음 사용 중
- 관계 프로필 사용 중

사용자가 Context 사용 여부를 확인할 수 있게 한다.

---

# 41. MON-002 모나 상담

## 메시지 구조

- 사용자 메시지
- 모나 메시지
- 추천 질문
- 관련 운세 카드

## 기본 답변 구조

```text
공감
↓
현재 운의 흐름
↓
현실적 관점
↓
작은 행동
```

항상 네 부분을 기계적으로 보여주지는 않지만 내부 prompt 구조는 이 흐름을 따른다.

---

# 42. MON-003 Context 설정

사용자가 현재 상담에 무엇을 반영할지 확인.

선택:

- 내 사주
- 오늘 운세
- 최근 마음
- 최근 고민
- 관계 프로필
- 이전 상담

Default는 필요한 최소 Context.

---

# 43. MON-004 이전 기록 불러오기

예:

> 지난달 직장 고민과 연결해서 볼까요?

관련 기록만 retrieval.

모든 history를 prompt에 넣지 않는다.

---

# 44. MON-005 상담 종료 요약

선택적으로 생성.

- 오늘 이야기한 주제
- 지금 마음
- 모나의 한마디
- 오늘 해볼 작은 행동

Action:

- 히스토리 저장
- 저장하지 않기

---

# 45. FUN-001 재미 허브

## 카드

- 타로
- 내 관상
- 꿈해몽
- 행운부적
- 나의 패션
- 행운 뽑기

상단 문구:

> 오늘은 조금 가볍게 즐겨볼까요?

---

# 46. FUN-010 타로 시작

## 선택

- 오늘 한 장
- 사랑
- 돈
- 일
- 행운
- 필요한 조언
- 자유 질문

---

# 47. FUN-011 타로 카드 선택

카드 뒷면 배열.

사용자가 직접 선택한다.

v1:

- 1장
- 3장

animation은 UI layer.

카드 선택 결과는 RNG seed와 함께 history에 저장 가능.

---

# 48. FUN-012 타로 결과

표시:

- 카드 이미지
- 카드명
- Upright / Reversed
- Theme
- Influence
- Guidance
- Mona Tip

타로는 “재미로 보는 콘텐츠”임을 결과 화면 하단에 자연스럽게 표시.

---

# 49. FUN-020 내 관상 시작

## 매우 중요한 정책

**본인의 사진만 허용한다.**

화면 안내:

> 이 기능은 본인 사진으로만 이용할 수 있어요. 본인 사진을 필터, 스티커, 프레임, 메이크업, 색감 보정 등으로 꾸민 사진은 괜찮아요.

Checkbox:

`이 사진은 제 본인 사진입니다.`

동의하지 않으면 진행하지 않는다.

---

# 50. FUN-021 관상 사진 선택

선택:

- 카메라
- 갤러리

허용:

- 본인 원본
- 본인 보정
- 본인 필터
- 본인 메이크업
- 본인 스티커
- 본인 프레임

허용하지 않음:

- 친구
- 가족
- 연인
- 유명인
- 연예인
- 제3자
- 인터넷 얼굴 사진

---

# 51. FUN-022 관상 품질 확인

확인:

- 얼굴 존재
- 한 명의 얼굴
- 지나치게 가려짐 여부
- 분석 가능한 해상도
- 얼굴 방향

꾸민 사진이라고 차단하지 않는다.

오류 예:

> 꾸민 사진도 괜찮아요. 다만 얼굴이 너무 많이 가려져 있어요. 얼굴이 조금 더 잘 보이는 사진을 사용해 주세요.

---

# 52. FUN-023 관상 결과

제공:

- 오늘의 인상
- 분위기
- 매력 포인트
- 이미지 키워드
- 오늘 어울리는 컬러
- 패션 TIP
- 모나 한마디

금지:

- 지능 판단
- 도덕성 판단
- 범죄성 판단
- 건강 진단
- 인간의 가치 판단

---

# 53. FUN-030 꿈해몽

## 입력

자유 텍스트.

예:

> 어제 할머니가 집에 오셔서 돈을 주는 꿈을 꿨어.

## 선택

꿈에서 느낀 감정:

- 편안
- 기쁨
- 불안
- 무서움
- 혼란
- 기억 안 남

---

# 54. FUN-031 꿈 결과

AI 구조화 + Dream Symbol DB.

표시:

- 꿈의 주요 상징
- 가능성 있는 의미
- 긍정적인 관점
- 조심할 관점
- 모나의 한마디

단정 표현 금지.

---

# 55. FUN-040 행운부적

종류:

- 용기
- 시험
- 취업
- 집중
- 인연
- 새로운 시작
- 마음 안정
- 여행
- 행운

선택 → 모나 부적 카드 생성.

---

# 56. FUN-050 나의 패션 홈

## 입력

오늘 상황:

- 출근
- 학교
- 면접
- 시험
- 데이트
- 모임
- 여행
- 회의
- 계약
- 특별한 날
- 일상

## 내부 Context

```text
Five Elements
+
Daily Fortune Factors
+
Situation
```

---

# 57. FUN-051 오늘의 Color

출력:

- Main Color
- Point Color
- 참고 컬러
- 작은 활용 TIP

예:

> 오늘의 컬러는 네이비예요. 옷 전체보다 가방이나 작은 액세서리에 포인트로 사용해도 좋아요.

Color는 의학적·과학적 효과를 주장하지 않는다.

---

# 58. FUN-052 오늘의 Gem

출력:

- 오늘의 보석 / 원석
- 색
- 상징 키워드
- 액세서리 제안
- 추천 상황

예:

> 오늘의 모나 젬 — Blue Topaz

치료효과/건강효능 주장 금지.

---

# 59. FUN-053 나의 패션 통합 결과

예:

## 오늘의 출근 패션

- Main Color
- Point Color
- Gem
- Accessory
- Mona Tip

Action:

- 저장
- 다른 상황 보기

---

# 60. FUN-060 오늘의 행운 뽑기

가벼운 랜덤 콘텐츠.

예:

- 숫자
- 색
- 음식
- 작은 행동
- 한마디

사주 계산 결과와 분리할 수 있다.

---

# 61. HIS-001 히스토리 홈

## 보기 방식

- Timeline
- Calendar
- Category

### Category

- 마음
- 운세
- 상담
- 궁합
- 재미
- 패션

---

# 62. HIS-002 Timeline

날짜별 표시.

예:

```text
2026.08.10

마음
걱정

오늘의 모나
결정을 서두르지 않기

상담
직장 고민

Color
Navy
```

---

# 63. HIS-003 월간 마음 요약

P1/P2.

- 자주 기록한 감정
- 자주 나온 고민
- 모나와 많이 이야기한 주제
- 저장한 오늘 행동

진단이 아니라 사용자의 기록 요약.

---

# 64. HIS-004 상세 기록

기록 하나를 열어:

- 당시 날짜
- 당시 운세
- 당시 마음
- 당시 고민
- 관련 모나 상담
- 저장된 재미 콘텐츠

삭제 가능.

---

# 65. ME-001 나

## 메뉴

- 내 프로필
- 관계 프로필
- 히스토리
- 저장
- 언어
- 개인정보
- 설정
- 앱 정보

---

# 66. ME-002 관계 프로필

목록:

- 배우자
- 연인
- 자녀
- 부모
- 친구
- 직장 등

각 관계 프로필은 사용자가 직접 추가/수정/삭제.

---

# 67. SET-001 언어

지원 준비:

- 한국어
- English
- 日本語
- 简体中文
- Français

향후:

- 繁體中文

언어 변경은 계산 결과를 재계산하지 않는다.

```text
Canonical Saju Data
    ↓
Locale Resource Switch
```

만 수행한다.

---

# 68. SET-002 Expert Terminology 옵션

향후 제공 가능.

### 기본

Friendly terms

### 상세

Traditional / Expert terms

예:

```text
기본: 책임·질서
상세: 정관 / Direct Officer / 正官
```

전문 용어 표시는 locale별 Terminology Registry 사용.

---

# 69. SET-003 개인정보

관리:

- 프로필 데이터
- 상담 데이터
- 히스토리
- 관계 프로필
- 얼굴 관련 데이터
- 계정

Action:

- 항목별 삭제
- 전체 데이터 삭제
- 계정 삭제

---

# 70. 공통 상태 설계

모든 데이터 의존 화면은 최소 다음 상태를 지원한다.

```text
LOADING
READY
EMPTY
PARTIAL
ERROR
OFFLINE
STALE
```

### PARTIAL 예

출생시간 모름 → 시주 상세 없음.

### STALE 예

오늘 운세 날짜가 바뀌었는데 캐시가 전날 결과.

---

# 71. 오프라인 정책

MVP에서 가능한 범위:

### Offline 가능

- 저장된 프로필
- 계산된 사주 원국
- 저장된 히스토리
- 일부 로컬 운세 계산
- Terminology Registry

### Online 필요 가능성

- Mona AI 상담
- 신규 콘텐츠 업데이트
- 향후 계정 동기화
- 서버 기반 이미지 기능

오프라인에서는 가능한 기능을 명확히 구분한다.

---

# 72. Analytics Event 기본 규칙

사용자의 민감한 상담 내용을 analytics에 그대로 저장하지 않는다.

예:

```text
screen_view
profile_completed
today_reading_opened
mood_saved
concern_selected
mona_chat_started
compatibility_opened
tarot_drawn
face_reading_started
style_viewed
history_opened
```

상담 본문, 생년월일, 얼굴 이미지 등은 일반 analytics event payload에 넣지 않는다.

---

# 73. i18n 개발 규칙

모든 화면 문자열은 resource key 사용.

잘못된 예:

```text
Text("오늘의 운세")
```

권장:

```text
t("screen.today.reading_title")
```

전문용어:

```text
term("ten_gods.direct_officer", mode="friendly")
```

상세:

```text
term("ten_gods.direct_officer", mode="expert")
```

---

# 74. 긴 문자열 대응

특히 영어/프랑스어를 기준으로 UI width를 테스트한다.

권장:

- 버튼 텍스트 2줄까지 허용 여부 정의
- 카드 제목 ellipsis 최소화
- dynamic height 사용
- fixed-width label 지양
- 200% text scale 테스트

---

# 75. CJK 대응

- 일본어와 중국어 줄바꿈 확인
- 한자 glyph fallback 확인
- locale별 punctuation 확인
- 한국어 한자 설명과 일본어 표기가 섞이지 않도록 QA

---

# 76. 날짜/시간 locale

예:

```text
ko-KR  2026년 8월 10일
en     Aug 10, 2026
ja-JP  2026年8月10日
zh-CN  2026年8月10日
fr-FR  10 août 2026
```

문자열 조립 금지.

locale formatter 사용.

---

# 77. 콘텐츠 번역 구조

운세 본문은 다음을 분리한다.

```text
interpretation_id
factor_ids
locale
content_version
```

같은 factor라도 locale별 문장을 독립 관리한다.

AI 번역 하나로 모든 언어를 자동 생성하는 구조를 canonical 방식으로 사용하지 않는다.

---

# 78. Mona AI 다국어 구조

Prompt는 locale별 기본 Persona를 사용한다.

```text
prompts/
├─ ko-KR/
├─ en/
├─ ja-JP/
├─ zh-CN/
└─ fr-FR/
```

AI에는 다음을 함께 전달한다.

```text
locale
term_ids
expert_labels
friendly_labels
fortune_factors
user_context
```

AI가 glossary 밖의 전문용어를 임의 생성하면 QA Review 대상으로 처리한다.

---

# 79. 화면별 권한 정책

### 카메라

FUN-021에서 실제 촬영 버튼을 눌렀을 때만 요청.

### 사진

갤러리 선택을 눌렀을 때만 요청.

### 알림

온보딩 첫 화면에서 미리 요구하지 않는다.

오늘 운세를 실제 사용한 뒤 opt-in 시점에 요청.

### 위치

출생지역은 현재 GPS가 아니라 도시 검색/선택으로도 입력 가능하게 한다.

불필요한 현재 위치 권한을 요구하지 않는다.

---

# 80. MVP 화면 범위

MVP v0.x에서 실제 구현할 화면:

```text
ONB-001
ONB-002
ONB-003

PRO-001
PRO-002
PRO-003
PRO-004
PRO-005
PRO-006
PRO-007

TOD-001
TOD-002
TOD-003
TOD-004
TOD-005

FOR-001

SAJ-001
SAJ-002
SAJ-003
SAJ-004
SAJ-007

FOR-012

MON-001
MON-002
MON-003
MON-005

HIS-001
HIS-002
HIS-004

ME-001
SET-001
SET-003
```

---

# 81. v1.x 확장 화면

MVP 안정 후:

- SAJ-005 관계 구조
- SAJ-006 사주 구조
- FOR-010 주간
- FOR-011 월간
- FOR-013 평생
- COM-001~003 궁합
- FOR-020 토정비결
- FOR-030 이름풀이
- FOR-040 별자리
- FOR-050 직관형 운세
- HIS-003 월간 요약

---

# 82. 재미 기능 확장 화면

후속:

- FUN-001
- FUN-010~012 타로
- FUN-020~023 본인 관상
- FUN-030~031 꿈해몽
- FUN-040 부적
- FUN-050~053 나의 패션
- FUN-060 행운 뽑기

---

# 83. 화면 개발 우선순위

### 1단계

프로필 + 사주

### 2단계

오늘 + 마음

### 3단계

모나 상담

### 4단계

히스토리

### 5단계

궁합 + 기간운세

### 6단계

부록

---

# 84. UI와 계산엔진의 연결 원칙

UI가 계산엔진 함수를 직접 호출하지 않는다.

```text
UI
↓
Application Service
↓
Saju Service
↓
Calculation Engine
```

예:

```text
CreateProfileUseCase
GenerateSajuUseCase
GetTodayReadingUseCase
StartMonaChatUseCase
GetHistoryUseCase
```

---

# 85. 화면 ViewModel 예

`TOD-004 Today Reading`

```json
{
  "screen_id": "TOD-004",
  "date": "2026-08-10",
  "overall": {
    "score": 78,
    "level": "GOOD"
  },
  "message_key": "fortune.today.summary.001",
  "domains": [
    {"id": "domain.love", "score": 76},
    {"id": "domain.career", "score": 69},
    {"id": "domain.wealth", "score": 65},
    {"id": "domain.study", "score": 81},
    {"id": "domain.relationships", "score": 74}
  ],
  "lucky_color_id": "color.navy",
  "action_key": "fortune.today.action.021",
  "evidence_ids": [
    "ten_gods.direct_officer",
    "interaction.combination"
  ]
}
```

화면은 이 데이터와 locale resource를 조합한다.

---

# 86. Accessibility 기본 원칙

- 색만으로 좋음/주의를 구분하지 않는다.
- 아이콘 + 텍스트 병기.
- 글자 확대 대응.
- 명암 대비 확보.
- 타로/관상 카드의 텍스트 대체 설명 제공.
- 애니메이션 감소 설정 고려.
- VoiceOver / TalkBack semantic label 준비.

---

# 87. 빈 상태 예시

### 히스토리 없음

> 아직 기록이 없어요. 오늘의 마음부터 남겨볼까요?

### 관계 프로필 없음

> 궁합을 보려면 상대 프로필을 먼저 추가해 주세요.

### 상담 이력 없음

> 처음 이야기해도 괜찮아요. 지금 마음에 떠오르는 것부터 말해 주세요.

---

# 88. 오류 문구 원칙

개발 오류 코드가 사용자에게 노출되지 않는다.

잘못된 예:

```text
SAJU_CALC_504 ERROR
```

사용자:

> 사주 계산 중 문제가 생겼어요. 입력한 생년월일과 시간을 다시 확인해 주세요.

상세 로그에는 error code 저장.

---

# 89. 알림 UX

사용자가 동의한 경우만.

추천:

- 오늘 운세
- 마음 체크
- 저장한 중요한 날짜

금지:

> 오늘 사고수가 있습니다.

권장:

> 오늘의 모나가 준비됐어요.

---

# 90. 앱 시작 후 대표 사용자 Journey

```text
설치
↓
모나 소개
↓
프로필 등록
↓
사주 계산
↓
나의 사주 요약
↓
오늘 마음 체크
↓
오늘 운세
↓
모나에게 질문
↓
상담
↓
오늘 행동 저장
↓
히스토리
↓
다음날 재방문
```

---

# 91. 장기 사용자 Journey

```text
오늘 운세
↓
매일 마음 기록
↓
모나 상담
↓
운세/고민 히스토리
↓
궁합 / 연간 운세
↓
타로 / 관상 / 패션
↓
몇 달 후
↓
과거의 나와 현재를 연결하는 모나
```

---

# 92. 개발 Gate

화면 구현 완료 기준:

```text
NAVIGATION_PASS
EMPTY_STATE_PASS
ERROR_STATE_PASS
PARTIAL_STATE_PASS
I18N_5_LOCALE_PASS
TERMINOLOGY_LINT_PASS
TEXT_SCALE_PASS
ACCESSIBILITY_LABEL_PASS
PRIVACY_FLOW_PASS
ANALYTICS_PRIVACY_PASS
```

---

# 93. 다음 설계 단계

다음 단계는 **모나 데이터 모델 + DB Schema v1**이다.

설계 대상:

```text
User
Profile
BirthProfile
RelationshipProfile
SajuChart
SajuFactor
FortunePeriod
Mood
Concern
ReadingSession
ChatSession
History
Compatibility
InterpretationRule
InterpretationContent
TermRegistry
LocaleResource
Color
Gem
Tarot
Dream
FaceReading
Talisman
```

핵심 원칙:

1. 계산 데이터와 표시 문자열 분리
2. 개인 데이터와 공용 콘텐츠 분리
3. 히스토리 append 중심
4. Rule Version 저장
5. locale-independent canonical IDs 저장
6. AI 대화 Context는 필요한 기록만 retrieval
7. 관상 원본 사진은 기본 DB 영구보관하지 않음

---

# 94. v1 결정 요약

- 하단 5개 탭: 오늘 / 운세 / 모나 / 재미 / 나
- 모나 상담이 중심 경험
- 프로필 생성 즉시 사주 계산
- 출생시간 모름 정상 지원
- 사주 화면은 Friendly / Expert layer 분리
- 오늘 화면에서 마음 → 고민 → 운세 → 행동 → 상담으로 연결
- 타로/관상/꿈/부적/패션은 재미 영역
- 관상은 본인 사진만 허용, 본인 사진 꾸미기는 허용
- 나의 패션은 Color + Gem + 상황별 스타일
- 히스토리는 핵심 기능
- 모든 화면은 `term_id`와 locale resource 사용
- 5개 기본 locale을 처음부터 고려
- UI에서 계산 엔진 직접 호출 금지
- 개인정보는 analytics와 분리
- 다음 단계는 Data Model / DB Schema

---

**MONA App IA & Screen Specification v1.0**  
**Next: MONA Data Model & Database Schema v1**
