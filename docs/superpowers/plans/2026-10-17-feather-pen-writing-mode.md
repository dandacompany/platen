# 깃털펜 작성 모드 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Platen에 기존 타자기와 깃털펜을 선택하는 작성 도구 모드를 추가한다.

**Architecture:** 현재 `index.html` 단일 파일 구조를 유지한다. 작성 도구 상태를 화면 효과와 Web Audio 효과의 공통 분기점으로 두고, 기존 `handleKey()` 입력 경로를 재사용한다. 커서 위치 계산은 기존 `updateScreen()`에 연결해 깃털펜도 현재 입력 위치를 따른다.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Web Audio API.

**Spec:** `docs/superpowers/specs/2026-10-17-feather-pen-writing-mode-design.md`

## Global Constraints

- 수정 파일은 `index.html` 한 개를 유지한다.
- 외부 라이브러리·빌드 도구·음원 파일을 추가하지 않는다.
- 기존 타자기 모드, 한글 조합, 자동 타이핑, 리셋, 램프 컨트롤을 유지한다.
- 기본 작성 도구는 `타자기`다.
- `main`에 직접 작업하지 않고 `feat/feather-pen-writing-mode`에서 작업한다.

## Review Focus

- 작성 도구를 바꾼 뒤에도 한글 조합 중인 글자와 커서 위치가 정확히 따라오는가.
- 공백·Enter·Backspace가 깃털펜 모드에서 일반 입력과 구별되는 소리를 내는가.
- 모드 전환과 자동 타이핑 중 기존 텍스트·카운터·조명이 유지되는가.
- 타자기 모드로 돌아왔을 때 기존 T자 가이드·잉크 효과·타자 소리가 그대로인가.
- `prefers-reduced-motion` 환경에서 애니메이션 없이도 입력이 가능한가.

---

### Task 1: 작성 도구 선택 UI와 모드 상태

**Files:**
- Modify: `index.html`의 하단 컨트롤 마크업과 화면 상태 영역

**Interfaces:**
- Produces: `writingMode` 값(`"typewriter"` 또는 `"feather"`), `writingModeSelect` 요소, `isFeatherMode()` 판정 함수.

- [ ] **Step 1: RED 확인**

브라우저 DOM 스모크 검사에서 `writingModeSelect`가 없고 작성 모드 상태가 없음을 확인한다.

- [ ] **Step 2: GREEN 구현**

하단 컨트롤에 `작성 도구` select를 추가하고 기본값을 `typewriter`로 둔다. 모드 변경 시 `desk` 또는 `lampAssembly`에 `feather-mode` 클래스를 토글하고 화면을 갱신한다.

- [ ] **Step 3: 검증**

`node --check`로 추출한 JavaScript 구문을 검사하고, 정적 HTML 검사에서 두 옵션과 기본값을 확인한다.

- [ ] **Step 4: Commit**

`git add index.html && git commit -m "작성 도구 선택 UI 추가"`

### Task 2: 깃털펜 시각 효과

**Files:**
- Modify: `index.html`의 CSS와 `updateScreen()`/입력 효과 로직

**Interfaces:**
- Consumes: Task 1의 `writingMode`, `isFeatherMode()`.
- Produces: 현재 입력 위치를 따라가는 `featherGuide` 요소와 `triggerFeatherStroke()` 함수.

- [ ] **Step 1: RED 확인**

정적 검사에서 `featherGuide`와 `triggerFeatherStroke`가 없음을 확인한다.

- [ ] **Step 2: GREEN 구현**

기존 `type-guide` 옆에 깃털펜 가이드를 추가하고 CSS로 깃털 모양과 미세한 흔들림을 표현한다. `updateScreen()`에서 현재 조합 글자 또는 커서 좌표를 사용해 깃털펜을 이동시키고, 입력 시 펜촉 스트로크 클래스를 재생한다. 타자기 모드에서는 기존 T자 가이드만 보인다.

- [ ] **Step 3: 검증**

기존 타자기 가이드 클래스와 새 깃털펜 클래스가 모드별로 상호 배타적인지 정적 검사하고, 축소 모션 CSS에서 깃털펜 애니메이션이 비활성화되는지 확인한다.

- [ ] **Step 4: Commit**

`git add index.html && git commit -m "깃털펜 커서와 필기 애니메이션 추가"`

### Task 3: 깃털펜 사운드와 입력 분기

**Files:**
- Modify: `index.html`의 `TypewriterAudio`, `handleKey()`, 모드 이벤트

**Interfaces:**
- Consumes: Task 1의 `isFeatherMode()`, Task 2의 `triggerFeatherStroke()`.
- Produces: `playFeatherStroke(kind)` (`"key"`, `"space"`, `"return"`, `"backspace"`) 사운드 함수.

- [ ] **Step 1: RED 확인**

정적 검사에서 `playFeatherStroke`가 없고 `handleKey()`가 작성 도구에 따라 분기하지 않음을 확인한다.

- [ ] **Step 2: GREEN 구현**

Web Audio API로 짧은 필기 마찰음을 합성한다. 일반 키·공백·줄바꿈·삭제를 다른 길이와 음색으로 구분하고, `handleKey()`에서 깃털펜 모드일 때만 해당 소리를 사용한다. 타자기 모드에서는 기존 소리를 그대로 호출한다.

- [ ] **Step 3: 검증**

정적 검사로 네 가지 입력 종류가 모두 `playFeatherStroke`에 연결되었는지 확인하고, 자동 타이핑도 `handleKey()` 경로를 사용하는지 확인한다.

- [ ] **Step 4: Commit**

`git add index.html && git commit -m "깃털펜 필기 소리와 입력 모드 연결"`

### Task 4: 전체 회귀 검증

**Files:**
- Verify: `index.html`, Git history and working tree

- [ ] **Step 1:** 저장소 PR 검사와 HTML 구문 검사를 실행한다.
- [ ] **Step 2:** 로컬 HTTP 서버로 `index.html`을 제공하고 정상 응답을 확인한다.
- [ ] **Step 3:** 브라우저에서 다음 수동 시나리오를 확인한다: 기본 타자기 입력, 깃털펜 전환, 한글 조합, 공백·Enter·Backspace, 자동 타이핑, 리셋, 램프 슬라이더, 램프 끄기.
- [ ] **Step 4:** 사용자 브라우저 확인 전에는 최종 커밋하지 않고 변경 내용을 보고한다.
