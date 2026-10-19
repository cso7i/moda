# 모나(MONA) 다국어·용어 표준집 v1.0

**문서 상태:** Canonical Terminology Draft  
**기본 언어:** 한국어(ko-KR)  
**기본 준비 언어:** 영어(en), 일본어(ja-JP), 중국어 간체(zh-CN), 프랑스어(fr-FR)  
**후속 확장:** 중국어 번체(zh-TW/zh-HK)

## 1. 목적

모나의 사주·명리·운세·궁합·타로·관상·꿈·부적·패션 기능이 여러 언어로 확장될 때 동일 개념이 화면마다 다른 단어로 번역되거나, AI가 즉석에서 동의어를 만들어 용어가 흔들리는 문제를 방지한다.

핵심 원칙은 **하나의 개념 = 하나의 고정 `term_id`** 이다. 사용자 화면의 번역문은 바뀔 수 있지만 `term_id`는 버전 호환성을 위해 바꾸지 않는다.

## 2. 다국어 기본 정책

- `ko-KR`: 기준 언어이자 기획 원문.
- `en`: 글로벌 기준 번역. BaZi 용어가 확립되어 있으면 기존 영어 관행을 따르고, 직역이 거칠면 원어/병기 + 친화 설명을 사용.
- `ja-JP`: 중국어를 기계 변환하지 않고 일본 四柱推命에서 실제 쓰는 용어를 별도 관리.
- `zh-CN`: 간체 중국어를 기본 중국어 버전으로 사용.
- `fr-FR`: 직역보다 프랑스어권에서 읽기 자연스러운 BaZi/타로 표현을 사용.
- `zh-TW`, `zh-HK`: 같은 `term_id`에 번체 번역만 추가할 수 있도록 구조를 열어둔다.

## 3. 용어를 3계층으로 분리

1. **Canonical ID**: 코드·DB·API·로그가 사용하는 불변 식별자. 예: `saju.day_master`.
2. **Expert Label**: 명리 전문가·상세 화면·도움말에서 사용하는 전통/기술 용어.
3. **Friendly Label**: 일반 사용자가 읽기 쉬운 표현. 의미를 완화할 수 있지만 기술 용어를 바꾸지는 않는다.

예:

```text
term_id       = ten_gods.seven_killings
ko.expert     = 편관(칠살)
en.expert     = Seven Killings / Indirect Officer (Qi Sha)
en.friendly   = Challenge & Drive
```

즉 영어 친화 화면에서 `Seven Killings`를 무조건 노출하지 않지만, 기술적으로 다른 개념으로 바꾸지도 않는다.

## 4. 영어 용어가 어색할 때의 규칙

영어에 딱 맞는 단어가 없다고 새 점술 용어를 임의 창작하지 않는다.

우선순위:

1. BaZi/영문 명리에서 널리 쓰이는 용어 사용.
2. 필요하면 중국어 병음 또는 BaZi 이름 병기.
3. 사용자 친화 설명에는 타로에서 익숙한 **Theme, Energy, Influence, Opportunity, Challenge, Guidance, Focus, Reflection, Balance, Flow, Timing** 같은 중립적 어휘를 보조 라벨로 사용.
4. 보조 라벨은 전문 용어를 대체하지 않는다.

이 전략은 `Eating God`, `Seven Killings`, `Useful God`, `Punishment`처럼 영어 직역이 거칠거나 오해를 낳기 쉬운 경우에 특히 적용한다.

## 5. 표기 원칙

- 영어: Title Case는 메뉴/용어명, 일반 문장에서는 sentence case.
- 일본어: 四柱推命의 일본 관용어를 우선. 중국 간체를 그대로 복사하지 않음.
- 중국어: v1은 간체. 전통 한자 원형이 필요한 필드는 `hanja`/`source_han`에 별도 저장.
- 프랑스어: 단어 이어붙이기 금지. 문장 전체를 현지화.
- 요소 `金`의 영어는 **Metal**. **Gold** 금지.
- 날짜/시간/숫자는 번역 문자열에 직접 조립하지 않고 locale formatter 사용.
- Unicode NFC 정규화.

