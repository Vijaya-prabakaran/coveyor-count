# coveyor-count
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ConveyorIQ — Live Monitor</title>
  <link
    href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Syne+Mono&family=Syne:wght@400;600;800&display=swap"
    rel="stylesheet" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mqtt/5.3.4/mqtt.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>

  <style>
    :root {
      --bg: #0a0a08;
      --surface: #111110;
      --panel: #161614;
      --border: #2a2a24;
      --amber: #f5a623;
      --amber2: #ffcc5c;
      --green: #6fcf6f;
      --red: #e05555;
      --muted: #5a5a4a;
      --text: #e8e4d8;
      --ff-head: 'Bebas Neue', sans-serif;
      --ff-mono: 'Syne Mono', monospace;
      --ff-body: 'Syne', sans-serif;
    }

    *,
    *::before,
    *::after {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html {
      scroll-behavior: smooth;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--ff-body);
      min-height: 100vh;
      overflow-x: hidden;
    }

    body::after {
      content: '';
      position: fixed;
      inset: 0;
      background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0, 0, 0, .07) 2px, rgba(0, 0, 0, .07) 4px);
      pointer-events: none;
      z-index: 999;
    }

    .wrap {
      position: relative;
      z-index: 1;
      max-width: 1300px;
      margin: 0 auto;
      padding: 0 28px 80px;
    }

    .topbar {
      display: flex;
      align-items: stretch;
      justify-content: space-between;
      border-bottom: 2px solid var(--amber);
      margin-bottom: 0;
    }

    .brand {
      padding: 22px 0;
      display: flex;
      align-items: baseline;
      gap: 16px;
    }

    .brand-name {
      font-family: var(--ff-head);
      font-size: 3rem;
      letter-spacing: 4px;
      color: var(--amber);
      line-height: 1;
    }

    .brand-tag {
      font-family: var(--ff-mono);
      font-size: 0.6rem;
      letter-spacing: 3px;
      color: var(--muted);
      border: 1px solid var(--border);
      padding: 4px 10px;
    }

    .topbar-right {
      display: flex;
      align-items: stretch;
    }

    .topbar-cell {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 0 28px;
      border-left: 1px solid var(--border);
      min-width: 120px;
    }

    .tc-label {
      font-family: var(--ff-mono);
      font-size: 0.58rem;
      letter-spacing: 3px;
      color: var(--muted);
      text-transform: uppercase;
      margin-bottom: 4px;
    }

    .tc-val {
      font-family: var(--ff-mono);
      font-size: 1rem;
      color: var(--text);
      letter-spacing: 1px;
    }

    #clock {
      color: var(--amber);
      font-size: 1.1rem;
      letter-spacing: 2px;
    }

    .status-strip {
      display: flex;
      border-bottom: 1px solid var(--border);
      margin-bottom: 36px;
      flex-wrap: wrap;
    }

    .status-item {
      display: flex;
      align-items: center;
      gap: 10px;
      padding: 10px 22px;
      font-family: var(--ff-mono);
      font-size: 0.65rem;
      letter-spacing: 2px;
      text-transform: uppercase;
      border-right: 1px solid var(--border);
      color: var(--muted);
    }

    .dot {
      width: 7px;
      height: 7px;
      border-radius: 50%;
      flex-shrink: 0;
    }

    .dot.on {
      background: var(--green);
      box-shadow: 0 0 6px var(--green);
      animation: blink 2s infinite;
    }

    .dot.off {
      background: var(--red);
      box-shadow: 0 0 6px var(--red);
    }

    .dot.warn {
      background: var(--amber);
      box-shadow: 0 0 6px var(--amber);
      animation: blink 1.2s infinite;
    }

    @keyframes blink {

      0%,
      100% {
        opacity: 1
      }

      50% {
        opacity: .3
      }
    }

    .main-grid {
      display: grid;
      grid-template-columns: 1fr 320px;
      gap: 24px;
      margin-bottom: 28px;
    }

    @media(max-width:900px) {
      .main-grid {
        grid-template-columns: 1fr;
      }
    }

    .hero {
      background: var(--panel);
      border: 1px solid var(--border);
      border-top: 3px solid var(--amber);
      padding: 40px 44px;
      position: relative;
      overflow: hidden;
    }

    .hero::before {
      content: 'LIVE';
      position: absolute;
      top: -8px;
      right: 24px;
      font-family: var(--ff-head);
      font-size: 9rem;
      color: rgba(245, 166, 35, .04);
      letter-spacing: 10px;
      pointer-events: none;
      line-height: 1;
    }

    .hero-label {
      font-family: var(--ff-mono);
      font-size: 0.63rem;
      letter-spacing: 4px;
      color: var(--muted);
      text-transform: uppercase;
      margin-bottom: 14px;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .hero-label::before {
      content: '';
      display: block;
      width: 20px;
      height: 1px;
      background: var(--amber);
    }

    #liveCount {
      font-family: var(--ff-head);
      font-size: clamp(5rem, 14vw, 9.5rem);
      color: var(--amber);
      line-height: .9;
      letter-spacing: -2px;
      text-shadow: 0 0 60px rgba(245, 166, 35, .2);
      transition: color .15s;
    }

    .hero-unit {
      font-family: var(--ff-mono);
      font-size: 0.7rem;
      letter-spacing: 4px;
      color: var(--muted);
      text-transform: uppercase;
      margin-top: 12px;
    }

    .rate-row {
      margin-top: 28px;
      display: flex;
      gap: 3px;
      align-items: flex-end;
      height: 48px;
    }

    .rate-bar {
      flex: 1;
      background: var(--amber);
      opacity: .12;
      border-radius: 1px;
      min-height: 4px;
      transition: height .3s ease, opacity .3s;
    }

    .rate-bar.active {
      opacity: .85;
    }

    .side-col {
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    .card {
      background: var(--panel);
      border: 1px solid var(--border);
      padding: 24px 26px;
      position: relative;
    }

    .card-label {
      font-family: var(--ff-mono);
      font-size: 0.6rem;
      letter-spacing: 3px;
      color: var(--muted);
      text-transform: uppercase;
      margin-bottom: 10px;
    }

    .card-num {
      font-family: var(--ff-head);
      font-size: 3rem;
      line-height: 1;
      letter-spacing: 1px;
    }

    .card-num.today {
      color: var(--amber2);
    }

    .card-num.reset {
      color: var(--red);
    }

    .card-num.yest {
      color: var(--text);
    }

    .card-num.week {
      color: var(--green);
    }

    .card-sub {
      font-family: var(--ff-mono);
      font-size: 0.62rem;
      color: var(--muted);
      margin-top: 6px;
      letter-spacing: 1px;
    }

    .card-badge {
      position: absolute;
      top: 18px;
      right: 18px;
      font-family: var(--ff-mono);
      font-size: 0.6rem;
      letter-spacing: 1px;
      padding: 3px 9px;
    }

    .badge-up {
      background: rgba(111, 207, 111, .1);
      color: var(--green);
      border: 1px solid rgba(111, 207, 111, .25);
    }

    .badge-down {
      background: rgba(224, 85, 85, .1);
      color: var(--red);
      border: 1px solid rgba(224, 85, 85, .25);
    }

    .badge-neutral {
      background: rgba(90, 90, 74, .15);
      color: var(--muted);
      border: 1px solid var(--border);
    }

    .chart-section {
      background: var(--panel);
      border: 1px solid var(--border);
      padding: 28px 32px;
      margin-bottom: 28px;
    }

    .section-hdr {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 22px;
    }

    .section-title {
      font-family: var(--ff-head);
      font-size: 1.3rem;
      letter-spacing: 3px;
      color: var(--text);
    }

    .range-btns {
      display: flex;
      gap: 6px;
    }

    .rbtn {
      font-family: var(--ff-mono);
      font-size: 0.62rem;
      letter-spacing: 1px;
      padding: 6px 13px;
      border: 1px solid var(--border);
      background: transparent;
      color: var(--muted);
      cursor: pointer;
      transition: all .15s;
    }

    .rbtn:hover,
    .rbtn.active {
      border-color: var(--amber);
      color: var(--amber);
      background: rgba(245, 166, 35, .07);
    }

    #chartCanvas {
      max-height: 260px;
    }

    .table-section {
      background: var(--panel);
      border: 1px solid var(--border);
    }

    .table-topbar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 20px 26px;
      border-bottom: 1px solid var(--border);
      flex-wrap: wrap;
      gap: 12px;
    }

    .controls {
      display: flex;
      gap: 10px;
      align-items: center;
    }

    .search {
      font-family: var(--ff-mono);
      font-size: 0.75rem;
      background: var(--surface);
      border: 1px solid var(--border);
      color: var(--text);
      padding: 8px 14px;
      outline: none;
      width: 180px;
      letter-spacing: 1px;
      transition: border-color .2s;
    }

    .search:focus {
      border-color: var(--amber);
    }

    .search::placeholder {
      color: var(--muted);
    }

    .csv-btn {
      font-family: var(--ff-mono);
      font-size: 0.68rem;
      letter-spacing: 1px;
      padding: 8px 18px;
      border: 1px solid var(--amber);
      background: rgba(245, 166, 35, .08);
      color: var(--amber);
      cursor: pointer;
      transition: all .2s;
      display: flex;
      align-items: center;
      gap: 7px;
      white-space: nowrap;
    }

    .csv-btn:hover {
      background: rgba(245, 166, 35, .16);
    }

    table {
      width: 100%;
      border-collapse: collapse;
    }

    thead th {
      font-family: var(--ff-mono);
      font-size: 0.6rem;
      letter-spacing: 3px;
      text-transform: uppercase;
      color: var(--muted);
      padding: 13px 26px;
      text-align: left;
      background: var(--surface);
      border-bottom: 1px solid var(--border);
    }

    tbody td {
      padding: 15px 26px;
      font-family: var(--ff-mono);
      font-size: 0.82rem;
      border-bottom: 1px solid rgba(42, 42, 36, .8);
      transition: background .15s;
      letter-spacing: .5px;
    }

    tbody tr:hover td {
      background: rgba(245, 166, 35, .025);
    }

    tbody tr:last-child td {
      border-bottom: none;
    }

    .tbl-date {
      color: var(--muted);
    }

    .tbl-day {
      color: var(--muted);
      font-size: .72rem;
    }

    .tbl-count {
      color: var(--amber2);
      font-size: .95rem;
    }

    .tbl-bar-cell {
      width: 200px;
    }

    .tbl-bar-wrap {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .tbl-bar-bg {
      flex: 1;
      height: 5px;
      background: var(--border);
      overflow: hidden;
    }

    .tbl-bar-fill {
      height: 100%;
      background: var(--amber);
      transition: width .4s ease;
    }

    .tbl-pct {
      font-size: 0.6rem;
      color: var(--muted);
      min-width: 34px;
      text-align: right;
    }

    .pager {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 14px 26px;
      border-top: 1px solid var(--border);
    }

    .pg-info {
      font-family: var(--ff-mono);
      font-size: 0.62rem;
      color: var(--muted);
      letter-spacing: 1px;
    }

    .pg-btns {
      display: flex;
      gap: 6px;
    }

    .pg-btn {
      font-family: var(--ff-mono);
      font-size: 0.62rem;
      padding: 6px 13px;
      border: 1px solid var(--border);
      background: transparent;
      color: var(--muted);
      cursor: pointer;
      transition: all .15s;
      letter-spacing: 1px;
    }

    .pg-btn:hover:not(:disabled) {
      border-color: var(--amber);
      color: var(--amber);
    }

    .pg-btn:disabled {
      opacity: .25;
      cursor: default;
    }

    @keyframes flash {
      0% {
        color: #fff;
        text-shadow: 0 0 80px rgba(255, 255, 255, .6)
      }

      100% {
        color: var(--amber);
        text-shadow: 0 0 60px rgba(245, 166, 35, .2)
      }
    }

    .flash {
      animation: flash .4s ease-out forwards;
    }

    @media(max-width:640px) {
      .topbar {
        flex-direction: column;
      }

      .topbar-right {
        border-top: 1px solid var(--border);
        width: 100%;
        justify-content: space-around;
      }

      .status-strip {
        flex-wrap: wrap;
      }

      .hero {
        padding: 28px 22px;
      }

      thead th,
      tbody td {
        padding: 12px 16px;
      }

      .tbl-bar-cell {
        display: none;
      }
    }
  </style>
</head>

<body>
  <div class="wrap">

    <div class="topbar">
      <div class="brand">
        <div class="brand-name">CONVEYORIQ</div>
        <div class="brand-tag">Object Monitor</div>
      </div>
      <div class="topbar-right">
        <div class="topbar-cell">
          <div class="tc-label">Time</div>
          <div class="tc-val" id="clock">--:--:--</div>
        </div>
        <div class="topbar-cell">
          <div class="tc-label">Date</div>
          <div class="tc-val" id="dateDisp">--</div>
        </div>
        <div class="topbar-cell">
          <div class="tc-label">MQTT</div>
          <div class="tc-val" id="mqttStatus" style="color:var(--amber)">INIT</div>
        </div>
      </div>
    </div>

    <div class="status-strip">
      <div class="status-item"><span class="dot warn" id="mqttDot"></span> <span id="mqttLabel">Adafruit IO</span></div>
      <div class="status-item"><span class="dot on" id="conveyorDot"></span> Conveyor Active</div>
      <div class="status-item"><span class="dot on"></span> 2 Sensors</div>
      <div class="status-item"><span class="dot on" id="syncDot"></span> <span id="syncLabel">DB Synced</span></div>
    </div>

    <div class="main-grid">
      <div class="hero">
        <div class="hero-label">Live Object Count</div>
        <div id="liveCount">0</div>
        <div class="hero-unit">objects detected today</div>
        <div class="rate-row" id="rateRow"></div>
      </div>
      <div class="side-col">
        <div class="card">
          <div class="card-label">Today's Total</div>
          <div class="card-num today" id="todayNum">0</div>
          <div class="card-sub" id="todayDate">--</div>
        </div>

        <div class="card">
          <div class="card-label">Count After Reset</div>
          <div class="card-num reset" id="resetNum">0</div>
          <div class="card-sub" id="resetSub">current session</div>
        </div>

        <div class="card">
          <div class="card-label">Yesterday</div>
          <div class="card-num yest" id="yesterNum">—</div>
          <div class="card-sub" id="yesterDate">--</div>
          <div class="card-badge badge-neutral" id="deltaBadge">--</div>
        </div>
        <div class="card">
          <div class="card-label">7-Day Avg</div>
          <div class="card-num week" id="weekAvg">—</div>
          <div class="card-sub">objects / day</div>
        </div>
      </div>
    </div>

    <div class="chart-section">
      <div class="section-hdr">
        <div class="section-title">TREND</div>
        <div class="range-btns">
          <button class="rbtn active" onclick="setRange(7,this)">7D</button>
          <button class="rbtn" onclick="setRange(14,this)">14D</button>
          <button class="rbtn" onclick="setRange(30,this)">30D</button>
        </div>
      </div>
      <canvas id="chartCanvas"></canvas>
    </div>

    <div class="table-section">
      <div class="table-topbar">
        <div class="section-title">DATABASE</div>
        <div class="controls">
          <input class="search" type="text" placeholder="Search date…" id="searchIn" oninput="onSearch()" />
          <button class="csv-btn" onclick="exportCSV()">&#x2193; Export CSV</button>
        </div>
      </div>
      <div style="overflow-x:auto">
        <table>
          <thead>
            <tr>
              <th>Date</th>
              <th>Day</th>
              <th>Count</th>
              <th class="tbl-bar-cell">Relative</th>
            </tr>
          </thead>
          <tbody id="tbody"></tbody>
        </table>
      </div>
      <div class="pager">
        <div class="pg-info" id="pgInfo">—</div>
        <div class="pg-btns">
          <button class="pg-btn" id="pgPrev" onclick="goPage(-1)">&#x2190; Prev</button>
          <button class="pg-btn" id="pgNext" onclick="goPage(1)">Next &#x2192;</button>
        </div>
      </div>
    </div>

  </div>

  <script>
    // ═══════════════════════════════════════════════════════════
    //  ADAFRUIT IO CONFIG
    // ═══════════════════════════════════════════════════════════
    const AIO_USERNAME = 'Santhosh_VB';
    const AIO_KEY = KEY;

    // Listen to BOTH feeds
    const FEED_TOTAL = 'total-count';    // Your ESP32 Total count feed
    const FEED_SHIFT = 'conveyor-count'; // Your ESP32 Shift count feed

    const BROKER_URL = 'wss://io.adafruit.com/mqtt';
    const TOPIC_TOTAL = `${AIO_USERNAME}/feeds/${FEED_TOTAL}`;
    const TOPIC_SHIFT = `${AIO_USERNAME}/feeds/${FEED_SHIFT}`;
    // Fetch history from the Total feed so charts don't drop when you reset!
    const AIO_API_URL = `https://io.adafruit.com/api/v2/${AIO_USERNAME}/feeds/${FEED_TOTAL}/data?limit=1000`;

    const DB_KEY = 'conveyoriq_v3_db';
    const STATE_KEY = 'conveyoriq_v3_state';

    // ═══════════════════════════════════════════════════════════
    //  WIFI / CONNECTIVITY-BASED RESET DETECTION
    //  ──────────────────────────────────────────────────────────
    //  If no MQTT message arrives for longer than this duration,
    //  the ESP is assumed to be offline (WiFi lost, power-off, or
    //  reboot).  The NEXT message after the silence = new session.
    //
    //  Tune based on your ESP's publish rate:
    //   • ESP publishes every 1-2s (busy conveyor)        → 60s
    //   • ESP publishes a heartbeat every 10s             → 30s
    //   • ESP only publishes on count (idle can last long)→ 5min+
    // ═══════════════════════════════════════════════════════════
    const POWEROFF_TIMEOUT = 60 * 1000;   // 60 seconds

    // Still used by fetchAIOHistory() when reconstructing cloud history
    const RESET_THRESH = 2;

    function calcTrueDailyTotal(points) {
      if (!points || points.length === 0) return 0;

      let sessionPeak = 0;
      for (let i = 0; i < points.length; i++) {
        sessionPeak = Math.max(sessionPeak, points[i].value);
      }
      return sessionPeak;
    }
    // ── STATE
    let db = loadDB();
    let chartRange = 7;
    let page = 0;
    const PER_PAGE = 10;
    let filtered = [];
    let rateHistory = new Array(20).fill(0);
    let chart;

    let todaySessionPeak = 0;
    let todayOffset = 0;
    let liveCount = 0;
    let resetCount = 0;
    let lastMessageTime = 0;   // timestamp of last MQTT message

    // ── HELPERS
    function todayStr() {
      const d = new Date();
      const y = d.getFullYear();
      const m = String(d.getMonth() + 1).padStart(2, '0');
      const day = String(d.getDate()).padStart(2, '0');
      return `${y}-${m}-${day}`;
    }
    function fmtDay(ds) { return new Date(ds + 'T00:00:00').toLocaleDateString('en-IN', { weekday: 'short' }); }

    // ── LOCALSTORAGE
    function loadDB() {
      try { return JSON.parse(localStorage.getItem(DB_KEY) || '{}'); }
      catch { return {}; }
    }
    function saveDB() { localStorage.setItem(DB_KEY, JSON.stringify(db)); }

    function saveState() {
      localStorage.setItem(STATE_KEY, JSON.stringify({
        date: todayStr(),
        todayOffset,
        todaySessionPeak,
        liveCount,
        resetCount,
        lastMessageTime
      }));
    }

    function loadState() {
      try {
        const s = JSON.parse(localStorage.getItem(STATE_KEY) || 'null');
        if (s && s.date === todayStr()) {
          todayOffset = s.todayOffset || 0;
          todaySessionPeak = s.todaySessionPeak || 0;
          liveCount = s.liveCount || 0;
          resetCount = s.resetCount || 0;
          lastMessageTime = s.lastMessageTime || 0;
        }
      } catch { }
    }

    // ── ADAFRUIT IO REST API (for cloud history rebuild)
    async function fetchAIOHistory() {
      if (!AIO_USERNAME) return;
      setSyncStatus('SYNCING…', 'warn');

      try {
        const res = await fetch(AIO_API_URL, { headers: { 'X-AIO-Key': AIO_KEY } });
        if (!res.ok) { setSyncStatus('SYNC ERR', 'off'); return; }

        const data = await res.json();
        const sorted = [...data].reverse();
        const byDate = {};
        sorted.forEach(pt => {
          const d = new Date(pt.created_at);
          const date = `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
          const val = parseInt(pt.value, 10);
          if (!isNaN(val)) {
            if (!byDate[date]) byDate[date] = [];
            byDate[date].push({ value: val, created_at: pt.created_at });
          }
        });

        const today = todayStr();
        Object.entries(byDate).forEach(([date, points]) => {
          const trueTotal = calcTrueDailyTotal(points);

          if (date === today) {
            // ── Only update cloud total for today if we have NO local data yet.
            //    Do NOT touch resetCount, offset, or sessionPeak — the live
            //    MQTT stream is the single source of truth for those.
            if (liveCount === 0 && trueTotal > 0) {
              liveCount = trueTotal;
              db[today] = trueTotal;
              saveDB();
              updateHero(liveCount);
            }
          } else {
            db[date] = trueTotal;
          }
        });

        saveDB();
        setSyncStatus('SYNCED', 'on');
        updateSideCards(); refreshTable(); renderChart();
      } catch (err) {
        console.warn('[AIO] Fetch error:', err);
        setSyncStatus('SYNC ERR', 'off');
      }
    }

    // ── MQTT
    function connectMQTT() {
      if (!AIO_USERNAME) { startDemo(); return; }

      loadState();
      if (liveCount > 0) {
        updateHero(liveCount); updateSideCards(); updateResetCard();
        refreshTable(); renderChart();
      }

      fetchAIOHistory();

      const client = mqtt.connect(BROKER_URL, {
        username: AIO_USERNAME,
        password: AIO_KEY,
        clientId: 'ciq_' + Math.random().toString(16).slice(2, 8),
        reconnectPeriod: 5000,
        connectTimeout: 10000,
      });

      client.on('connect', () => {
        setMQTTStatus('ONLINE', 'on', 'var(--green)');
        // Subscribe to BOTH feeds at the same time
        client.subscribe(TOPIC_TOTAL);
        client.subscribe(TOPIC_SHIFT);
      });

      client.on('message', (topic, payload) => {
        // --- NEW: Tell the website the ESP32 is alive! ---
        lastMessageTime = Date.now();
        const convDot = document.getElementById('conveyorDot');
        if (convDot) convDot.className = 'dot on';

        const val = parseInt(payload.toString(), 10);
        if (isNaN(val)) return;

        if (topic === TOPIC_TOTAL) {
          // Update database
          const today = todayStr();
          db[today] = val;
          saveDB();
          document.getElementById('todayNum').textContent = val.toLocaleString();

          // Make the Total Count drive the BIG FONT
          liveCount = val;
          saveState();
          updateHero(val);

          // ✨ THE FIX: Tell the side cards and the badge to recalculate! ✨
          updateSideCards();

          refreshTable();
          renderChart();
        }
        else if (topic === TOPIC_SHIFT) {
          // 1. Check if the ESP32 reset the shift count back to 0
          if (todaySessionPeak > 0 && val < todaySessionPeak) {
            resetCount++;
          }

          // 2. Update our internal memory for the shift
          todaySessionPeak = val;
          saveState();

          // 3. Update ONLY the "Count After Reset" side card
          document.getElementById('resetNum').textContent = val.toLocaleString();

          // 4. Update the "Number of resets" text
          const sub = document.getElementById('resetSub');
          if (resetCount === 0) sub.textContent = 'no resets today';
          else if (resetCount === 1) sub.textContent = 'since 1 reset today';
          else sub.textContent = `since ${resetCount} resets today`;
        }
      });

      client.on('error', () => setMQTTStatus('ERROR', 'off', 'var(--red)'));
      client.on('reconnect', () => setMQTTStatus('RETRY…', 'warn', 'var(--amber)'));
      client.on('offline', () => setMQTTStatus('OFFLINE', 'off', 'var(--red)'));
    }

    function setMQTTStatus(label, dotClass, color) {
      document.getElementById('mqttStatus').textContent = label;
      document.getElementById('mqttStatus').style.color = color;
      document.getElementById('mqttDot').className = 'dot ' + dotClass;
    }
    function setSyncStatus(label, dotClass) {
      document.getElementById('syncLabel').textContent = label;
      document.getElementById('syncDot').className = 'dot ' + dotClass;
    }

    // ═══════════════════════════════════════════════════════════════════════
    //  onNewCount — WIFI / CONNECTIVITY-BASED RESET DETECTION
    //  ──────────────────────────────────────────────────────────────────────
    //  Reset is detected ONLY by MQTT silence (no value-comparison logic).
    //  If the broker was silent for > POWEROFF_TIMEOUT, the incoming message
    //  is treated as the first tick of a brand-new session.
    // ═══════════════════════════════════════════════════════════════════════

    // ── HERO
    function updateHero(count) {
      const el = document.getElementById('liveCount');
      el.textContent = count.toLocaleString();
      el.classList.remove('flash');
      void el.offsetWidth;
      el.classList.add('flash');

      const last = rateHistory[rateHistory.length - 1] || 0;
      const delta = Math.max(0, count - last);
      rateHistory.push(delta);
      rateHistory = rateHistory.slice(-20);
      renderRateBars();
    }

    function renderRateBars() {
      const max = Math.max(...rateHistory, 1);
      document.getElementById('rateRow').innerHTML = rateHistory.map((v, i) => {
        const h = Math.max(4, Math.round((v / max) * 44));
        const cls = i === rateHistory.length - 1 ? ' active' : '';
        return `<div class="rate-bar${cls}" style="height:${h}px"></div>`;
      }).join('');
    }

    // ── Count After Reset card
    function updateResetCard() {
      document.getElementById('resetNum').textContent = todaySessionPeak.toLocaleString();
      const sub = document.getElementById('resetSub');
      if (resetCount === 0) sub.textContent = 'no resets today';
      else if (resetCount === 1) sub.textContent = 'since 1 reset today';
      else sub.textContent = `since ${resetCount} resets today`;
    }

    // ── SIDE CARDS
    function updateSideCards() {
      const today = todayStr();
      const sorted = Object.keys(db).sort().reverse();

      document.getElementById('todayNum').textContent = (db[today] || 0).toLocaleString();
      document.getElementById('todayDate').textContent = today;

      const yestKey = sorted.find(k => k < today);
      if (yestKey) {
        document.getElementById('yesterNum').textContent = db[yestKey].toLocaleString();
        document.getElementById('yesterDate').textContent = yestKey;
        const delta = (db[today] || 0) - db[yestKey];
        const badge = document.getElementById('deltaBadge');
        badge.textContent = (delta >= 0 ? '▲ +' : '▼ ') + Math.abs(delta).toLocaleString();
        badge.className = 'card-badge ' + (delta > 0 ? 'badge-up' : delta < 0 ? 'badge-down' : 'badge-neutral');
      }

      const past = sorted.filter(k => k < today).slice(0, 7);
      if (past.length) {
        const avg = Math.round(past.reduce((s, k) => s + db[k], 0) / past.length);
        document.getElementById('weekAvg').textContent = avg.toLocaleString();
      }
    }

    // ── CHART
    function renderChart() {
      const sorted = Object.keys(db).sort();
      const slice = sorted.slice(-chartRange);
      const labels = slice.map(k => k.slice(5));
      const counts = slice.map(k => db[k]);
      const today = todayStr();
      const colors = slice.map(k => k === today ? 'rgba(245,166,35,.92)' : 'rgba(245,166,35,.32)');

      if (chart) chart.destroy();

      chart = new Chart(document.getElementById('chartCanvas'), {
        type: 'bar',
        data: {
          labels,
          datasets: [{
            label: 'Objects',
            data: counts,
            backgroundColor: colors,
            borderColor: 'rgba(245,166,35,.5)',
            borderWidth: 1,
            borderRadius: 2,
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: false },
            tooltip: {
              backgroundColor: '#161614', borderColor: '#2a2a24', borderWidth: 1,
              titleColor: '#e8e4d8', bodyColor: '#5a5a4a',
              titleFont: { family: 'Syne Mono' }, bodyFont: { family: 'Syne Mono' },
              callbacks: { label: ctx => ` ${ctx.parsed.y.toLocaleString()} objects` }
            }
          },
          scales: {
            x: { grid: { color: 'rgba(42,42,36,.5)' }, ticks: { color: '#5a5a4a', font: { family: 'Syne Mono', size: 10 } } },
            y: { grid: { color: 'rgba(42,42,36,.5)' }, ticks: { color: '#5a5a4a', font: { family: 'Syne Mono', size: 10 } } }
          }
        }
      });
    }

    function setRange(n, btn) {
      chartRange = n;
      document.querySelectorAll('.rbtn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      renderChart();
    }

    // ── TABLE
    function refreshTable() {
      const q = document.getElementById('searchIn').value.toLowerCase();
      const sorted = Object.keys(db).sort().reverse();
      filtered = sorted.filter(k => k.toLowerCase().includes(q));
      page = 0;
      renderTable();
    }

    function renderTable() {
      const start = page * PER_PAGE;
      const end = Math.min(start + PER_PAGE, filtered.length);
      const slice = filtered.slice(start, end);
      const maxVal = Math.max(...Object.values(db), 1);
      const today = todayStr();

      document.getElementById('tbody').innerHTML = slice.map(k => {
        const count = db[k] || 0;
        const pct = Math.round((count / maxVal) * 100);
        const isToday = k === today;
        return `<tr>
      <td class="tbl-date" style="${isToday ? 'color:var(--amber)' : ''}">${k}${isToday ? '&nbsp;◀' : ''}</td>
      <td class="tbl-day">${fmtDay(k)}</td>
      <td class="tbl-count">${count.toLocaleString()}</td>
      <td class="tbl-bar-cell">
        <div class="tbl-bar-wrap">
          <div class="tbl-bar-bg"><div class="tbl-bar-fill" style="width:${pct}%"></div></div>
          <div class="tbl-pct">${pct}%</div>
        </div>
      </td>
    </tr>`;
      }).join('') || `<tr><td colspan="4" style="text-align:center;padding:30px;font-family:var(--ff-mono);font-size:.7rem;letter-spacing:2px;color:var(--muted)">NO RECORDS FOUND</td></tr>`;

      document.getElementById('pgInfo').textContent =
        filtered.length ? `${start + 1}–${end} of ${filtered.length} records` : '0 records';
      document.getElementById('pgPrev').disabled = page === 0;
      document.getElementById('pgNext').disabled = end >= filtered.length;
    }

    function onSearch() { page = 0; refreshTable(); }
    function goPage(d) { page += d; renderTable(); }

    // ── CSV EXPORT
    function exportCSV() {
      const sorted = Object.keys(db).sort().reverse();
      const csv = 'Date,Day,Count\n' + sorted.map(k => `${k},${fmtDay(k)},${db[k]}`).join('\n');
      const a = Object.assign(document.createElement('a'), {
        href: URL.createObjectURL(new Blob([csv], { type: 'text/csv' })),
        download: `conveyor_${todayStr()}.csv`
      });
      a.click();
    }

    // ── CLOCK
    // ── CLOCK & POWER OFF DETECTOR
    function tick() {
      const now = new Date();
      document.getElementById('clock').textContent = now.toLocaleTimeString('en-IN', { hour12: false });
      document.getElementById('dateDisp').textContent = now.toLocaleDateString('en-IN', { day: '2-digit', month: 'short', year: '2-digit' });

      // --- NEW: Check if ESP32 went offline! (60 seconds of silence) ---
      if (lastMessageTime > 0 && (now.getTime() - lastMessageTime > POWEROFF_TIMEOUT)) {
        const convDot = document.getElementById('conveyorDot');
        if (convDot) convDot.className = 'dot off'; // Turn it RED
      }

      const today = todayStr();
      try {
        const s = JSON.parse(localStorage.getItem(STATE_KEY) || 'null');
        if (s && s.date && s.date !== today) {
          todayOffset = 0; todaySessionPeak = 0; liveCount = 0;
          resetCount = 0; lastMessageTime = 0;
          saveState();
          updateHero(0); updateSideCards(); updateResetCard();
          refreshTable(); renderChart();
        }
      } catch { }
    }
    setInterval(tick, 1000); tick();

    // ── DEMO MODE
    function startDemo() {
      if (Object.keys(db).length === 0) {
        const now = new Date();
        for (let i = 29; i >= 1; i--) {
          const d = new Date(now);
          d.setDate(d.getDate() - i);
          db[d.toISOString().split('T')[0]] = Math.floor(Math.random() * 400 + 600);
        }
        saveDB();
      }

      loadState();
      updateHero(liveCount); updateSideCards(); updateResetCard();
      refreshTable(); renderChart();

      let espCounter = todaySessionPeak;
      let tickCount = 0;

      setInterval(() => {
        tickCount++;
        // Simulate a WiFi drop: skip publishing for 70s so silence > timeout
        if (tickCount % 150 === 0) {
          console.log('[DEMO] Simulated ESP power-off / WiFi drop (70s silence)');
          setTimeout(() => { espCounter = 0; }, 70000);
          return;
        }
        espCounter += 1;
        onNewCount(espCounter);
      }, 1000);
    }

    // ── INIT
    document.getElementById('rateRow').innerHTML = '<div class="rate-bar" style="height:4px"></div>'.repeat(20);
    updateSideCards();
    updateResetCard();
    refreshTable();
    renderChart();
    connectMQTT();
  </script>
</body>

</html>
