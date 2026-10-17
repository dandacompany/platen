# 문장 리듬 시각화 구현 계획

> **For implementer:** Use TDD throughout. Write failing test first. Watch it fail. Then implement.

**Goal:** 실제 타이핑의 속도와 1초 이상 멈춤을 종이 오른쪽 여백의 실시간 잉크 파형으로 시각화한다.

**Architecture:** `index.html` 안에 본문과 같은 transform을 공유하는 장식 레이어를 추가한다. 실제 키보드 이벤트에서만 타임스탬프를 기록하고, 시간 간격을 DOM 막대의 길이·농도·구분선으로 변환한다. 입력 내용과 입력 이력은 저장하지 않는다.

**Tech Stack:** HTML, CSS, vanilla JavaScript, 브라우저 내 수동 회귀 확인 (외부 라이브러리·빌드 도구 없음)

---

### Task 1: 리듬 여백 레이어와 잉크 막대 생성기 추가

**Files:**
- Modify: `index.html`
- Test: 브라우저에서 `index.html`을 열어 DOM 및 시각 상태를 확인

**Step 1: 실패하는 수동 검증 기준 작성**

브라우저에서 `index.html`을 열고 아무 키나 입력한다.

Expected: 오른쪽 종이 여백에 리듬 막대가 나타나지 않아야 한다. (기능 구현 전 기준)

**Step 2: 최소 마크업과 CSS 구현**

- `#typeImpactLayer` 뒤에 `#rhythmMarginLayer`를 추가한다.
- 레이어는 `aria-hidden="true"`, `pointer-events: none`으로 설정한다.
- `.rhythm-mark`는 오른쪽 여백에 표시되는 짧은 가로 잉크 막대다.
- `.rhythm-pause`는 긴 멈춤을 보여 주는 얇고 옅은 구분선이다.
- 모션 감소 환경에서는 진입 애니메이션을 비활성화한다.

**Step 3: 막대 생성 함수 구현**

```js
function createRhythmMark({ cursorTop, interval, isPause }) {
  const mark = document.createElement("span");
  mark.className = `rhythm-mark${isPause ? " rhythm-pause" : ""}`;
  mark.style.top = `${cursorTop}px`;
  if (!isPause) {
    const normalized = Math.max(0, Math.min(1, 1 - interval / 900));
    mark.style.setProperty("--rhythm-length", `${12 + normalized * 38}px`);
    mark.style.setProperty("--rhythm-opacity", `${0.25 + normalized * 0.6}`);
  }
  rhythmMarginLayer.appendChild(mark);
}
```

**Step 4: 수동 검증 — 표시가 생성되는지 확인**

- 개발자 도구 콘솔에서 `createRhythmMark({ cursorTop: 120, interval: 150, isPause: false })`를 실행한다.
- Expected: 오른쪽 여백의 해당 높이에 잉크 막대가 하나 나타난다.

**Step 5: Commit**

```bash
git add index.html
git commit -m "문장 리듬 레이어: 여백에 잉크 막대 표시"
```

### Task 2: 실제 키보드 리듬 기록과 종이 위치 동기화

**Files:**
- Modify: `index.html`
- Test: 브라우저에서 실제 입력·자동 데모를 비교

**Step 1: 실패하는 수동 검증 기준 작성**

- 실제 키를 두 번 입력하고 1초 이상 기다렸다가 한 번 더 입력한다.
- Expected: 기능 구현 전에는 속도 차이·멈춤 구분선이 보이지 않는다.

**Step 2: 입력 간격 기록 함수 구현**

```js
let lastRhythmInputAt = null;
const RHYTHM_PAUSE_MS = 1000;

function recordRhythmInput() {
  const now = performance.now();
  const interval = lastRhythmInputAt === null ? null : now - lastRhythmInputAt;
  lastRhythmInputAt = now;
  if (interval === null) return;

  const layerRect = rhythmMarginLayer.getBoundingClientRect();
  const cursorRect = cursorAnchor.getBoundingClientRect();
  const cursorTop = cursorRect.top - layerRect.top + cursorRect.height / 2;
  createRhythmMark({
    cursorTop,
    interval,
    isPause: interval >= RHYTHM_PAUSE_MS,
  });
}
```

**Step 3: 실제 키보드 이벤트에만 연결**

- `window`의 `keydown` 처리에서 기존 입력 대상·수정키 필터를 통과한 뒤 `recordRhythmInput()`을 호출한다.
- `stepDemo()` 및 램프/글꼴/크기 컨트롤에는 호출을 추가하지 않는다.
- `updateScreen()`에서 `rhythmMarginLayer.style.transform = paperSheet.style.transform`을 적용한다.

**Step 4: 수동 검증 — 실제 입력과 데모 구분 확인**

- 빠르게 5글자를 입력한다. Expected: 진하고 긴 막대가 누적된다.
- 1초 이상 멈춘 뒤 1글자를 입력한다. Expected: 옅은 구분선이 생긴다.
- 자동 타이핑을 재생한다. Expected: 새 막대가 생기지 않는다.
- 슬라이더·글꼴·글자 크기 컨트롤을 조작한다. Expected: 새 막대가 생기지 않는다.

**Step 5: Commit**

```bash
git add index.html
git commit -m "문장 리듬 기록: 실제 타이핑 속도와 멈춤 반영"
```

### Task 3: 리셋·접근성·회귀 검증

**Files:**
- Modify: `index.html`
- Test: 브라우저에서 전체 입력 흐름 확인

**Step 1: 실패하는 수동 검증 기준 작성**

리듬 막대를 만든 후 왼쪽 상단 리셋 버튼을 누른다.

Expected: 구현 전에는 본문만 초기화되거나 리듬 막대가 남는다.

**Step 2: 초기화 함수와 리셋 연결 구현**

```js
function clearRhythmMarks() {
  rhythmMarginLayer.replaceChildren();
  lastRhythmInputAt = null;
}
```

- `startDemo(true)`의 초기화 흐름에서 `clearRhythmMarks()`를 호출한다.
- 모션 감소 스타일에서는 `.rhythm-mark` 애니메이션을 없앤다.

**Step 3: 전체 회귀 검증**

- 실제 한글 조합, Backspace, Enter, 숫자·문장부호 입력
- 리듬 막대와 멈춤 구분선
- 자동 타이핑 시작/중지/재시작
- 리셋 후 본문·파형 동시 초기화
- 램프, 글꼴, 글자 크기, 잉크 효과
- `prefers-reduced-motion` 상태에서 파형이 애니메이션 없이 표시되는지 확인

Expected: 기존 기능이 모두 유지되고, 리듬 파형은 실제 입력에만 표시된다.

**Step 4: Commit**

```bash
git add index.html
git commit -m "문장 리듬 시각화: 리셋과 접근성 검증"
```