## 6. 핵심 표준 용어

| term_id | 한국어 | English | 日本語 | 简体中文 | Français |
|---|---|---|---|---|---|
| `saju.four_pillars` | 사주 | Four Pillars of Destiny (BaZi) | 四柱推命 | 四柱命理（八字） | Quatre Piliers de la Destinée (BaZi) |
| `saju.eight_characters` | 사주팔자 | Eight Characters (BaZi) | 四柱八字 | 生辰八字 | Huit Caractères (BaZi) |
| `saju.chart` | 명식 / 원국 | BaZi Natal Chart | 命式 | 八字命盘 / 命局 | Thème natal BaZi |
| `saju.day_stem` | 일간 | Day Stem | 日干 | 日干 | Tronc du jour |
| `saju.day_master` | 일주 / 일간의 주체(일주체) | Day Master (Rì Zhǔ) | 日主 | 日主 | Maître du Jour (Rì Zhǔ) |
| `calendar.solar` | 양력 | Gregorian / Solar Calendar | 新暦 / グレゴリオ暦 | 公历 / 阳历 | Calendrier grégorien / solaire |
| `calendar.lunisolar` | 음력 | East Asian Lunisolar Calendar | 旧暦 / 太陰太陽暦 | 农历 / 阴历 | Calendrier luni-solaire |
| `calendar.leap_month` | 윤달 | Leap Month | 閏月 | 闰月 | Mois intercalaire |
| `calendar.solar_terms` | 24절기 | 24 Solar Terms | 二十四節気 | 二十四节气 | 24 termes solaires |
| `calendar.lichun` | 입춘 | Start of Spring (Lìchūn) | 立春 | 立春 | Début du printemps (Lìchūn) |
| `calendar.true_solar_time` | 진태양시 | True Solar Time | 真太陽時 | 真太阳时 | Temps solaire vrai |
| `pillar.year` | 년주 | Year Pillar | 年柱 | 年柱 | Pilier de l’année |
| `pillar.month` | 월주 | Month Pillar | 月柱 | 月柱 | Pilier du mois |
| `pillar.day` | 일주 | Day Pillar | 日柱 | 日柱 | Pilier du jour |
| `pillar.hour` | 시주 | Hour Pillar | 時柱 | 时柱 | Pilier de l’heure |
| `saju.heavenly_stems` | 천간 | Heavenly Stems | 十干 / 天干 | 天干 | Tiges célestes |
| `saju.earthly_branches` | 지지 | Earthly Branches | 十二支 / 地支 | 地支 | Branches terrestres |
| `saju.hidden_stems` | 지장간 | Hidden Stems | 蔵干 | 藏干 | Troncs cachés |
| `saju.stem_branch_pair` | 간지 | Stem-Branch Pair (Gan-Zhi) | 干支 | 干支 | Couple Tige-Branche (Gan-Zhi) |
| `saju.sexagenary_cycle` | 육십갑자 | Sexagenary Cycle / 60 Jiazi | 六十干支 | 六十甲子 | Cycle sexagésimal / 60 Jiazi |
| `principle.yin` | 음 | Yin | 陰 | 阴 | Yin |
| `principle.yang` | 양 | Yang | 陽 | 阳 | Yang |
| `principle.five_elements` | 오행 | Five Elements / Five Phases (Wu Xing) | 五行 | 五行 | Cinq Éléments / Cinq Phases (Wuxing) |
| `relation.generating` | 상생 | Generating Cycle | 相生 | 相生 | Cycle de génération |
| `relation.controlling` | 상극 | Controlling Cycle | 相剋 | 相克 | Cycle de contrôle |
| `element.wood` | 목 | Wood | 木 | 木 | Bois |
| `element.fire` | 화 | Fire | 火 | 火 | Feu |
| `element.earth` | 토 | Earth | 土 | 土 | Terre |
| `element.metal` | 금 | Metal | 金 | 金 | Métal |
| `element.water` | 수 | Water | 水 | 水 | Eau |

## 7. 십신 표준

