# Simulator Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `index.html` 을 3섹션 프레젠테이션 구조로 재편 — 시스템 오버뷰, 시나리오 카드 선택, 플로우 애니메이션.

**Architecture:** 단일 `index.html` 파일 유지. 기존 시나리오 데이터·키워드 매칭·애니메이션 로직은 그대로 재활용. HTML 구조를 3개 `<section>` 으로 재편하고, CSS와 JS를 추가·수정해 카드 클릭→시뮬레이션 연동 및 프로그레스 바를 구현.

**Tech Stack:** 순수 HTML/CSS/JavaScript (빌드 도구·외부 라이브러리 없음)

---

## 파일 구조

```
index.html   — 유일한 파일. 전체 수정 대상.
```

기존 코드 중 **재활용**하는 부분:
- `scenarios` 배열 (8개 시나리오 데이터)
- `flowTemplates` 객체
- `routeByLevel` 객체
- `ruleKeywords` 객체
- `findScenario()` 함수
- `classifyLevelByRule()` 함수
- `getAgentLabel()` 함수
- `buildFlow()` 함수
- `animateSteps()` 함수 (내부 `typeText`, `showStep` 포함)

기존 코드 중 **수정**하는 부분:
- `renderSimulation()` — 프로그레스 바 업데이트 로직 추가
- `renderEmpty()` — 섹션 3 헤더 숨기기 추가
- 이벤트 리스너 — 카드 클릭 핸들러 추가, 기존 runButton/clearButton 유지

---

## Task 1: HTML 골격 재편 — 3섹션 구조

**Files:**
- Modify: `index.html` (HTML body 전체 교체)

- [ ] **Step 1: `<body>` 안의 기존 `.page` div를 3섹션 구조로 교체**

기존:
```html
<div class="page">
  <div class="topbar">...</div>
  <div class="search-box">...</div>
  <div class="simulation" id="simulationArea">...</div>
</div>
```

교체할 내용:
```html
<div class="page">

  <!-- 섹션 1: 시스템 오버뷰 -->
  <section class="section-overview">
    <div class="overview-header">
      <h1>CONX Foundation AI Agent System</h1>
      <p class="overview-subtitle">상황 유형별 승인 라우팅 체계 — 입력된 상황을 분류하고 적절한 에이전트와 승인 경로로 자동 라우팅합니다.</p>
    </div>
    <div class="overview-grid" id="overviewGrid">
      <!-- Task 2에서 채움 -->
    </div>
  </section>

  <!-- 섹션 2: 시나리오 선택 -->
  <section class="section-select">
    <h2 class="section-title">시나리오 선택</h2>
    <div id="scenarioGroups">
      <!-- Task 4에서 채움 -->
    </div>
    <div class="custom-input-box">
      <p class="custom-input-label">💬 등록되지 않은 상황 직접 입력</p>
      <div class="search-input-group">
        <input id="scenarioInput" class="search-input" type="text" placeholder="예: 공시 등록, 거래소 문의, MOU 체결" aria-label="시나리오 입력" />
        <div class="search-action">
          <button class="button primary" id="runButton" type="button">시뮬레이션 시작</button>
          <button class="button secondary" id="clearButton" type="button" disabled>초기화</button>
        </div>
      </div>
    </div>
  </section>

  <!-- 섹션 3: 플로우 애니메이션 -->
  <section class="section-flow" id="sectionFlow">
    <div class="flow-meta" id="flowMeta" hidden>
      <div class="flow-meta-top">
        <span class="flow-level-badge" id="flowLevelBadge"></span>
        <span class="flow-title-text" id="flowTitleText"></span>
        <span class="flow-agent-label" id="flowAgentLabel"></span>
      </div>
      <div class="flow-progress-wrap">
        <div class="flow-progress-bar" id="flowProgressBar"></div>
      </div>
      <div class="flow-progress-label" id="flowProgressLabel"></div>
    </div>
    <div class="simulation" id="simulationArea">
      <div class="simulation-header">
        <strong>시뮬레이션 결과</strong>
        <h2 id="simulationTitle">시나리오를 선택하거나 직접 입력하세요</h2>
      </div>
      <div class="simulation-body" id="simulationBody">
        <div class="empty-state">위에서 시나리오 카드를 클릭하거나 상황을 직접 입력하세요.</div>
      </div>
    </div>
    <div class="flow-back-wrap" id="flowBackWrap" hidden>
      <button class="button secondary" id="backToSelectButton" type="button">← 다른 시나리오 보기</button>
    </div>
  </section>

</div>
```

