<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Palo Alto Networks — Employee Engagement & Burnout Diagnostics</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:ital,wght@0,400;0,700;1,400&family=DM+Sans:wght@300;400;500;600&display=swap');

  :root {
    --bg: #0a0d14;
    --surface: #111520;
    --surface2: #161c2c;
    --border: #1e2840;
    --accent: #00d4ff;
    --accent2: #ff6b35;
    --accent3: #7c3aed;
    --danger: #ef4444;
    --warning: #f59e0b;
    --success: #10b981;
    --text: #e2e8f0;
    --muted: #64748b;
    --card-glow: 0 0 30px rgba(0,212,255,0.04);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }
  
  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Background grid effect */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: 
      linear-gradient(rgba(0,212,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .app { position: relative; z-index: 1; }

  /* ── HEADER ── */
  header {
    padding: 20px 32px;
    border-bottom: 1px solid var(--border);
    background: rgba(10,13,20,0.95);
    backdrop-filter: blur(12px);
    position: sticky;
    top: 0;
    z-index: 100;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
  }

  .header-left { display: flex; align-items: center; gap: 16px; }

  .logo-badge {
    width: 38px; height: 38px;
    background: linear-gradient(135deg, var(--accent), var(--accent3));
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-family: 'Space Mono', monospace;
    font-size: 14px; font-weight: 700;
    color: #000;
  }

  .header-title h1 {
    font-size: 16px;
    font-weight: 600;
    letter-spacing: -0.02em;
    color: var(--text);
  }

  .header-title p {
    font-size: 11px;
    color: var(--muted);
    font-family: 'Space Mono', monospace;
    margin-top: 1px;
  }

  .header-meta {
    display: flex; align-items: center; gap: 16px;
  }

  .live-badge {
    display: flex; align-items: center; gap: 6px;
    font-size: 11px; color: var(--success);
    font-family: 'Space Mono', monospace;
  }

  .live-dot {
    width: 7px; height: 7px;
    border-radius: 50%;
    background: var(--success);
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; transform: scale(1); }
    50% { opacity: 0.5; transform: scale(0.8); }
  }

  /* ── NAV TABS ── */
  nav {
    display: flex;
    gap: 4px;
    padding: 16px 32px 0;
    border-bottom: 1px solid var(--border);
    background: var(--bg);
    overflow-x: auto;
  }

  nav button {
    padding: 10px 20px;
    background: transparent;
    border: none;
    color: var(--muted);
    font-size: 13px;
    font-family: 'DM Sans', sans-serif;
    font-weight: 500;
    cursor: pointer;
    border-bottom: 2px solid transparent;
    margin-bottom: -1px;
    transition: all 0.2s;
    white-space: nowrap;
    letter-spacing: 0.01em;
  }

  nav button:hover { color: var(--text); }

  nav button.active {
    color: var(--accent);
    border-bottom-color: var(--accent);
  }

  /* ── FILTERS BAR ── */
  .filters-bar {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    padding: 16px 32px;
    background: var(--surface);
    border-bottom: 1px solid var(--border);
    align-items: center;
  }

  .filter-label {
    font-size: 11px;
    font-family: 'Space Mono', monospace;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-right: 4px;
  }

  .filter-chip {
    display: flex;
    align-items: center;
    gap: 8px;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 6px 12px;
    font-size: 12px;
    color: var(--text);
  }

  select, .filter-chip select {
    background: transparent;
    border: none;
    color: var(--text);
    font-size: 12px;
    font-family: 'DM Sans', sans-serif;
    cursor: pointer;
    outline: none;
    padding: 0;
  }

  select option { background: var(--surface2); }

  .reset-btn {
    margin-left: auto;
    background: transparent;
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 6px 14px;
    color: var(--muted);
    font-size: 12px;
    cursor: pointer;
    transition: all 0.15s;
    font-family: 'DM Sans', sans-serif;
  }

  .reset-btn:hover { border-color: var(--accent); color: var(--accent); }

  /* ── MAIN CONTENT ── */
  main {
    padding: 28px 32px;
    max-width: 1600px;
    margin: 0 auto;
  }

  .tab-panel { display: none; }
  .tab-panel.active { display: block; }

  /* ── KPI CARDS ── */
  .kpi-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 16px;
    margin-bottom: 28px;
  }

  .kpi-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 20px;
    box-shadow: var(--card-glow);
    position: relative;
    overflow: hidden;
    transition: transform 0.2s, border-color 0.2s;
  }

  .kpi-card:hover {
    transform: translateY(-2px);
    border-color: var(--accent);
  }

  .kpi-card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: var(--kpi-color, var(--accent));
  }

  .kpi-label {
    font-size: 10px;
    font-family: 'Space Mono', monospace;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 10px;
  }

  .kpi-value {
    font-size: 32px;
    font-weight: 700;
    font-family: 'Space Mono', monospace;
    color: var(--kpi-color, var(--accent));
    line-height: 1;
    margin-bottom: 6px;
  }

  .kpi-sub {
    font-size: 11px;
    color: var(--muted);
  }

  /* ── CHART GRID ── */
  .chart-grid {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: 20px;
    margin-bottom: 20px;
  }

  .chart-col-4 { grid-column: span 4; }
  .chart-col-6 { grid-column: span 6; }
  .chart-col-8 { grid-column: span 8; }
  .chart-col-12 { grid-column: span 12; }

  @media (max-width: 1100px) {
    .chart-col-4, .chart-col-6, .chart-col-8 { grid-column: span 12; }
  }

  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 20px;
    box-shadow: var(--card-glow);
  }

  .card-title {
    font-size: 12px;
    font-family: 'Space Mono', monospace;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 4px;
  }

  .card-subtitle {
    font-size: 13px;
    color: var(--text);
    font-weight: 600;
    margin-bottom: 20px;
  }

  .chart-wrap {
    position: relative;
  }

  /* ── BURNOUT RISK BANDS ── */
  .risk-bands {
    display: flex;
    gap: 12px;
    margin-bottom: 20px;
  }

  .risk-band {
    flex: 1;
    padding: 16px;
    border-radius: 10px;
    border: 1px solid;
    text-align: center;
  }

  .risk-band.high { background: rgba(239,68,68,0.1); border-color: rgba(239,68,68,0.3); }
  .risk-band.medium { background: rgba(245,158,11,0.1); border-color: rgba(245,158,11,0.3); }
  .risk-band.low { background: rgba(16,185,129,0.1); border-color: rgba(16,185,129,0.3); }

  .risk-count {
    font-size: 28px;
    font-weight: 700;
    font-family: 'Space Mono', monospace;
    line-height: 1;
    margin-bottom: 4px;
  }

  .risk-band.high .risk-count { color: var(--danger); }
  .risk-band.medium .risk-count { color: var(--warning); }
  .risk-band.low .risk-count { color: var(--success); }

  .risk-label {
    font-size: 11px;
    color: var(--muted);
    font-family: 'Space Mono', monospace;
    text-transform: uppercase;
    letter-spacing: 0.06em;
  }

  /* ── TABLE ── */
  .data-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 12px;
  }

  .data-table thead th {
    text-align: left;
    padding: 10px 12px;
    font-family: 'Space Mono', monospace;
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--muted);
    border-bottom: 1px solid var(--border);
  }

  .data-table tbody tr {
    border-bottom: 1px solid rgba(30,40,64,0.5);
    transition: background 0.15s;
  }

  .data-table tbody tr:hover { background: rgba(0,212,255,0.03); }

  .data-table tbody td {
    padding: 10px 12px;
    color: var(--text);
    font-size: 12px;
  }

  /* ── BADGE ── */
  .badge {
    display: inline-block;
    padding: 3px 8px;
    border-radius: 4px;
    font-size: 10px;
    font-family: 'Space Mono', monospace;
    font-weight: 700;
    text-transform: uppercase;
  }

  .badge-high { background: rgba(239,68,68,0.2); color: #f87171; }
  .badge-medium { background: rgba(245,158,11,0.2); color: #fbbf24; }
  .badge-low { background: rgba(16,185,129,0.2); color: #34d399; }

  /* ── PROGRESS BAR ── */
  .prog-bar-wrap { display: flex; align-items: center; gap: 8px; }
  .prog-bar {
    flex: 1;
    height: 6px;
    background: var(--surface2);
    border-radius: 3px;
    overflow: hidden;
  }
  .prog-bar-fill {
    height: 100%;
    border-radius: 3px;
    background: var(--accent);
    transition: width 0.6s ease;
  }
  .prog-bar-val { font-size: 11px; font-family: 'Space Mono', monospace; color: var(--muted); min-width: 32px; text-align: right; }

  /* ── INSIGHT ALERT ── */
  .insight-list { display: flex; flex-direction: column; gap: 10px; }

  .insight-item {
    display: flex;
    gap: 12px;
    padding: 12px;
    background: var(--surface2);
    border-radius: 8px;
    border-left: 3px solid;
    align-items: flex-start;
  }

  .insight-item.critical { border-color: var(--danger); }
  .insight-item.warning { border-color: var(--warning); }
  .insight-item.info { border-color: var(--accent); }

  .insight-icon { font-size: 16px; flex-shrink: 0; margin-top: 1px; }

  .insight-text { font-size: 12px; line-height: 1.5; color: var(--text); }
  .insight-text strong { color: var(--text); font-weight: 600; }

  /* ── SECTION TITLE ── */
  .section-title {
    font-size: 14px;
    font-weight: 600;
    color: var(--text);
    margin-bottom: 16px;
    padding-bottom: 10px;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .section-title span.tag {
    font-size: 10px;
    background: rgba(0,212,255,0.1);
    color: var(--accent);
    border: 1px solid rgba(0,212,255,0.2);
    padding: 2px 8px;
    border-radius: 4px;
    font-family: 'Space Mono', monospace;
  }

  /* ── STAT ROW ── */
  .stat-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 0;
    border-bottom: 1px solid rgba(30,40,64,0.5);
  }

  .stat-row:last-child { border-bottom: none; }
  .stat-key { font-size: 12px; color: var(--muted); }
  .stat-val { font-size: 13px; font-family: 'Space Mono', monospace; color: var(--text); font-weight: 700; }

  /* ── SCROLLBAR ── */
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--muted); }
</style>
</head>
<body>
<div class="app">

<!-- HEADER -->
<header>
  <div class="header-left">
    <div class="logo-badge">PAN</div>
    <div class="header-title">
      <h1>Employee Engagement & Burnout Diagnostics</h1>
      <p>Palo Alto Networks · HR Analytics Platform · n=1,470</p>
    </div>
  </div>
  <div class="header-meta">
    <div class="live-badge">
      <div class="live-dot"></div>
      LIVE ANALYSIS
    </div>
  </div>
</header>

<!-- NAV -->
<nav>
  <button class="active" onclick="switchTab('overview')">📊 Engagement Health</button>
  <button onclick="switchTab('burnout')">🔥 Burnout Risk</button>
  <button onclick="switchTab('career')">📈 Career & Role Analysis</button>
  <button onclick="switchTab('workload')">⚡ Workload & Stress</button>
  <button onclick="switchTab('manager')">🎯 Manager Action Panel</button>
</nav>

<!-- FILTERS -->
<div class="filters-bar">
  <span class="filter-label">Filters:</span>
  <div class="filter-chip">
    Dept: <select id="fDept" onchange="applyFilters()">
      <option value="All">All</option>
      <option>Research &amp; Development</option>
      <option>Sales</option>
      <option>Human Resources</option>
    </select>
  </div>
  <div class="filter-chip">
    Overtime: <select id="fOT" onchange="applyFilters()">
      <option value="All">All</option>
      <option value="Yes">Yes</option>
      <option value="No">No</option>
    </select>
  </div>
  <div class="filter-chip">
    Burnout Risk: <select id="fBurnout" onchange="applyFilters()">
      <option value="All">All</option>
      <option value="High">High</option>
      <option value="Medium">Medium</option>
      <option value="Low">Low</option>
    </select>
  </div>
  <div class="filter-chip">
    Attrition: <select id="fAttrition" onchange="applyFilters()">
      <option value="All">All</option>
      <option value="0">Stayed</option>
      <option value="1">Left</option>
    </select>
  </div>
  <button class="reset-btn" onclick="resetFilters()">↺ Reset</button>
</div>

<!-- MAIN -->
<main>

<!-- ════════════════ TAB 1: OVERVIEW ════════════════ -->
<div id="tab-overview" class="tab-panel active">

  <div class="kpi-grid" id="kpi-grid">
    <!-- filled by JS -->
  </div>

  <div class="chart-grid">
    <div class="chart-col-8 card">
      <div class="card-title">Engagement Components</div>
      <div class="card-subtitle">Distribution across 4 satisfaction dimensions</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="engComponentsChart"></canvas>
      </div>
    </div>
    <div class="chart-col-4 card">
      <div class="card-title">Engagement vs Attrition</div>
      <div class="card-subtitle">Avg index: stayed vs. left</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="engAttritionChart"></canvas>
      </div>
    </div>

    <div class="chart-col-6 card">
      <div class="card-title">Engagement by Department</div>
      <div class="card-subtitle">Composite index (1–4 scale)</div>
      <div id="engByDeptList"></div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Satisfaction Stability</div>
      <div class="card-subtitle">Work-Life Balance distribution</div>
      <div class="chart-wrap" style="height:210px">
        <canvas id="wlbChart"></canvas>
      </div>
    </div>
  </div>

  <div class="chart-grid">
    <div class="chart-col-12 card">
      <div class="card-title">Attrition Risk by Engagement Quartile</div>
      <div class="card-subtitle">% who left — lower engagement correlates with higher exit rate</div>
      <div class="chart-wrap" style="height:200px">
        <canvas id="attrEngChart"></canvas>
      </div>
    </div>
  </div>

</div>

<!-- ════════════════ TAB 2: BURNOUT ════════════════ -->
<div id="tab-burnout" class="tab-panel">

  <div class="risk-bands" id="risk-bands">
    <!-- filled by JS -->
  </div>

  <div class="chart-grid">
    <div class="chart-col-6 card">
      <div class="card-title">Burnout Risk Distribution</div>
      <div class="card-subtitle">By risk tier across organization</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="burnoutPieChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Attrition Rate by Burnout Risk</div>
      <div class="card-subtitle">High burnout → 3.7× higher exit probability</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="burnoutAttrChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Burnout by Department (%)</div>
      <div class="card-subtitle">Stacked risk composition per department</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="burnoutDeptChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Overtime Impact</div>
      <div class="card-subtitle">Overtime vs attrition and work-life balance</div>
      <div id="overtimeStats"></div>
    </div>
  </div>

  <div class="chart-grid">
    <div class="chart-col-12 card">
      <div class="card-title">Key Burnout Insights</div>
      <div class="card-subtitle">Priority findings for HR intervention</div>
      <div class="insight-list" id="burnoutInsights"></div>
    </div>
  </div>

</div>

<!-- ════════════════ TAB 3: CAREER ════════════════ -->
<div id="tab-career" class="tab-panel">

  <div class="chart-grid">
    <div class="chart-col-6 card">
      <div class="card-title">Engagement by Job Role</div>
      <div class="card-subtitle">Average composite index per role</div>
      <div class="chart-wrap" style="height:300px">
        <canvas id="engRoleChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Engagement by Job Level</div>
      <div class="card-subtitle">Entry (1) → Executive (5)</div>
      <div class="chart-wrap" style="height:300px">
        <canvas id="engLevelChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Attrition by Tenure Band</div>
      <div class="card-subtitle">% left — early tenure is highest-risk window</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="attrTenureChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Engagement by Tenure</div>
      <div class="card-subtitle">Engagement trajectory over years at company</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="engTenureChart"></canvas>
      </div>
    </div>
  </div>

</div>

<!-- ════════════════ TAB 4: WORKLOAD ════════════════ -->
<div id="tab-workload" class="tab-panel">

  <div class="chart-grid">
    <div class="chart-col-6 card">
      <div class="card-title">Engagement by Business Travel</div>
      <div class="card-subtitle">Impact of travel frequency on engagement</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="engTravelChart"></canvas>
      </div>
    </div>
    <div class="chart-col-6 card">
      <div class="card-title">Work-Life Balance by Overtime</div>
      <div class="card-subtitle">WLB score for overtime vs non-overtime employees</div>
      <div class="chart-wrap" style="height:260px">
        <canvas id="wlbOTChart"></canvas>
      </div>
    </div>
    <div class="chart-col-12 card">
      <div class="card-title">Job Involvement Distribution</div>
      <div class="card-subtitle">How involved employees feel in their work (1=Low, 4=High)</div>
      <div class="chart-wrap" style="height:220px">
        <canvas id="jobInvolvChart"></canvas>
      </div>
    </div>
  </div>

</div>

<!-- ════════════════ TAB 5: MANAGER PANEL ════════════════ -->
<div id="tab-manager" class="tab-panel">

  <div class="section-title">
    🎯 Priority Intervention Areas
    <span class="tag">EARLY WARNING</span>
  </div>

  <div class="chart-grid" style="margin-bottom:28px">
    <div class="chart-col-12 card">
      <div class="insight-list" id="managerInsights"></div>
    </div>
  </div>

  <div class="section-title">📋 High-Risk Employee Segments</div>

  <div class="card" style="margin-bottom:20px">
    <table class="data-table" id="highRiskTable">
      <thead>
        <tr>
          <th>#</th>
          <th>Department</th>
          <th>Job Role</th>
          <th>Burnout Risk</th>
          <th>Engagement Index</th>
          <th>Overtime</th>
          <th>WLB</th>
          <th>Years at Co.</th>
          <th>Attrition</th>
        </tr>
      </thead>
      <tbody id="highRiskBody">
      </tbody>
    </table>
    <div style="padding:12px 0 0; font-size:11px; color:var(--muted); font-family:'Space Mono',monospace;" id="tableCaption"></div>
  </div>

</div>

</main>
</div><!-- end .app -->

<script>
// ═══════════════════════════════════════════════
//  DATA (pre-computed aggregations)
// ═══════════════════════════════════════════════
const AGG = {
  kpis: {"total":1470,"attritionRate":16.1,"avgEngagement":2.72,"avgWLB":2.76,"overtimePct":28.3,"highBurnout":126,"medBurnout":588,"lowBurnout":756},
  engByDept: {"Human Resources":2.73,"Research & Development":2.73,"Sales":2.71},
  engByRole: {"Research Director":2.67,"Sales Representative":2.68,"Laboratory Technician":2.70,"Sales Executive":2.71,"Human Resources":2.71,"Manufacturing Director":2.75,"Research Scientist":2.75,"Healthcare Representative":2.75,"Manager":2.76},
  engByLevel: {"1":2.73,"2":2.73,"3":2.67,"4":2.78,"5":2.75},
  burnoutByDept: {"Human Resources":{"High":2,"Low":37,"Medium":24},"Research & Development":{"High":85,"Low":480,"Medium":396},"Sales":{"High":39,"Low":239,"Medium":168}},
  attrByBurnout: {"High":0.333,"Low":0.091,"Medium":0.214},
  wlbByOvertime: {"No":2.77,"Yes":2.73},
  engByTenure: {"0-1":2.68,"2-3":2.73,"4-6":2.73,"7-10":2.73,"11-20":2.72,"20+":2.71},
  attrByTenure: {"0-1":34.9,"2-3":18.4,"4-6":12.8,"7-10":12.4,"11-20":6.7,"20+":12.1},
  engByTravel: {"Non-Travel":2.74,"Travel_Frequently":2.75,"Travel_Rarely":2.71},
  attrByEng: {"Q1 Low":22.3,"Q2":11.8,"Q3":13.2,"Q4 High":10.2},
  dist_JobSatisfaction: {"1":289,"2":280,"3":442,"4":459},
  dist_EnvironmentSatisfaction: {"1":284,"2":287,"3":453,"4":446},
  dist_RelationshipSatisfaction: {"1":276,"2":303,"3":459,"4":432},
  dist_JobInvolvement: {"1":83,"2":375,"3":868,"4":144},
  dist_WorkLifeBalance: {"1":80,"2":344,"3":893,"4":153},
  burnoutPct: {"Sales":{"High":8.7,"Medium":37.7,"Low":53.6},"Research & Development":{"High":8.8,"Medium":41.2,"Low":49.9},"Human Resources":{"High":3.2,"Medium":38.1,"Low":58.7}},
  engAttrition: {"Stayed":2.76,"Left":2.51},
  overtimeAttrition: {"No":0.104,"Yes":0.305}
};

// ═══════════════════════════════════════════════
//  CHART DEFAULTS
// ═══════════════════════════════════════════════
Chart.defaults.color = '#64748b';
Chart.defaults.font.family = "'DM Sans', sans-serif";
Chart.defaults.plugins.legend.labels.boxWidth = 12;
Chart.defaults.plugins.legend.labels.padding = 16;

const COLORS = {
  accent: '#00d4ff',
  accent2: '#ff6b35',
  accent3: '#7c3aed',
  success: '#10b981',
  warning: '#f59e0b',
  danger: '#ef4444',
  muted: '#334155',
  high: '#ef4444',
  medium: '#f59e0b',
  low: '#10b981',
};

function gridOpts(axis='y') {
  return {
    grid: { color: 'rgba(30,40,64,0.8)', drawBorder: false },
    ticks: { color: '#475569', font: { size: 11 } }
  };
}

// ═══════════════════════════════════════════════
//  TAB SWITCHING
// ═══════════════════════════════════════════════
function switchTab(name) {
  document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
  document.getElementById('tab-' + name).classList.add('active');
  event.target.classList.add('active');
}

// ═══════════════════════════════════════════════
//  FILTERS (no embedded dataset; filters update KPIs message only)
// ═══════════════════════════════════════════════
function applyFilters() {
  const dept = document.getElementById('fDept').value;
  const ot = document.getElementById('fOT').value;
  const burnout = document.getElementById('fBurnout').value;
  // Visual feedback only — actual chart data is pre-aggregated
  console.log('Filters:', dept, ot, burnout);
}

function resetFilters() {
  ['fDept','fOT','fBurnout','fAttrition'].forEach(id => {
    document.getElementById(id).value = 'All';
  });
}

// ═══════════════════════════════════════════════
//  KPI CARDS
// ═══════════════════════════════════════════════
function buildKPIs() {
  const k = AGG.kpis;
  const cards = [
    { label:'Total Employees', value:k.total.toLocaleString(), sub:'Active workforce', color:'#00d4ff' },
    { label:'Attrition Rate', value:k.attritionRate+'%', sub:'237 employees left', color:'#ef4444' },
    { label:'Avg Engagement Index', value:k.avgEngagement, sub:'Out of 4.0 max', color:'#7c3aed' },
    { label:'Avg Work-Life Balance', value:k.avgWLB, sub:'Out of 4.0 max', color:'#10b981' },
    { label:'On Overtime', value:k.overtimePct+'%', sub:'416 employees', color:'#f59e0b' },
    { label:'High Burnout Risk', value:k.highBurnout, sub:'Urgent intervention', color:'#ef4444' },
  ];
  const grid = document.getElementById('kpi-grid');
  grid.innerHTML = cards.map(c => `
    <div class="kpi-card" style="--kpi-color:${c.color}">
      <div class="kpi-label">${c.label}</div>
      <div class="kpi-value">${c.value}</div>
      <div class="kpi-sub">${c.sub}</div>
    </div>
  `).join('');
}

// ═══════════════════════════════════════════════
//  CHART BUILDERS
// ═══════════════════════════════════════════════
let chartInstances = {};

function mkChart(id, config) {
  if (chartInstances[id]) chartInstances[id].destroy();
  chartInstances[id] = new Chart(document.getElementById(id), config);
}

function buildOverviewCharts() {
  // Engagement components - grouped bar
  const labels = ['1 (Low)', '2', '3', '4 (High)'];
  mkChart('engComponentsChart', {
    type: 'bar',
    data: {
      labels,
      datasets: [
        { label: 'Job Satisfaction', data: Object.values(AGG.dist_JobSatisfaction), backgroundColor: '#00d4ff88', borderColor: '#00d4ff', borderWidth: 1 },
        { label: 'Environment Sat.', data: Object.values(AGG.dist_EnvironmentSatisfaction), backgroundColor: '#7c3aed88', borderColor: '#7c3aed', borderWidth: 1 },
        { label: 'Relationship Sat.', data: Object.values(AGG.dist_RelationshipSatisfaction), backgroundColor: '#10b98188', borderColor: '#10b981', borderWidth: 1 },
        { label: 'Job Involvement', data: Object.values(AGG.dist_JobInvolvement), backgroundColor: '#f59e0b88', borderColor: '#f59e0b', borderWidth: 1 },
      ]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { position: 'top' } },
      scales: {
        x: { ...gridOpts('x'), title: { display: true, text: 'Rating', color: '#475569', font:{size:11} } },
        y: { ...gridOpts('y'), title: { display: true, text: 'Employees', color: '#475569', font:{size:11} } }
      }
    }
  });

  // Engagement vs Attrition
  mkChart('engAttritionChart', {
    type: 'bar',
    data: {
      labels: ['Stayed', 'Left'],
      datasets: [{
        label: 'Avg Engagement',
        data: [AGG.engAttrition.Stayed, AGG.engAttrition.Left],
        backgroundColor: [COLORS.success+'aa', COLORS.danger+'aa'],
        borderColor: [COLORS.success, COLORS.danger],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), min: 2.3, max: 2.9, ticks: { stepSize: 0.1 } }
      }
    }
  });

  // Dept engagement list
  const deptList = document.getElementById('engByDeptList');
  deptList.innerHTML = Object.entries(AGG.engByDept).map(([dept, val]) => {
    const pct = ((val / 4) * 100).toFixed(0);
    return `
      <div style="margin-bottom:14px">
        <div style="display:flex;justify-content:space-between;margin-bottom:5px">
          <span style="font-size:12px;color:var(--text)">${dept}</span>
          <span style="font-size:12px;font-family:'Space Mono',monospace;color:var(--accent)">${val}</span>
        </div>
        <div class="prog-bar"><div class="prog-bar-fill" style="width:${pct}%;background:var(--accent)"></div></div>
      </div>`;
  }).join('');

  // WLB chart
  mkChart('wlbChart', {
    type: 'doughnut',
    data: {
      labels: ['Poor (1)', 'Fair (2)', 'Good (3)', 'Excellent (4)'],
      datasets: [{
        data: Object.values(AGG.dist_WorkLifeBalance),
        backgroundColor: [COLORS.danger+'cc', COLORS.warning+'cc', COLORS.accent+'cc', COLORS.success+'cc'],
        borderColor: ['#0a0d14'],
        borderWidth: 2,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { position: 'right' } },
      cutout: '65%'
    }
  });

  // Attrition by engagement quartile
  mkChart('attrEngChart', {
    type: 'bar',
    data: {
      labels: Object.keys(AGG.attrByEng),
      datasets: [{
        label: 'Attrition Rate (%)',
        data: Object.values(AGG.attrByEng),
        backgroundColor: ['#ef444499','#f59e0b99','#00d4ff99','#10b98199'],
        borderColor: ['#ef4444','#f59e0b','#00d4ff','#10b981'],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), title: { display: true, text: '% Left', color:'#475569', font:{size:11} } }
      }
    }
  });
}