일본어는 四柱推命에서 널리 쓰이는 `通変星` 체계를 고려하여 `正印` 대신 `印綬` 등 일본 관용 표기를 Expert Label에 반영한다. 중국어 편관은 `七杀（偏官）`를 병기한다.

| term_id | 한국어 | English expert | English friendly | 日本語 | 简体中文 | Français friendly |
|---|---|---|---|---|---|---|
| `ten_gods.system` | 십신 | Ten Gods (Shí Shén) | Ten Relationship Archetypes | 通変星（十神） | 十神 | Dix relations archétypales |
| `ten_gods.peer` | 비견 | Friend / Peer (Bi Jian) | Peer | 比肩 | 比肩 | Pair |
| `ten_gods.rob_wealth` | 겁재 | Rob Wealth (Jie Cai) | Competitor / Shared Resources | 劫財 | 劫财 | Compétition |
| `ten_gods.eating_god` | 식신 | Eating God (Shi Shen) | Creator / Expression | 食神 | 食神 | Créateur / Expression |
| `ten_gods.hurting_officer` | 상관 | Hurting Officer (Shang Guan) | Challenger / Unconventional Expression | 傷官 | 伤官 | Expression libre |
| `ten_gods.indirect_wealth` | 편재 | Indirect Wealth (Pian Cai) | Opportunity Wealth | 偏財 | 偏财 | Richesse d’opportunité |
| `ten_gods.direct_wealth` | 정재 | Direct Wealth (Zheng Cai) | Steady Wealth | 正財 | 正财 | Richesse régulière |
| `ten_gods.seven_killings` | 편관(칠살) | Seven Killings / Indirect Officer (Qi Sha) | Challenge & Drive | 偏官（七殺） | 七杀（偏官） | Défi & impulsion |
| `ten_gods.direct_officer` | 정관 | Direct Officer (Zheng Guan) | Structure & Responsibility | 正官 | 正官 | Structure & responsabilité |
| `ten_gods.indirect_resource` | 편인 | Indirect Resource (Pian Yin) | Intuitive Resource | 偏印 | 偏印 | Ressource intuitive |
| `ten_gods.direct_resource` | 정인 | Direct Resource (Zheng Yin) | Supportive Resource | 印綬（正印） | 正印 | Ressource de soutien |

### 십신 영어 친화 라벨의 원칙

- Friend/Peer → **Peer**
- Rob Wealth → **Competitor / Shared Resources**
- Eating God → **Creator / Expression**
- Hurting Officer → **Challenger / Unconventional Expression**
- Indirect Wealth → **Opportunity Wealth**
- Direct Wealth → **Steady Wealth**
- Seven Killings → **Challenge & Drive**
- Direct Officer → **Structure & Responsibility**
- Indirect Resource → **Intuitive Resource**
- Direct Resource → **Supportive Resource**

위 친화 라벨은 타로의 archetype/guidance 어휘처럼 읽기 쉽게 만든 **설명용 별칭**이며 명리 계산의 공식 명칭이 아니다.

## 8. 합·충·형·파·해 / 구조 / 운의 흐름