- [ ] **Step 2: 브라우저에서 `index.html` 열어 3섹션 뼈대 확인**

- 오버뷰 섹션 빈 상태로 표시
- 시나리오 선택 섹션에 입력창·버튼 표시
- 플로우 섹션에 빈 시뮬레이션 영역 표시
- 기존 기능(버튼 클릭) 동작 확인

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "refactor: restructure body into 3-section layout skeleton"
```

---

## Task 2: 섹션 1 — 오버뷰 HTML 채우기

**Files:**
- Modify: `index.html` (JS로 `overviewGrid` innerHTML 채우는 코드 추가)

오버뷰 그리드는 JS로 렌더링한다. 기존 `routeByLevel` 데이터를 활용.

- [ ] **Step 1: `</script>` 직전에 오버뷰 렌더링 코드 추가**

```javascript
// ── 오버뷰 렌더링 ──────────────────────────────
const overviewConfig = [
  {
    level: 'Green',
    emoji: '🟢',
    label: 'GREEN — 일반 운영',
    desc: '내부 운영, 정기 업데이트, 커뮤니티 공지 등 리스크가 낮은 상황',
    examples: ['커뮤니티 공지', '테크 문서 업데이트', 'CS 대응'],
    chain: ['실무 에이전트', 'CoS', '배포']
  },
  {
    level: 'Yellow',
    emoji: '🟡',
    label: 'YELLOW — 외부 커뮤니케이션',
    desc: '거래소 대응, 파트너 커뮤니케이션 등 외부 노출이 있는 상황',
    examples: ['공시 등록', '거래소 커뮤니케이션'],
    chain: ['실무 에이전트', 'Risk & Compliance', 'CoS', 'CEO Agent', '배포']
  },
  {
    level: 'Red',
    emoji: '🔴',
    label: 'RED — 고위험 의사결정',
    desc: '계약, MOU, 온체인 프로포절 등 법적·재무적 영향이 큰 상황',
    examples: ['파트너 MOU', '온체인 프로포절', '법무 계약 검토'],
    chain: ['실무 에이전트', 'Risk & Compliance', 'CoS', 'CEO Agent', '내부 협의체 (Human)', 'CEO Agent', '배포'],
    humanIndex: 4
  }
];

function renderOverview() {
  const grid = document.getElementById('overviewGrid');
  if (!grid) return;
  grid.innerHTML = overviewConfig.map(cfg => `
    <div class="route-col ${cfg.level.toLowerCase()}">
      <div class="route-level-badge">${cfg.emoji} ${cfg.label}</div>
      <p class="route-level-desc">${cfg.desc}</p>
      <div class="route-examples">
        ${cfg.examples.map(e => `<span>${e}</span>`).join('')}
      </div>
      <div class="route-chain">
        ${cfg.chain.map((role, i) => {
          const isHuman = cfg.humanIndex === i;
          const isDeploy = role === '배포';
          return `
            <div class="role-chip ${isHuman ? 'human' : ''} ${isDeploy ? 'deploy' : ''}">${role}</div>
            ${i < cfg.chain.length - 1 ? '<div class="chain-arrow">↓</div>' : ''}
          `;
        }).join('')}
      </div>
    </div>
  `).join('');
}

renderOverview();
```

- [ ] **Step 2: 브라우저 새로고침 — 오버뷰 그리드 3열 표시 확인**

- Green / Yellow / Red 3열이 나란히 표시됨
- 각 열에 역할 칩 체인이 세로로 나열됨

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add system overview routing map (section 1)"
```

