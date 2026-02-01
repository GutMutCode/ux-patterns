# UX/UI 문제 패턴 사전

> 프로젝트에서 발견된 UX/UI 문제와 해결 패턴을 체계적으로 정리한 문서

---

## 문서 구조

각 문제는 다음 형식으로 정리:

```
### [문제 ID] 문제 요약

**발견일**: YYYY-MM-DD
**발견 위치**: 파일/화면 경로
**심각도**: Critical / Major / Minor
**상태**: 해결됨 / 미해결 / 검토중

#### 1. 공식 용어 (Official Terms)
| 영어 | 한국어 | 정의 |
|------|--------|------|

#### 2. 해결 패턴 (Solution Patterns)
| 패턴명 | 설명 | 적용 사례 |
|--------|------|----------|

#### 3. 휴리스틱 분류 (Heuristic Classification)
- Nielsen Norman Group 10 Heuristics 기준

#### 4. 실무 표현 (Practical Expressions)
- 사용자/개발자가 실제로 사용하는 표현

#### 5. 해결 방법 (Solution)
- 구체적인 해결 코드/방법

#### 6. 참고 자료 (References)
- 관련 문서, 아티클 링크
```

---

## 문제 목록

| ID | 문제 | 심각도 | 상태 | 발견일 |
|----|------|--------|------|--------|
| [UX-001](#ux-001-스크롤-시-컨트롤-이탈) | 스크롤 시 컨트롤 이탈 | Major | 해결됨 | 2026-02-01 |
| [UX-002](#ux-002-고정-요소-겹침) | 고정 요소 겹침 | Major | 해결됨 | 2026-02-01 |

---

## UX-001: 스크롤 시 컨트롤 이탈

**발견일**: 2026-02-01  
**발견 위치**: `hana/lib/hanary_web/live/squad_live.ex` (스쿼드 작업 목록)  
**심각도**: Major  
**상태**: 해결됨  
**커밋**: `6cfe25f`

### 문제 설명

스쿼드 화면에서 작업 목록을 스크롤하면 "모든 작업 / 내 작업 / AI 작업" 필터 탭이 함께 스크롤되어 화면에서 사라짐. 사용자가 필터를 변경하려면 다시 맨 위로 스크롤해야 함.

### 1. 공식 용어 (Official Terms)

| 영어 | 한국어 | 정의 |
|------|--------|------|
| **Scroll-away navigation** | 스크롤 이탈 내비게이션 | 스크롤 시 뷰포트에서 사라지는 내비게이션 요소 |
| **Loss of context** | 컨텍스트 손실 | 사용자가 현재 상태나 위치 정보를 인지할 수 없게 되는 상황 |
| **Hidden controls** | 숨겨진 컨트롤 | 필요한 시점에 접근할 수 없는 UI 컨트롤 |
| **Navigation persistence** | 내비게이션 지속성 | 내비게이션이 항상 접근 가능한 상태를 유지하는 특성 |
| **Viewport affinity** | 뷰포트 친화성 | UI 요소가 뷰포트(화면)에 얼마나 잘 고정되는지의 정도 |

### 2. 해결 패턴 (Solution Patterns)

| 패턴명 | 설명 | 적용 사례 |
|--------|------|----------|
| **Sticky positioning** | CSS `position: sticky`로 스크롤 시 상단 고정 | 헤더, 탭바, 필터 |
| **Fixed positioning** | CSS `position: fixed`로 뷰포트에 완전 고정 | 모달, FAB, 사이드바 |
| **Persistent navigation** | 스크롤과 무관하게 항상 보이는 내비게이션 | GNB, 탭바 |
| **Floating action button (FAB)** | 화면 하단/측면에 떠있는 액션 버튼 | 추가 버튼, 채팅 버튼 |
| **Scroll container separation** | 스크롤 영역과 고정 영역을 구조적으로 분리 | 본 케이스에 적용 |

### 3. 휴리스틱 분류 (Heuristic Classification)

**Nielsen Norman Group 10 Usability Heuristics** 기준:

| 휴리스틱 | 위반 여부 | 설명 |
|----------|----------|------|
| **#7: Flexibility and efficiency of use** | **위반** | 자주 사용하는 필터에 빠르게 접근할 수 없음 |
| **#6: Recognition rather than recall** | **위반** | 현재 어떤 필터가 적용되어 있는지 확인 불가 |
| **#1: Visibility of system status** | **부분 위반** | 현재 보기 모드가 보이지 않아 상태 파악 어려움 |

**WCAG 2.1 접근성 기준**:
- **2.4.8 Location (AAA)**: 사용자가 현재 위치를 파악할 수 있어야 함

### 4. 실무 표현 (Practical Expressions)

**사용자 피드백**:
- "스크롤하면 탭이 안 보여요"
- "필터 바꾸려면 다시 올라가야 해요"
- "지금 뭘 보고 있는지 모르겠어요"

**개발자 표현**:
- "탭을 sticky로 고정해야 할 것 같아요"
- "스크롤 영역 밖으로 빼야 해요"
- "overflow 컨테이너 구조 문제예요"

**디자이너 표현**:
- "필터는 항상 보여야 해요"
- "컨텍스트가 유지되어야 해요"
- "고정 영역과 스크롤 영역 분리가 필요해요"

### 5. 해결 방법 (Solution)

**선택한 방법**: Scroll container separation (스크롤 컨테이너 분리)

고정할 요소를 `overflow-y-auto` 스크롤 컨테이너 밖으로 이동하여 구조적으로 분리.

| 방법 | 장점 | 단점 | 채택 |
|------|------|------|------|
| Sticky positioning | 변경 최소화 | 완료 섹션 열면 밀림 | X |
| Container separation | 구조적 해결, 일관성 | 코드 이동 필요 | **O** |
| Fixed positioning | 확실한 고정 | 레이아웃 복잡도 증가 | X |

### 6. 참고 자료 (References)

- [Nielsen Norman Group: 10 Usability Heuristics](https://www.nngroup.com/articles/ten-usability-heuristics/)
- [MDN: CSS position sticky](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

---

## UX-002: 고정 요소 겹침

**발견일**: 2026-02-01  
**발견 위치**: `hana/lib/hanary_web/live/tasks_live.ex` (작업 화면)  
**심각도**: Major  
**상태**: 해결됨  

### 문제 설명

작업 화면에서 Toggle + Hero 영역을 `sticky top-0`으로 고정했으나, 페이지 상단에 GNB(Global Navigation Bar)가 `sticky top-0 z-40`으로 이미 고정되어 있어서 스크롤 시 sticky 영역이 GNB 뒤로 들어가 윗부분이 가려짐.

### 1. 공식 용어 (Official Terms)

| 영어 | 한국어 | 정의 |
|------|--------|------|
| **Sticky/Fixed Element Overlap** | 고정 요소 겹침 | 여러 고정 요소가 동일한 위치에 겹쳐 콘텐츠가 가려지는 현상 |
| **Content Occlusion** | 콘텐츠 가림 | 다른 UI 요소에 의해 콘텐츠가 보이지 않게 되는 상태 |
| **Z-index Stacking Conflict** | Z-인덱스 쌓임 충돌 | 여러 요소의 z-index가 충돌하여 의도치 않은 레이어 순서가 발생 |
| **Viewport Collision** | 뷰포트 충돌 | 뷰포트 내 고정 요소들이 서로 공간을 침범하는 상황 |
| **Header Occlusion** | 헤더 가림 | 고정된 헤더에 의해 하위 콘텐츠가 가려지는 현상 |

### 2. 해결 패턴 (Solution Patterns)

| 패턴명 | 설명 | 적용 사례 |
|--------|------|----------|
| **Top offset adjustment** | sticky/fixed 요소의 top 값을 상위 고정 요소 높이만큼 조정 | 본 케이스에 적용 |
| **CSS custom properties** | GNB 높이를 CSS 변수로 정의하여 일관성 유지 | `--header-height: 56px` |
| **Stacking context management** | z-index 체계를 설계하여 레이어 충돌 방지 | z-10, z-20, z-30, z-40 |
| **Scroll margin/padding** | `scroll-margin-top`으로 앵커 링크 시 여백 확보 | 앵커 네비게이션 |

### 3. 휴리스틱 분류 (Heuristic Classification)

**Nielsen Norman Group 10 Usability Heuristics** 기준:

| 휴리스틱 | 위반 여부 | 설명 |
|----------|----------|------|
| **#1: Visibility of system status** | **위반** | 고정 영역의 일부가 가려져 현재 상태를 볼 수 없음 |
| **#8: Aesthetic and minimalist design** | **부분 위반** | 요소 겹침으로 인한 시각적 혼란 |

### 4. 실무 표현 (Practical Expressions)

**사용자 피드백**:
- "상단바에 가려져서 안 보여요"
- "위가 잘려요"
- "스크롤하면 뭔가 가려져요"

**개발자 표현**:
- "sticky가 GNB 뒤로 들어가요"
- "top offset이 안 맞아요"
- "z-index 문제인 것 같아요"

**디자이너 표현**:
- "고정 영역끼리 충돌해요"
- "레이어 겹침 이슈가 있어요"
- "헤더 높이만큼 밀어줘야 해요"

### 5. 해결 방법 (Solution)

**선택한 방법**: Top offset adjustment

`sticky top-0` → `sticky top-14` (56px)로 변경하여 GNB 높이만큼 아래에 고정.

| 방법 | 장점 | 단점 | 채택 |
|------|------|------|------|
| Top offset | 간단, 즉시 적용 | 하드코딩, GNB 높이 변경 시 수정 필요 | **O** |
| CSS variable | 유지보수 용이 | 초기 설정 필요 | 향후 고려 |
| z-index 조정 | 레이어 명확화 | 근본 해결 아님 (여전히 겹침) | X |

### 6. 참고 자료 (References)

- [MDN: CSS position sticky](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky)
- [CSS Tricks: Practical CSS Scroll Snapping](https://css-tricks.com/practical-css-scroll-snapping/)
- [Web.dev: Content hidden under fixed headers](https://web.dev/articles/css-scroll-snap)

---

## 카테고리별 인덱스

### 내비게이션 (Navigation)
- [UX-001](#ux-001-스크롤-시-컨트롤-이탈) - 스크롤 시 컨트롤 이탈

### 레이아웃 (Layout)
- [UX-002](#ux-002-고정-요소-겹침) - 고정 요소 겹침

### 폼 (Forms)
(아직 없음)

### 피드백 (Feedback)
(아직 없음)

### 접근성 (Accessibility)
(아직 없음)

### 반응형 (Responsive)
(아직 없음)

---

## 용어 사전 (Glossary)

| 용어 | 정의 |
|------|------|
| **Viewport** | 사용자가 현재 보고 있는 화면 영역 |
| **Sticky** | 스크롤 시 특정 위치에 도달하면 고정되는 CSS 속성 |
| **Fixed** | 뷰포트를 기준으로 항상 같은 위치에 고정되는 CSS 속성 |
| **Overflow** | 컨테이너 크기를 초과하는 콘텐츠의 처리 방식 |
| **Heuristic** | 경험에 기반한 문제 해결 방법론 |
| **FAB** | Floating Action Button, 화면에 떠있는 액션 버튼 |
| **Occlusion** | 다른 요소에 의해 콘텐츠가 가려지는 현상 |
| **Stacking Context** | CSS에서 요소들이 z축으로 쌓이는 맥락 |
| **Z-index** | 요소의 z축 순서를 지정하는 CSS 속성 |
| **GNB** | Global Navigation Bar, 전역 상단 내비게이션 바 |

---

## 변경 이력

| 날짜 | 변경 내용 |
|------|----------|
| 2026-02-01 | 문서 생성, UX-001 추가 |
| 2026-02-01 | UX-002 고정 요소 겹침 추가 |
| 2026-02-01 | 코드 예시 간소화 |