| term_id | 한국어 expert | English expert | English friendly | 日本語 | 简体中文 | Français friendly |
|---|---|---|---|---|---|---|
| `interaction.combination` | 합 | Combination | Connection | 合 | 合 | Connexion |
| `interaction.clash` | 충 | Clash | Tension / Change | 冲 | 冲 | Tension / changement |
| `interaction.punishment` | 형 | Punishment | Friction / Tension | 刑 | 刑 | Friction / tension |
| `interaction.break` | 파 | Break / Destruction | Disruption | 破 | 破 | Perturbation |
| `interaction.harm` | 해 | Harm | Subtle Strain | 害 | 害 | Tension subtile |
| `structure.strong_day_master` | 신강 | Strong Day Master | Stronger Day Master | 身強 | 身强 | Maître du Jour plutôt fort |
| `structure.weak_day_master` | 신약 | Weak Day Master | Weaker Day Master | 身弱 | 身弱 | Maître du Jour plutôt faible |
| `structure.pattern` | 격국 | Chart Structure / Pattern (Ge Ju) | Chart Structure | 格局 | 格局 | Structure du thème |
| `structure.yong_shen` | 용신 | Yong Shen / Useful Element | Balancing Element | 用神 | 用神 | Élément d’équilibre |
| `structure.xi_shen` | 희신 | Xi Shen / Favorable Element | Supportive Element | 喜神 | 喜神 | Élément de soutien |
| `structure.ji_shen` | 기신 | Ji Shen / Unfavorable Element | Challenging Element | 忌神 | 忌神 | Élément à modérer |
| `structure.shen_sha` | 신살 | Symbolic Stars (Shen Sha) | Symbolic Stars | 神殺 / 特殊星 | 神煞 | Étoiles symboliques |
| `structure.twelve_life_stages` | 십이운성 | Twelve Life Stages / Twelve Growth Phases | Twelve Life Stages | 十二運 / 十二運星 | 十二长生 / 十二运 | Douze phases |
| `timing.da_yun` | 대운 | 10-Year Luck Pillars (Da Yun) | 10-Year Cycle | 大運 | 大运 | Cycle de 10 ans |
| `timing.liu_nian` | 세운 | Annual Cycle (Liu Nian) | Yearly Flow | 流年 / 年運 | 流年 | Flux annuel |
| `timing.monthly` | 월운 | Monthly Cycle | Monthly Flow | 月運 | 月运 | Flux mensuel |
| `timing.daily` | 일운 | Daily Cycle | Daily Flow | 日運 | 日运 | Flux du jour |

특히 `형(Punishment)`, `편관/칠살(Seven Killings)`, `기신(Unfavorable Element)`은 상세 도움말에서는 전통명을 보존하되 일상 화면에서는 공포성 단어를 완화한다.

## 9. 제품/운세/부록 표준 용어

| term_id | 한국어 | English friendly | 日本語 friendly | 简体中文 friendly | Français friendly |
|---|---|---|---|---|---|
| `reading.today` | 오늘의 모나 | Today’s Reading | 今日のモナ | 今日解读 | Lecture du jour |
| `reading.week` | 이번 주 | This Week | 今週 | 本周 | Cette semaine |
| `reading.month` | 이번 달 | This Month | 今月 | 本月 | Ce mois-ci |
| `reading.year` | 올해 | This Year | 今年 | 今年 | Cette année |
| `reading.life_path` | 인생의 흐름 | Life Path Overview | 人生の流れ | 人生走势 | Parcours de vie |
| `product.compatibility` | 궁합 | Compatibility | 相性 | 契合度 | Compatibilité |
| `product.tojeong` | 토정비결 | Tojeong Annual Guidance | 韓国伝統の年運ガイド | 韩国传统年度指引 | Guide annuel traditionnel coréen |
| `product.name_reading` | 이름풀이 | Name Reading | 姓名判断 | 姓名分析 | Lecture du nom |
| `product.zodiac` | 별자리 | Zodiac | 星座 | 星座 | Zodiaque |
| `product.intuitive_reading` | 직관형 운세 | Intuitive Reading | 直感リーディング | 直觉解读 | Lecture intuitive |
| `domain.wealth` | 돈 | Money & Resources | お金 | 金钱与资源 | Finances & ressources |
| `domain.career` | 일 | Career | 仕事 | 事业 | Carrière |
| `domain.love` | 사랑 | Love | 恋愛 | 感情 | Amour |
| `domain.relationships` | 관계 | Relationships | 人間関係 | 人际关系 | Relations |
| `domain.study` | 공부·성장 | Study & Growth | 学び | 学习与成长 | Études & progression |
| `domain.wellbeing` | 생활 컨디션 | Well-being | コンディション | 状态 | Bien-être |
| `fun.tarot` | 타로 | Tarot | タロット | 塔罗 | Tarot |
| `fun.face_reading` | 재미로 보는 내 관상 | Fun Face Reading | 楽しむ人相占い | 趣味面相解读 | Lecture ludique du visage |
| `fun.dream_interpretation` | 꿈해몽 | Dream Reading | 夢占い | 梦境解读 | Lecture des rêves |
| `fun.talisman` | 행운부적 | Lucky Talisman | お守り | 幸运护符 | Talisman |
| `fun.my_style` | 나의 패션 | My Style | 私のスタイル | 我的风格 | Mon style |
| `fun.lucky_color` | 오늘의 컬러 | Color of the Day | 今日のカラー | 今日色彩 | Couleur du jour |
| `fun.gem` | 오늘의 보석 | Gem of the Day | 今日の宝石 | 今日宝石 | Pierre du jour |