---

## Task 3: 섹션 1 — 오버뷰 CSS

**Files:**
- Modify: `index.html` (`<style>` 블록에 추가)

- [ ] **Step 1: `<style>` 블록 닫는 `</style>` 직전에 오버뷰 CSS 추가**

```css
/* ── 섹션 공통 ── */
.section-overview,
.section-select,
.section-flow {
  margin-bottom: 48px;
}

.section-title {
  margin: 0 0 20px;
  font-size: clamp(1.2rem, 2vw, 1.5rem);
  color: #2b2a28;
}

/* ── 섹션 1: 오버뷰 ── */
.overview-header {
  margin-bottom: 24px;
}

.overview-header h1 {
  margin: 0 0 8px;
  font-size: clamp(1.8rem, 3vw, 2.4rem);
  line-height: 1.1;
}

.overview-subtitle {
  margin: 0;
  color: #5b584f;
  font-size: 0.95rem;
  line-height: 1.6;
  max-width: 760px;
}

.overview-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

@media (max-width: 740px) {
  .overview-grid { grid-template-columns: 1fr; }
}

.route-col {
  border-radius: var(--radius);
  border: 1.5px solid var(--border);
  padding: 20px;
  display: grid;
  gap: 14px;
  background: var(--surface);
}

.route-col.green { border-color: rgba(29,158,117,.35); background: #f0faf5; }
.route-col.yellow { border-color: rgba(186,117,23,.35); background: #fdf8ec; }
.route-col.red { border-color: rgba(163,45,45,.35); background: #fdf0f0; }

.route-level-badge {
  font-size: 0.82rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #2b2a28;
}

.route-level-desc {
  margin: 0;
  font-size: 0.85rem;
  color: #5b584f;
  line-height: 1.5;
}

.route-examples {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.route-examples span {
  font-size: 0.78rem;
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(43,42,40,.07);
  color: #5b584f;
}

.route-chain {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0;
}

.role-chip {
  font-size: 0.85rem;
  font-weight: 600;
  padding: 8px 14px;
  border-radius: 12px;
  background: #fff;
  border: 1px solid rgba(43,42,40,.12);
  color: #2b2a28;
  width: 100%;
}

.role-chip.human {
  background: #fff7e6;
  border-color: rgba(186,117,23,.4);
  color: #7a4f00;
}

.role-chip.deploy {
  background: #f0f0f0;
  color: #5b584f;
  font-weight: 700;
}

.chain-arrow {
  color: rgba(43,42,40,.35);
  font-size: 1rem;
  padding: 2px 0 2px 14px;
  line-height: 1;
}
```

- [ ] **Step 2: 브라우저 새로고침 — 오버뷰 3열 디자인 확인**

- 각 열이 레벨 색상으로 구분됨
- 역할 칩이 정렬되어 나열됨
- 모바일에서 1열로 스택됨 (DevTools로 확인)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: overview section layout and color-coded route columns"
```

---

## Task 4: 섹션 2 — 시나리오 카드 HTML

**Files:**
- Modify: `index.html` (JS로 `scenarioGroups` innerHTML 채우는 코드 추가)

- [ ] **Step 1: `renderOverview()` 호출 아래에 카드 렌더링 코드 추가**

```javascript
// ── 시나리오 카드 렌더링 ──────────────────────────
const groupOrder = [
  { level: 'Red',    emoji: '🔴', label: 'HIGH RISK' },
  { level: 'Yellow', emoji: '🟡', label: 'MEDIUM RISK' },
  { level: 'Green',  emoji: '🟢', label: 'LOW RISK' }
];

