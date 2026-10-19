# MONA MVP Application Architecture v0.1

```text
Presentation
    ↓
Application UseCase
    ↓
Domain Service
    ↓
Repository Interface
    ↓
SQLite / Asset / Server Adapter
```

## Saju

```text
BirthInput
↓
SajuCoreService
↓
CalendarReferenceRepository
↓
precomputed SQLite Calendar Reference
↓
SajuChart
```

## Interpretation / Mona

```text
Saju & Fortune Factors
↓
InterpretationRuleService
↓
InterpretationResult
↓
MonaContextBuilder
↓
Server-side AI Gateway (next)
↓
MonaResponseValidator
```

## 고정 원칙

- Python 사주 엔진은 Golden Reference.
- Dart 엔진은 mobile runtime.
- Calendar DB는 읽기 전용 운용.
- 사용자 DB는 별도 SQLite.
- 일반 UI 문구는 ARB.
- 전문용어는 Terminology Registry.
- AI credential은 mobile client에 넣지 않음.
- Expert Pending 계산은 user-facing final fact로 노출하지 않음.
- 개발 가정 Rule로 Release build 금지.