### 토정비결

토정비결처럼 다른 언어권에 직접 대응하는 전통명칭이 없는 고유 콘텐츠는 억지 번역을 만들지 않는다.

- English: `Tojeong Bigyeol — Traditional Korean Annual Fortune Reading`
- 日本語: `トジョン・ビギョル（韓国伝統の年運占い）`
- 简体中文: `土亭秘诀（韩国传统年度运势解读）`
- Français: `Tojeong Bigyeol — lecture annuelle traditionnelle coréenne`

## 10. 타로 표준 용어

| term_id | 한국어 | English | 日本語 | 简体中文 | Français |
|---|---|---|---|---|---|
| `tarot.major_arcana` | 메이저 아르카나 | Major Arcana | 大アルカナ | 大阿卡纳 | Arcanes majeurs |
| `tarot.minor_arcana` | 마이너 아르카나 | Minor Arcana | 小アルカナ | 小阿卡纳 | Arcanes mineurs |
| `tarot.spread` | 스프레드 | Spread | スプレッド / 展開法 | 牌阵 | Tirage |
| `tarot.upright` | 정방향 | Upright | 正位置 | 正位 | À l’endroit |
| `tarot.reversed` | 역방향 | Reversed | 逆位置 | 逆位 | Renversée / inversée |

프랑스어 타로는 `arcanes majeurs / arcanes mineurs`, 카드 배열은 `tirage`를 기본으로 한다.

## 11. 타로에서 차용하는 모나 공통 UX 어휘

명리 전문 번역이 딱딱하거나 직역이 부자연스러운 경우 다음 어휘를 **설명 계층에서만** 활용한다.

| term_id | 한국어 | English | 日本語 | 简体中文 | Français |
|---|---|---|---|---|---|
| `ux.theme` | 테마 | Theme | テーマ | 主题 | Thème |
| `ux.energy` | 에너지 | Energy | エネルギー | 能量 | Énergie |
| `ux.influence` | 영향 | Influence | 影響 | 影响 | Influence |
| `ux.opportunity` | 기회 | Opportunity | 好機 | 机会 | Opportunité |
| `ux.challenge` | 도전·과제 | Challenge | 課題 | 挑战 | Défi |
| `ux.guidance` | 조언·길잡이 | Guidance | ガイダンス | 指引 | Conseil |
| `ux.focus` | 집중 포인트 | Focus | フォーカス | 重点 | Point d’attention |
| `ux.reflection` | 돌아보기·성찰 | Reflection | 振り返り | 反思 | Réflexion |
| `ux.balance` | 균형 | Balance | バランス | 平衡 | Équilibre |
| `ux.flow` | 흐름 | Flow | 流れ | 趋势 / 流动 | Flux |
| `ux.timing` | 시기 | Timing | タイミング | 时机 | Moment |
| `ux.caution` | 주의 | Caution | 注意 | 注意 | Vigilance |
| `ux.potential` | 가능성 | Potential | 可能性 | 可能性 | Potentiel |
| `ux.choice` | 선택 | Choice | 選択 | 选择 | Choix |
| `ux.transition` | 전환 | Transition | 転換 | 转变 | Transition |
| `ux.resources` | 자원 | Resources | 資源 | 资源 | Ressources |
| `ux.expression` | 표현 | Expression | 表現 | 表达 | Expression |
| `ux.structure` | 구조 | Structure | 構造 | 结构 | Structure |

예:

```text
정관이 활성화되었습니다.                ← 전문가 상세
Structure & Responsibility is emphasized. ← 영어 친화 설명

편관의 영향이 강합니다.                ← 전문가 상세
Challenge & Drive is a stronger theme.    ← 영어 친화 설명
```