function renderScenarioCards() {
  const container = document.getElementById('scenarioGroups');
  if (!container) return;

  container.innerHTML = groupOrder.map(group => {
    const cards = scenarios.filter(s => s.level === group.level);
    return `
      <div class="scenario-group">
        <div class="group-heading ${group.level.toLowerCase()}">
          ${group.emoji} ${group.label}
        </div>
        <div class="scenario-card-grid">
          ${cards.map(s => `
            <div class="scenario-card ${s.level.toLowerCase()}"
                 data-scenario-id="${s.id}"
                 role="button"
                 tabindex="0"
                 aria-label="${s.title} 시뮬레이션 시작">
              <div class="card-level-bar"></div>
              <div class="card-body">
                <div class="card-title">${s.title}</div>
                <div class="card-meta">
                  ${s.agent ? `<span class="card-agent">${s.agent}</span>` : ''}
                  <span class="card-steps">${(flowTemplates[s.id] || s.flow || []).length}단계</span>
                </div>
              </div>
            </div>
          `).join('')}
        </div>
      </div>
    `;
  }).join('');
}

renderScenarioCards();
```

- [ ] **Step 2: 카드 클릭 이벤트 리스너 추가 (기존 이벤트 리스너 블록 아래에)**

```javascript
// ── 시나리오 카드 클릭 핸들러 ──────────────────────
document.getElementById('scenarioGroups').addEventListener('click', (e) => {
  const card = e.target.closest('.scenario-card');
  if (!card) return;

  const scenarioId = card.dataset.scenarioId;
  const scenario = scenarios.find(s => s.id === scenarioId);
  if (!scenario) return;

  // 선택 상태 갱신
  document.querySelectorAll('.scenario-card').forEach(c => c.classList.remove('selected'));
  card.classList.add('selected');

  // 입력창 초기화
  scenarioInput.value = '';

  // 시뮬레이션 실행
  clearAnimationTimers();
  renderSimulation(scenario, scenario.title);
});

document.getElementById('scenarioGroups').addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.target.dispatchEvent(new MouseEvent('click', { bubbles: true }));
  }
});
```

- [ ] **Step 3: 브라우저 확인 — 카드 그리드 표시 및 클릭 시 시뮬레이션 동작 확인**

- 3개 그룹(Red/Yellow/Green)에 8개 카드 표시
- 카드 클릭 시 섹션 3으로 스크롤 + 애니메이션 실행

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: scenario card grid with click-to-simulate (section 2)"
```

---

## Task 5: 섹션 2 — 카드 CSS

**Files:**
- Modify: `index.html` (`<style>` 블록에 추가)

- [ ] **Step 1: `</style>` 직전에 카드 CSS 추가**

```css
/* ── 섹션 2: 시나리오 선택 ── */
.scenario-group {
  margin-bottom: 28px;
}

.group-heading {
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  padding: 6px 0;
  margin-bottom: 12px;
  color: #5b584f;
}

.group-heading.red   { color: var(--red); }
.group-heading.yellow { color: var(--yellow); }
.group-heading.green  { color: var(--green); }

.scenario-card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 12px;
}

.scenario-card {
  display: flex;
  border-radius: 18px;
  border: 1.5px solid rgba(43,42,40,.12);
  background: var(--surface);
  box-shadow: 0 4px 14px rgba(38,36,33,.05);
  cursor: pointer;
  overflow: hidden;
  transition: transform 0.15s ease, box-shadow 0.15s ease, border-color 0.15s ease;
}

.scenario-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(38,36,33,.1);
}

.scenario-card.selected {
  box-shadow: 0 8px 24px rgba(38,36,33,.12);
}

.scenario-card.green.selected  { border-color: #1d9e75; }
.scenario-card.yellow.selected { border-color: #ba7517; }
.scenario-card.red.selected    { border-color: #a32d2d; }

.card-level-bar {
  width: 5px;
  flex-shrink: 0;
}

.scenario-card.green  .card-level-bar { background: #1d9e75; }
.scenario-card.yellow .card-level-bar { background: #ba7517; }
.scenario-card.red    .card-level-bar { background: #a32d2d; }

.card-body {
  padding: 16px 16px 16px 14px;
  display: grid;
  gap: 8px;
  flex: 1;
}

.card-title {
  font-size: 0.95rem;
  font-weight: 700;
  color: #2b2a28;
  line-height: 1.3;
}

.card-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  align-items: center;
}

.card-agent {
  font-size: 0.75rem;
  color: #6d6761;
}

.card-steps {
  font-size: 0.75rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 999px;
  background: rgba(43,42,40,.07);
  color: #5b584f;
}

/* ── 커스텀 입력 박스 ── */
.custom-input-box {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 20px;
  box-shadow: var(--shadow);
  margin-top: 8px;
}

.custom-input-label {
  margin: 0 0 12px;
  font-size: 0.9rem;
  color: #5b584f;
}

.search-input-group {
  display: grid;
  gap: 10px;
}

.search-action {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: flex-end;
}
```