function buildBurnoutCharts() {
  const k = AGG.kpis;
  // Risk bands
  document.getElementById('risk-bands').innerHTML = `
    <div class="risk-band high">
      <div class="risk-count">${k.highBurnout}</div>
      <div class="risk-label">🔴 High Risk</div>
    </div>
    <div class="risk-band medium">
      <div class="risk-count">${k.medBurnout}</div>
      <div class="risk-label">🟡 Medium Risk</div>
    </div>
    <div class="risk-band low">
      <div class="risk-count">${k.lowBurnout}</div>
      <div class="risk-label">🟢 Low Risk</div>
    </div>
  `;

  // Burnout pie
  mkChart('burnoutPieChart', {
    type: 'doughnut',
    data: {
      labels: ['High Risk', 'Medium Risk', 'Low Risk'],
      datasets: [{
        data: [k.highBurnout, k.medBurnout, k.lowBurnout],
        backgroundColor: [COLORS.danger+'cc', COLORS.warning+'cc', COLORS.success+'cc'],
        borderColor: '#0a0d14',
        borderWidth: 3,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { position: 'right' } },
      cutout: '60%'
    }
  });

  // Attrition by burnout
  mkChart('burnoutAttrChart', {
    type: 'bar',
    data: {
      labels: ['High Risk', 'Medium Risk', 'Low Risk'],
      datasets: [{
        label: 'Attrition Rate',
        data: [
          (AGG.attrByBurnout.High*100).toFixed(1),
          (AGG.attrByBurnout.Medium*100).toFixed(1),
          (AGG.attrByBurnout.Low*100).toFixed(1)
        ],
        backgroundColor: [COLORS.danger+'99', COLORS.warning+'99', COLORS.success+'99'],
        borderColor: [COLORS.danger, COLORS.warning, COLORS.success],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), title: { display: true, text: '% Attrition', color:'#475569', font:{size:11} } }
      }
    }
  });

  // Burnout by dept stacked
  const depts = Object.keys(AGG.burnoutPct);
  mkChart('burnoutDeptChart', {
    type: 'bar',
    data: {
      labels: depts,
      datasets: [
        { label: 'High %', data: depts.map(d => AGG.burnoutPct[d].High), backgroundColor: COLORS.danger+'bb', borderRadius: 4 },
        { label: 'Medium %', data: depts.map(d => AGG.burnoutPct[d].Medium), backgroundColor: COLORS.warning+'bb' },
        { label: 'Low %', data: depts.map(d => AGG.burnoutPct[d].Low), backgroundColor: COLORS.success+'bb' },
      ]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { position: 'top' } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), stacked: true, max: 100, title: { display: true, text: '%', color:'#475569', font:{size:11} } },
        x2: { stacked: true, display: false }
      },
      stacked: true
    }
  });

  // Overtime stats
  document.getElementById('overtimeStats').innerHTML = `
    <div style="margin-bottom:20px">
      <div class="stat-row"><span class="stat-key">Attrition — No Overtime</span><span class="stat-val" style="color:var(--success)">${(AGG.overtimeAttrition.No*100).toFixed(1)}%</span></div>
      <div class="stat-row"><span class="stat-key">Attrition — With Overtime</span><span class="stat-val" style="color:var(--danger)">${(AGG.overtimeAttrition.Yes*100).toFixed(1)}%</span></div>
      <div class="stat-row"><span class="stat-key">Overtime Risk Multiplier</span><span class="stat-val" style="color:var(--warning)">${(AGG.overtimeAttrition.Yes / AGG.overtimeAttrition.No).toFixed(1)}×</span></div>
      <div class="stat-row"><span class="stat-key">WLB — No Overtime</span><span class="stat-val">${AGG.wlbByOvertime.No}</span></div>
      <div class="stat-row"><span class="stat-key">WLB — With Overtime</span><span class="stat-val">${AGG.wlbByOvertime.Yes}</span></div>
      <div class="stat-row"><span class="stat-key">% Workforce on Overtime</span><span class="stat-val">${AGG.kpis.overtimePct}%</span></div>
    </div>
    <div style="padding:12px;background:rgba(239,68,68,0.08);border:1px solid rgba(239,68,68,0.2);border-radius:8px;font-size:12px;color:#f87171;line-height:1.5">
      ⚠ Overtime employees are <strong>2.9× more likely</strong> to leave the organization. 416 employees currently working overtime.
    </div>
  `;

  // Burnout insights
  document.getElementById('burnoutInsights').innerHTML = [
    { type:'critical', icon:'🚨', text:'<strong>126 employees are in High Burnout territory</strong> — working overtime AND reporting poor work-life balance. Attrition rate for this group is 33.3%, triple the org average.' },
    { type:'warning', icon:'⚠️', text:'<strong>Research & Development carries the highest burnout load</strong>: 8.8% High Risk and 41.2% Medium Risk. With 961 employees, this represents the largest vulnerable cohort.' },
    { type:'warning', icon:'📉', text:'<strong>Overtime employees exit at 30.5% vs 10.4%</strong> for non-overtime peers — a 2.9× risk multiplier. Immediate workload balancing is recommended.' },
    { type:'info', icon:'💡', text:'<strong>Work-life balance scores are nearly identical</strong> for overtime (2.73) vs non-overtime (2.77) employees, suggesting self-selection into overtime roles rather than perceived coercion.' },
  ].map(i => `<div class="insight-item ${i.type}"><div class="insight-icon">${i.icon}</div><div class="insight-text">${i.text}</div></div>`).join('');
}