## 12. 금지/주의 번역

| 개념 | 사용하지 않을 기본 UI 표현 | 권장 |
|---|---|---|
| 金(오행) | Gold | Metal |
| 用神 | Useful God 단독 | Yong Shen / Balancing Element |
| 神煞 | Evil Stars | Symbolic Stars (Shen Sha) |
| 七殺 | Seven Killings 단독 친화 UI | Challenge & Drive + expert 도움말 |
| 刑 | Punishment 단독 친화 UI | Friction / Tension + expert 도움말 |
| 관상 | Physiognomy 기본 메뉴명 | Fun Face Reading / Traditional Face Reading |
| 음력 | 기술문서에서 단순 Lunar Calendar만 사용 | Lunisolar Calendar, UI에서는 Lunar 허용 |
| 오늘 운세 | English에서 무조건 Daily Fortune | Today’s Reading / Daily Flow 우선 |

## 13. 코드/DB 국제화 구조

문자열을 코드에 직접 넣지 않는다.

```text
term_id: saju.day_master

locales:
  ko-KR:
    expert: 일주 / 일간의 주체
    friendly: 나를 나타내는 일간
  en:
    expert: Day Master (Rì Zhǔ)
    friendly: Day Master
  ja-JP:
    expert: 日主
    friendly: 日主
  zh-CN:
    expert: 日主
    friendly: 日主
  fr-FR:
    expert: Maître du Jour (Rì Zhǔ)
    friendly: Maître du Jour
```

권장 resource key:

```text
terms.saju.day_master.expert
terms.saju.day_master.friendly
terms.saju.day_master.help

screen.today.title
screen.today.mood_prompt
screen.saju.detail.title
action.save
action.view_details
```

## 14. 문장 템플릿 규칙

단어를 이어붙여 문장을 만들지 않는다.

잘못된 방식:

```text
localized_term + ' is strong today'
```

권장:

```text
fortune.ten_god.emphasis_sentence
```

각 언어가 문장 전체를 독립 번역한다. 영어/프랑스어의 단복수와 관사, 일본어/중국어의 어순을 locale별 문장 템플릿으로 해결한다.

## 15. AI 사용 규칙

Mona AI prompt에 raw 번역 문자열만 전달하지 않고 `term_id`를 함께 전달한다.

```json
{
  "term_id": "ten_gods.direct_officer",
  "expert_label": "Direct Officer (Zheng Guan)",
  "friendly_label": "Structure & Responsibility"
}
```

AI 규칙:

- Glossary에 없는 전문 동의어를 임의 생성하지 않는다.
- 전문 용어를 설명할 때 expert label을 바꾸지 않는다.
- 사용자 친화 문장은 friendly label과 UX lexicon을 활용한다.
- `Seven Killings`, `Punishment`, `Evil`, `Disaster` 같은 위협성 표현을 문맥 없이 확대하지 않는다.
- 지역별 번역은 literal translation이 아니라 locale의 승인된 문구만 사용한다.

## 16. Terminology Lint

빌드/콘텐츠 QA에 용어 검사기를 둔다.

검사 예:

- `金`을 English `Gold`로 번역하면 FAIL
- `saju.day_master`를 한 화면에서는 Day Master, 다른 화면에서는 Day Lord로 사용하면 FAIL
- 일본어 정인을 승인 없이 正印/印星/印綬로 혼용하면 FAIL
- `토정비결`을 영어에서 화면별로 Tojeong, Korean Fortune Book, Annual Secret 등으로 임의 변경하면 FAIL
- AI 출력에 등록되지 않은 명리 전문 용어가 등장하면 Review Queue로 전송

## 17. 번역 승인 상태

각 term은 다음 상태를 가진다.

```text
DRAFT
LINGUIST_REVIEWED
DOMAIN_REVIEWED
APPROVED
DEPRECATED
```

명리 핵심 용어는 **언어 검수 + 명리 도메인 검수**가 모두 끝나야 `APPROVED`로 올린다.

## 18. 다국어 QA 체크리스트