- [ ] **Step 2: 브라우저 확인 — 카드 디자인 확인**

- 좌측 색상 바로 레벨 구분
- hover 시 살짝 올라오는 효과
- 클릭(selected) 시 테두리 강조

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: scenario card grid and custom input box styling"
```

---

## Task 6: 섹션 3 — 프로그레스 바 및 헤더 연동

**Files:**
- Modify: `index.html` (`renderSimulation()` 함수 수정 + `animateSteps()` 수정)

- [ ] **Step 1: `renderSimulation()` 함수를 아래 내용으로 교체**

기존 `renderSimulation` 함수 전체를 찾아서 교체:

```javascript
function renderSimulation(scenario, query) {
  const level = scenario ? scenario.level : classifyLevelByRule(query);
  const route = routeByLevel[level] || routeByLevel.Green;
  const items = scenario ? buildFlow(scenario) : route;
  const agentLabel = getAgentLabel(scenario, query);
  const title = scenario ? scenario.title : '룰 기반 시나리오 분석';

  // 섹션 3 헤더 업데이트
  const flowMeta = document.getElementById('flowMeta');
  const flowLevelBadge = document.getElementById('flowLevelBadge');
  const flowTitleText = document.getElementById('flowTitleText');
  const flowAgentLabel = document.getElementById('flowAgentLabel');
  const flowProgressBar = document.getElementById('flowProgressBar');
  const flowProgressLabel = document.getElementById('flowProgressLabel');

  if (flowMeta) {
    flowMeta.hidden = false;
    flowLevelBadge.textContent = level.toUpperCase();
    flowLevelBadge.className = `flow-level-badge ${level.toLowerCase()}`;
    flowTitleText.textContent = title;
    flowAgentLabel.textContent = agentLabel ? `담당: ${agentLabel}` : '';
    flowProgressBar.style.width = '0%';
    flowProgressBar.className = `flow-progress-bar ${level.toLowerCase()}`;
    flowProgressLabel.textContent = `0 / ${items.length} 단계`;
  }

  // 완료 버튼 숨기기 (시작 시)
  const flowBackWrap = document.getElementById('flowBackWrap');
  if (flowBackWrap) flowBackWrap.hidden = true;

  simulationTitle.textContent = title;
  if (simulationArea) {
    simulationArea.classList.remove('red', 'yellow', 'green');
    simulationArea.classList.add(level.toLowerCase());
  }

  const inferenceHint = scenario ? `<span>MD 기준 AI 추론을 적용한 실제적 흐름</span>` : '<span>입력 기반 추론 흐름</span>';

  simulationBody.innerHTML = `
    <div class="hint-line" style="gap:10px; margin-bottom:16px;">
      <span>리스크 레벨: ${level}</span>
      <span>단계 수: ${items.length}</span>
      ${agentLabel ? `<span>담당: ${agentLabel}</span>` : ''}
      ${inferenceHint}
    </div>
    <div class="node-list">
      ${items.map((item, index) => `
        <div class="node-item" data-index="${index}" data-type="${scenario ? item.role.toUpperCase() + ' : ' + item.result : item.label}">
          <div class="node-step">${scenario ? 'STEP ' + item.step : item.label}</div>
          <p class="node-role">${item.role}</p>
          <p class="typing-line"></p>
          <p class="node-desc">${scenario ? item.description : item.action}</p>
        </div>
        ${index < items.length - 1 ? '<div class="timeline-connector" data-index="connector-' + index + '"></div>' : ''}
      `).join('')}
    </div>
  `;

  const nodes = Array.from(simulationBody.querySelectorAll('.node-item'));
  const connectors = Array.from(simulationBody.querySelectorAll('.timeline-connector'));

  nodes.forEach((node, idx) => {
    node.classList.remove('active', 'completed', 'visible');
    node.style.transitionDelay = `${idx * 120}ms`;
  });
  connectors.forEach((connector, idx) => {
    connector.classList.remove('visible', 'active');
    connector.style.transitionDelay = `${idx * 120 + 80}ms`;
  });

  clearButton.disabled = false;
  animateSteps(nodes, connectors, items.length);

  if (simulationArea) {
    simulationArea.scrollIntoView({ behavior: 'smooth', block: 'start' });
  }
}
```

- [ ] **Step 2: `animateSteps()` 함수 시그니처와 내부 로직 수정**

기존 `function animateSteps(nodes, connectors)` 를 찾아서 교체:

```javascript
function animateSteps(nodes, connectors, totalSteps) {
  clearAnimationTimers();
  let current = 0;

  function updateProgress(completedCount) {
    const flowProgressBar = document.getElementById('flowProgressBar');
    const flowProgressLabel = document.getElementById('flowProgressLabel');
    if (flowProgressBar) {
      flowProgressBar.style.width = `${(completedCount / totalSteps) * 100}%`;
    }
    if (flowProgressLabel) {
      flowProgressLabel.textContent = `${completedCount} / ${totalSteps} 단계`;
    }
  }

  function typeText(element, text, callback) {
    element.textContent = '';
    element.classList.add('typing-cursor');
    let index = 0;
    const frame = () => {
      if (index <= text.length) {
        element.textContent = text.slice(0, index);
        index += 1;
        const timer = setTimeout(frame, 45);
        animationTimers.push(timer);
      } else {
        element.classList.remove('typing-cursor');
        if (callback) callback();
      }
    };
    frame();
  }

  function showStep(idx) {
    if (idx >= nodes.length) {
      // 모든 스텝 완료
      updateProgress(totalSteps);
      const flowBackWrap = document.getElementById('flowBackWrap');
      if (flowBackWrap) flowBackWrap.hidden = false;
      return;
    }
    const node = nodes[idx];
    const typeLine = node.querySelector('.typing-line');
    const currentDesc = node.querySelector('.node-desc');

    nodes.slice(0, idx).forEach((prevNode) => {
      prevNode.classList.add('completed');
      prevNode.classList.remove('active');
    });

    node.classList.add('visible', 'active');
    node.scrollIntoView({ behavior: 'smooth', block: 'nearest' });

    if (idx > 0) {
      const prevConnector = connectors[idx - 1];
      if (prevConnector) prevConnector.classList.add('visible', 'active');
    }

    updateProgress(idx);

    const text = node.dataset.type || '';
    if (typeLine && text) {
      typeText(typeLine, text, () => {
        if (currentDesc) currentDesc.style.opacity = '1';
        const timer = setTimeout(() => showStep(idx + 1), 900);
        animationTimers.push(timer);
      });
    } else {
      const timer = setTimeout(() => showStep(idx + 1), 1100);
      animationTimers.push(timer);
    }
  }

  showStep(0);
}
```

- [ ] **Step 3: `renderEmpty()` 함수 수정 — 헤더 숨기기 추가**

기존 `renderEmpty` 함수를 찾아서 교체:

```javascript
function renderEmpty(message) {
  if (simulationArea) {
    simulationArea.classList.remove('red', 'yellow', 'green');
  }
  const flowMeta = document.getElementById('flowMeta');
  if (flowMeta) flowMeta.hidden = true;
  const flowBackWrap = document.getElementById('flowBackWrap');
  if (flowBackWrap) flowBackWrap.hidden = true;
  simulationTitle.textContent = '시뮬레이션 결과';
  simulationBody.innerHTML = `<div class="empty-state">${message}</div>`;
  clearButton.disabled = true;
}
```

- [ ] **Step 4: "다른 시나리오 보기" 버튼 이벤트 추가 (기존 이벤트 리스너 블록 아래에)**

```javascript
document.getElementById('backToSelectButton').addEventListener('click', () => {
  const sectionSelect = document.querySelector('.section-select');
  if (sectionSelect) sectionSelect.scrollIntoView({ behavior: 'smooth', block: 'start' });
  document.querySelectorAll('.scenario-card').forEach(c => c.classList.remove('selected'));
});
```

- [ ] **Step 5: 브라우저 확인**

- 카드 클릭 시: 헤더바에 레벨 뱃지 + 제목 + 에이전트 표시
- 프로그레스 바가 스텝 진행에 따라 채워짐
- 마지막 스텝 완료 후 "다른 시나리오 보기" 버튼 등장
- 버튼 클릭 시 섹션 2로 스크롤

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: progress bar and flow meta header for section 3"
```