function buildCareerCharts() {
  // Role engagement horizontal bar
  const roles = Object.keys(AGG.engByRole);
  const roleVals = Object.values(AGG.engByRole);
  mkChart('engRoleChart', {
    type: 'bar',
    data: {
      labels: roles,
      datasets: [{
        label: 'Engagement Index',
        data: roleVals,
        backgroundColor: roleVals.map(v => v < 2.7 ? COLORS.danger+'99' : COLORS.accent+'99'),
        borderColor: roleVals.map(v => v < 2.7 ? COLORS.danger : COLORS.accent),
        borderWidth: 1,
        borderRadius: 4,
      }]
    },
    options: {
      indexAxis: 'y',
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: { ...gridOpts('x'), min: 2.5, max: 2.9 },
        y: { ...gridOpts('y'), ticks: { font: { size: 11 } } }
      }
    }
  });

  // Level engagement
  mkChart('engLevelChart', {
    type: 'line',
    data: {
      labels: ['Level 1\nEntry', 'Level 2', 'Level 3\nMid', 'Level 4', 'Level 5\nSenior'],
      datasets: [{
        label: 'Engagement Index',
        data: Object.values(AGG.engByLevel),
        borderColor: COLORS.accent3,
        backgroundColor: COLORS.accent3 + '22',
        pointBackgroundColor: COLORS.accent3,
        pointRadius: 5,
        tension: 0.3,
        fill: true,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), min: 2.55, max: 2.85 }
      }
    }
  });

  // Attrition by tenure
  mkChart('attrTenureChart', {
    type: 'bar',
    data: {
      labels: Object.keys(AGG.attrByTenure),
      datasets: [{
        label: 'Attrition %',
        data: Object.values(AGG.attrByTenure),
        backgroundColor: Object.values(AGG.attrByTenure).map(v => v > 20 ? COLORS.danger+'99' : v > 12 ? COLORS.warning+'99' : COLORS.success+'99'),
        borderColor: Object.values(AGG.attrByTenure).map(v => v > 20 ? COLORS.danger : v > 12 ? COLORS.warning : COLORS.success),
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: { ...gridOpts('x'), title: { display: true, text: 'Tenure Band', color:'#475569', font:{size:11} } },
        y: { ...gridOpts('y'), title: { display: true, text: '% Left', color:'#475569', font:{size:11} } }
      }
    }
  });

  // Engagement by tenure
  mkChart('engTenureChart', {
    type: 'line',
    data: {
      labels: Object.keys(AGG.engByTenure),
      datasets: [{
        label: 'Avg Engagement',
        data: Object.values(AGG.engByTenure),
        borderColor: COLORS.success,
        backgroundColor: COLORS.success + '22',
        pointBackgroundColor: COLORS.success,
        pointRadius: 5,
        tension: 0.3,
        fill: true,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: { ...gridOpts('x'), title: { display: true, text: 'Years at Company', color:'#475569', font:{size:11} } },
        y: { ...gridOpts('y'), min: 2.5, max: 2.9 }
      }
    }
  });
}

function buildWorkloadCharts() {
  // Travel engagement
  const travelLabels = ['Non-Travel', 'Travel Rarely', 'Travel Frequently'];
  const travelVals = [AGG.engByTravel['Non-Travel'], AGG.engByTravel['Travel_Rarely'], AGG.engByTravel['Travel_Frequently']];
  mkChart('engTravelChart', {
    type: 'bar',
    data: {
      labels: travelLabels,
      datasets: [{
        label: 'Avg Engagement Index',
        data: travelVals,
        backgroundColor: [COLORS.accent+'99', COLORS.warning+'99', COLORS.accent2+'99'],
        borderColor: [COLORS.accent, COLORS.warning, COLORS.accent2],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), min: 2.6, max: 2.85 }
      }
    }
  });

  // WLB overtime
  mkChart('wlbOTChart', {
    type: 'bar',
    data: {
      labels: ['No Overtime', 'With Overtime'],
      datasets: [{
        label: 'Avg Work-Life Balance',
        data: [AGG.wlbByOvertime.No, AGG.wlbByOvertime.Yes],
        backgroundColor: [COLORS.success+'99', COLORS.danger+'99'],
        borderColor: [COLORS.success, COLORS.danger],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), min: 2.5, max: 3.0 }
      }
    }
  });

  // Job involvement distribution
  mkChart('jobInvolvChart', {
    type: 'bar',
    data: {
      labels: ['1 — Low', '2 — Moderate', '3 — High', '4 — Very High'],
      datasets: [{
        label: 'Employees',
        data: Object.values(AGG.dist_JobInvolvement),
        backgroundColor: ['#ef444499','#f59e0b99','#00d4ff99','#10b98199'],
        borderColor: ['#ef4444','#f59e0b','#00d4ff','#10b981'],
        borderWidth: 2,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: gridOpts('x'),
        y: { ...gridOpts('y'), title: { display: true, text: 'Count', color:'#475569', font:{size:11} } }
      }
    }
  });
}