- 동일 term_id가 모든 지원 locale에 존재하는가?
- Expert/Friendly label이 뒤바뀌지 않았는가?
- 중국 간체와 일본 한자 표기가 기계적으로 섞이지 않았는가?
- French 문장의 성/수·관사·축약이 자연스러운가?
- 영어 기술어와 친화어가 다른 개념처럼 보이지 않는가?
- CJK에서 글자 잘림/줄바꿈 문제가 없는가?
- 긴 프랑스어/영어 라벨에서도 버튼 폭이 유지되는가?
- 날짜·시간이 locale 포맷으로 출력되는가?
- AI가 glossary 밖 전문용어를 만들지 않는가?
- 동일 결과가 언어만 바뀌고 계산값은 동일하게 유지되는가?

## 19. 사주 엔진과 i18n의 경계

사주 계산엔진은 번역 문자열을 반환하지 않는다.

```text
Saju Engine
  ↓
Canonical IDs / Enums
  ↓
Terminology Registry
  ↓
Locale Resource
  ↓
UI / Mona AI
```

예:

```json
{
  "ten_god_id": "ten_gods.direct_wealth",
  "element_id": "element.earth",
  "interaction_ids": ["interaction.combination"]
}
```

계산 결과에 `정재`, `Direct Wealth`, `正財` 같은 display 문자열을 직접 저장하지 않는다.

## 20. 향후 파일 구조 권장

```text
localization/
├─ terminology/
│  ├─ terms.json
│  ├─ aliases.json
│  └─ deprecated_terms.json
├─ locales/
│  ├─ ko-KR.json
│  ├─ en.json
│  ├─ ja-JP.json
│  ├─ zh-CN.json
│  └─ fr-FR.json
├─ prompts/
│  ├─ ko-KR/
│  ├─ en/
│  ├─ ja-JP/
│  ├─ zh-CN/
│  └─ fr-FR/
└─ tests/
   ├─ terminology_lint
   ├─ locale_completeness
   └─ snapshot_tests
```

## 21. v1 확정 사항

1. 한국어를 canonical 기획 언어로 한다.
2. 기본 다국어 준비는 `ko-KR / en / ja-JP / zh-CN / fr-FR`.
3. 중국어는 v1에서 간체를 기본으로 하고 번체 확장이 가능한 구조로 만든다.
4. 모든 전문 용어는 immutable `term_id`로 관리한다.
5. Expert Label과 Friendly Label을 반드시 분리한다.
6. 영어 직역이 어색한 경우 BaZi 원어/표준 번역을 보존하고 Tarot-style UX 어휘를 설명용으로 활용한다.
7. 계산엔진은 locale 독립적 ID만 반환한다.
8. AI는 glossary에 없는 전문 용어를 임의 생성하지 않는다.
9. 일본어/중국어/프랑스어는 영어 번역의 2차 번역으로 만들지 않고 각각 승인된 locale 리소스로 관리한다.
10. 용어 표준집과 machine-readable registry를 함께 버전 관리한다.

## 22. 참고 근거

- 중국의 공식 문화 설명 자료에서 `天干 / 地支`를 전통 역법의 기본 단위로 설명하고 있음.
- 일본 국립국회도서관의 달력 자료는 `十干 / 十二支`, `陰陽五行`과 60개 간지 체계를 확인하는 기준으로 활용.
- 영어권 BaZi 관행에서 `Four Pillars of Destiny`, `Heavenly Stems`, `Earthly Branches`, `Day Master`, `Ten Gods`가 널리 사용됨.
- 프랑스어권 BaZi 자료에서 `Quatre Piliers`, `Tiges célestes`, `Branches terrestres`, `Maître du Jour`, `Dix Dieux`가 사용됨.
- 프랑스 국립도서관(BnF) 및 일본 국립국회도서관의 타로 자료에서 `arcanes majeurs / mineurs`, `大アルカナ / 小アルカナ` 등 타로 기본 용어를 확인.

**Machine-readable companion:** `mona_terms_v1.json`

---

**MONA Multilingual & Terminology Standard v1.0**