---

## Task 7: 섹션 3 — 플로우 영역 CSS

**Files:**
- Modify: `index.html` (`<style>` 블록에 추가)

- [ ] **Step 1: `</style>` 직전에 섹션 3 CSS 추가**

```css
/* ── 섹션 3: 플로우 메타 헤더 ── */
.flow-meta {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 18px 22px;
  margin-bottom: 16px;
  box-shadow: var(--shadow);
  display: grid;
  gap: 12px;
}

.flow-meta-top {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
}

.flow-level-badge {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  padding: 4px 12px;
  border-radius: 999px;
}

.flow-level-badge.green  { background: #eaf8ef; color: #1d9e75; }
.flow-level-badge.yellow { background: #fff4d9; color: #ba7517; }
.flow-level-badge.red    { background: #ffe6e6; color: #a32d2d; }

.flow-title-text {
  font-size: 1.05rem;
  font-weight: 700;
  color: #2b2a28;
  flex: 1;
}

.flow-agent-label {
  font-size: 0.85rem;
  color: #6d6761;
}

.flow-progress-wrap {
  height: 6px;
  border-radius: 999px;
  background: rgba(43,42,40,.1);
  overflow: hidden;
}

.flow-progress-bar {
  height: 100%;
  border-radius: 999px;
  width: 0%;
  transition: width 0.4s ease;
}

.flow-progress-bar.green  { background: #1d9e75; }
.flow-progress-bar.yellow { background: #ba7517; }
.flow-progress-bar.red    { background: #a32d2d; }

.flow-progress-label {
  font-size: 0.82rem;
  color: #6d6761;
}

/* ── 섹션 3: 스텝 카드 크기 확대 (프로젝터 가독성) ── */
.node-item {
  padding: 22px 24px;
}

.node-item .node-role {
  font-size: 1.1rem;
}

.node-item .node-desc {
  font-size: 0.95rem;
}

/* ── 완료 버튼 영역 ── */
.flow-back-wrap {
  margin-top: 24px;
  display: flex;
  justify-content: center;
}
```