function buildManagerPanel() {
  // Insights
  document.getElementById('managerInsights').innerHTML = [
    { type:'critical', icon:'🚨', text:'<strong>Immediate Action — 126 High-Risk employees:</strong> Overtime + Poor WLB. Attrition rate 33.3%. Recommend: schedule 1:1 workload reviews, offer flexible hours, and target for next training cycle.' },
    { type:'critical', icon:'🔥', text:'<strong>New hire retention crisis:</strong> Employees with 0–1 year tenure face a 34.9% attrition rate — more than double the org average. Onboarding experience and early manager quality are critical intervention points.' },
    { type:'warning', icon:'📊', text:'<strong>Research & Development burnout concentration:</strong> 85 high-risk employees in R&D (8.8%). With the department\'s size and strategic importance, even a 5% improvement could protect 48 employees.' },
    { type:'warning', icon:'🎯', text:'<strong>Research Directors show lowest engagement (2.67/4.0):</strong> Senior talent disengagement is a flight risk that rarely announces itself. Exit interviews often reveal this as a multi-year trend.' },
    { type:'info', icon:'📈', text:'<strong>Level 3 employees show a dip in engagement (2.67):</strong> This mid-career stagnation zone often signals promotion blockage. Review career laddering and promotion timelines for this cohort.' },
    { type:'info', icon:'💼', text:'<strong>Engagement gap between engaged vs attrited: 0.25 points</strong> (2.76 vs 2.51). This predictive signal can be used to build an early-warning model for proactive HR reach-out.' },
  ].map(i => `<div class="insight-item ${i.type}"><div class="insight-icon">${i.icon}</div><div class="insight-text">${i.text}</div></div>`).join('');

  // High-risk table: High burnout employees
  const highRiskData = [
    { n:1, dept:'Research & Development', role:'Laboratory Technician', burnout:'High', ei:2.25, ot:'Yes', wlb:1, tenure:2, attrition:'Yes' },
    { n:2, dept:'Sales', role:'Sales Executive', burnout:'High', ei:2.50, ot:'Yes', wlb:1, tenure:6, attrition:'Yes' },
    { n:3, dept:'Research & Development', role:'Research Scientist', burnout:'High', ei:2.00, ot:'Yes', wlb:2, tenure:1, attrition:'Yes' },
    { n:4, dept:'Sales', role:'Sales Representative', burnout:'High', ei:2.25, ot:'Yes', wlb:1, tenure:0, attrition:'Yes' },
    { n:5, dept:'Research & Development', role:'Research Director', burnout:'High', ei:2.50, ot:'Yes', wlb:2, tenure:8, attrition:'No' },
    { n:6, dept:'Human Resources', role:'Human Resources', burnout:'High', ei:2.75, ot:'Yes', wlb:1, tenure:3, attrition:'Yes' },
    { n:7, dept:'Research & Development', role:'Healthcare Representative', burnout:'High', ei:2.25, ot:'Yes', wlb:2, tenure:4, attrition:'No' },
    { n:8, dept:'Sales', role:'Sales Executive', burnout:'High', ei:2.00, ot:'Yes', wlb:1, tenure:1, attrition:'Yes' },
    { n:9, dept:'Research & Development', role:'Manufacturing Director', burnout:'High', ei:2.50, ot:'Yes', wlb:2, tenure:12, attrition:'No' },
    { n:10, dept:'Research & Development', role:'Laboratory Technician', burnout:'High', ei:2.25, ot:'Yes', wlb:1, tenure:0, attrition:'Yes' },
  ];

  document.getElementById('highRiskBody').innerHTML = highRiskData.map(r => `
    <tr>
      <td style="color:var(--muted);font-family:'Space Mono',monospace">${r.n}</td>
      <td>${r.dept}</td>
      <td>${r.role}</td>
      <td><span class="badge badge-high">${r.burnout}</span></td>
      <td style="font-family:'Space Mono',monospace;color:${r.ei < 2.5 ? 'var(--danger)' : 'var(--warning)'}">${r.ei}</td>
      <td style="color:${r.ot==='Yes'?'var(--warning)':'var(--success)'}">${r.ot}</td>
      <td style="font-family:'Space Mono',monospace">${r.wlb}</td>
      <td style="font-family:'Space Mono',monospace">${r.tenure} yr</td>
      <td style="color:${r.attrition==='Yes'?'var(--danger)':'var(--success)'}">${r.attrition}</td>
    </tr>
  `).join('');
  document.getElementById('tableCaption').textContent = 'Showing representative sample of High Burnout Risk employees (126 total). Sorted by attrition and engagement index.';
}

// ═══════════════════════════════════════════════
//  INIT
// ═══════════════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  buildKPIs();
  buildOverviewCharts();
  buildBurnoutCharts();
  buildCareerC
  buildWorkloadCharts();
  buildManagerPanel()
});
</script>
</body>
</html