- [ ] **Step 2: 브라우저 확인**

- 프로그레스 바 색상이 레벨에 맞게 표시됨 (green/yellow/red)
- 스텝 카드 텍스트가 이전보다 크고 여백이 넉넉함
- "다른 시나리오 보기" 버튼이 완료 후 중앙 하단에 표시됨

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: flow section progress bar, larger step cards"
```

---

## Task 8: 기존 CSS 정리 — 구 topbar/search-box 스타일 제거 및 조정

**Files:**
- Modify: `index.html` (`<style>` 블록에서 불필요 스타일 제거)

- [ ] **Step 1: 더 이상 사용되지 않는 CSS 클래스 제거**

`<style>` 블록에서 아래 선택자 블록 삭제 (HTML에서 해당 요소가 제거됐으므로):

```css
/* 삭제 대상 */
.topbar { ... }
.topbar h1 { ... }
.topbar p { ... }
.search-box { ... }
.hint-line { ... }
.hint-line span { ... }
```

단, `.hint-line` 은 `simulationBody` 안에서도 인라인으로 사용 중이므로 삭제하지 말고 유지.

- [ ] **Step 2: `.page` 패딩 확인**

`.page` 의 `padding` 이 너무 좁거나 넓으면 조정:
```css
.page {
  max-width: 980px;
  margin: 0 auto;
  padding: 32px 20px 60px;
}
```

- [ ] **Step 3: 브라우저 확인 — 전체 레이아웃 이상 없음 확인**

- 콘솔에 JS 오류 없음
- 세 섹션이 자연스럽게 이어짐
- 카드 클릭 → 애니메이션 → 완료 → 버튼 → 섹션 2 스크롤 전체 플로우 동작

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: remove unused CSS from old topbar/search-box layout"
```

---

## Task 9: clearButton 연동 및 최종 통합 확인

**Files:**
- Modify: `index.html` (clearButton 이벤트 리스너 확인 및 수정)

- [ ] **Step 1: clearButton 이벤트 리스너 확인**

기존 clearButton 이벤트 리스너가 아래 내용을 포함하는지 확인:

```javascript
clearButton.addEventListener('click', () => {
  scenarioInput.value = '';
  clearAnimationTimers();
  // 카드 선택 상태 해제
  document.querySelectorAll('.scenario-card').forEach(c => c.classList.remove('selected'));
  renderEmpty('위에서 시나리오 카드를 클릭하거나 상황을 직접 입력하세요.');
  clearButton.disabled = true;
});
```

기존 코드에 `document.querySelectorAll('.scenario-card').forEach(...)` 라인이 없으면 추가.

- [ ] **Step 2: 전체 시나리오 8개 순서대로 클릭 테스트**

각 시나리오 카드 클릭 후 확인:
- 카드 선택 강조 표시 ✓
- 섹션 3으로 스크롤 ✓
- 레벨에 맞는 색상 테마 ✓
- 프로그레스 바 단계별 진행 ✓
- 완료 후 버튼 등장 ✓

커스텀 입력 테스트:
- "공시 등록" 입력 → 시뮬레이션 동작 ✓
- 알 수 없는 키워드 입력 → 룰 기반 추론 흐름 표시 ✓
- 초기화 버튼 → 카드 선택 해제 + 빈 상태 복원 ✓

- [ ] **Step 3: 모바일 반응형 확인 (DevTools 375px)**

- 오버뷰 3열 → 1열 스택 ✓
- 카드 그리드 1열 ✓
- 버튼 전체 너비 ✓

- [ ] **Step 4: 최종 Commit**

```bash
git add index.html
git commit -m "fix: clearButton syncs card selection state, final integration"
```

---

## 자체 검토 (Spec Coverage)

| 스펙 요구사항 | 구현 Task |
|--------------|----------|
| 섹션 1: 3열 라우팅 맵 | Task 2, 3 |
| 섹션 2: 레벨별 카드 그리드 | Task 4, 5 |
| 섹션 2: 카드 클릭 → 시뮬레이션 | Task 4 |
| 섹션 2: 커스텀 텍스트 입력 유지 | Task 1 (HTML 구조에 포함) |
| 섹션 3: 프로그레스 바 | Task 6, 7 |
| 섹션 3: 플로우 메타 헤더 | Task 6, 7 |
| 섹션 3: 완료 후 "다른 시나리오" 버튼 | Task 6 |
| 스텝 카드 크기 확대 | Task 7 |
| 기존 애니메이션 로직 유지 | Task 6 (재활용) |
| 단일 HTML 파일 유지 | 전체 |
| 모바일 반응형 유지 | Task 3, 5, 8 |
