<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>FinVault — Kubernetes Architecture</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap');

  :root {
    --bg:       #060d1a;
    --grid:     rgba(255,255,255,0.03);
    --blue:     #3b82f6;
    --blue-dim: rgba(59,130,246,0.12);
    --blue-glow:#1d4ed8;
    --teal:     #14b8a6;
    --teal-dim: rgba(20,184,166,0.12);
    --purple:   #8b5cf6;
    --purple-dim:rgba(139,92,246,0.12);
    --green:    #10b981;
    --green-dim:rgba(16,185,129,0.12);
    --orange:   #f97316;
    --orange-dim:rgba(249,115,22,0.12);
    --red:      #ef4444;
    --slate:    #94a3b8;
    --text:     #f1f5f9;
    --text2:    #94a3b8;
    --text3:    #475569;
    --border:   rgba(255,255,255,0.08);
    --mono:     'JetBrains Mono', monospace;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Inter', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Grid background */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(var(--grid) 1px, transparent 1px),
      linear-gradient(90deg, var(--grid) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  /* Radial glow center */
  body::after {
    content: '';
    position: fixed;
    top: 20%;
    left: 50%;
    transform: translateX(-50%);
    width: 900px;
    height: 600px;
    background: radial-gradient(ellipse, rgba(59,130,246,0.07) 0%, transparent 70%);
    pointer-events: none;
    z-index: 0;
  }

  .page {
    position: relative;
    z-index: 1;
    max-width: 1400px;
    margin: 0 auto;
    padding: 40px 32px 80px;
  }

  /* ── HEADER ── */
  .header {
    text-align: center;
    margin-bottom: 56px;
  }
  .logo-row {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 14px;
    margin-bottom: 16px;
  }
  .logo-icon {
    width: 52px; height: 52px;
    background: linear-gradient(135deg, var(--blue), var(--purple));
    border-radius: 14px;
    display: flex; align-items: center; justify-content: center;
    font-size: 26px;
    box-shadow: 0 0 30px rgba(59,130,246,0.4);
  }
  .logo-text {
    font-size: 36px;
    font-weight: 700;
    letter-spacing: -1px;
  }
  .logo-text em { color: var(--blue); font-style: normal; }
  .tagline {
    font-size: 14px;
    color: var(--text2);
    letter-spacing: 0.12em;
    text-transform: uppercase;
    font-weight: 500;
  }
  .pill {
    display: inline-block;
    margin-top: 16px;
    padding: 5px 14px;
    background: var(--blue-dim);
    border: 1px solid rgba(59,130,246,0.3);
    border-radius: 100px;
    font-size: 12px;
    color: var(--blue);
    font-family: var(--mono);
    font-weight: 500;
  }

  /* ── LAYERS ── */
  .arch-wrap {
    display: flex;
    flex-direction: column;
    gap: 0;
  }

  .layer {
    position: relative;
    border-radius: 20px;
    padding: 28px 32px;
    margin-bottom: 8px;
  }

  .layer-label {
    position: absolute;
    top: -12px;
    left: 24px;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    padding: 3px 12px;
    border-radius: 100px;
    font-family: var(--mono);
  }

  /* ── INTERNET LAYER ── */
  .layer-internet {
    background: rgba(249,115,22,0.05);
    border: 1px solid rgba(249,115,22,0.2);
  }
  .layer-internet .layer-label {
    background: rgba(249,115,22,0.15);
    color: var(--orange);
    border: 1px solid rgba(249,115,22,0.3);
  }

  /* ── INGRESS LAYER ── */
  .layer-ingress {
    background: rgba(139,92,246,0.05);
    border: 1px solid rgba(139,92,246,0.2);
  }
  .layer-ingress .layer-label {
    background: rgba(139,92,246,0.15);
    color: var(--purple);
    border: 1px solid rgba(139,92,246,0.3);
  }

  /* ── NAMESPACE (K8s cluster) ── */
  .layer-cluster {
    background: rgba(59,130,246,0.03);
    border: 2px dashed rgba(59,130,246,0.2);
    padding: 32px;
    border-radius: 24px;
  }
  .layer-cluster .layer-label {
    background: rgba(59,130,246,0.15);
    color: var(--blue);
    border: 1px solid rgba(59,130,246,0.3);
    font-size: 11px;
    padding: 5px 16px;
  }

  /* ── SERVICES LAYER ── */
  .layer-services {
    background: rgba(20,184,166,0.04);
    border: 1px solid rgba(20,184,166,0.15);
  }
  .layer-services .layer-label {
    background: rgba(20,184,166,0.12);
    color: var(--teal);
    border: 1px solid rgba(20,184,166,0.25);
  }

  /* ── DATA LAYER ── */
  .layer-data {
    background: rgba(16,185,129,0.04);
    border: 1px solid rgba(16,185,129,0.15);
  }
  .layer-data .layer-label {
    background: rgba(16,185,129,0.12);
    color: var(--green);
    border: 1px solid rgba(16,185,129,0.25);
  }

  /* ── NODES (boxes) ── */
  .nodes-row {
    display: flex;
    gap: 16px;
    align-items: flex-start;
    flex-wrap: wrap;
    justify-content: center;
  }

  .node {
    flex: 1;
    min-width: 180px;
    max-width: 280px;
    background: rgba(255,255,255,0.03);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 20px;
    position: relative;
    transition: transform .2s, box-shadow .2s;
    cursor: default;
  }
  .node:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 32px rgba(0,0,0,0.4);
  }

  .node-icon { font-size: 28px; margin-bottom: 10px; display: block; }
  .node-name {
    font-size: 14px;
    font-weight: 700;
    margin-bottom: 4px;
    letter-spacing: -0.3px;
  }
  .node-type {
    font-size: 10px;
    font-family: var(--mono);
    font-weight: 600;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    padding: 2px 8px;
    border-radius: 6px;
    display: inline-block;
    margin-bottom: 12px;
  }
  .node-detail {
    font-size: 11px;
    color: var(--text2);
    line-height: 1.7;
  }
  .node-detail span {
    display: block;
    font-family: var(--mono);
    color: var(--text3);
    font-size: 10px;
  }

  /* Colour accents per node type */
  .node.blue  { border-color: rgba(59,130,246,0.3);  }
  .node.blue  .node-type { background: var(--blue-dim); color: var(--blue); }
  .node.blue:hover { box-shadow: 0 8px 32px rgba(59,130,246,0.15); }

  .node.teal  { border-color: rgba(20,184,166,0.3);  }
  .node.teal  .node-type { background: var(--teal-dim); color: var(--teal); }
  .node.teal:hover { box-shadow: 0 8px 32px rgba(20,184,166,0.15); }

  .node.purple { border-color: rgba(139,92,246,0.3); }
  .node.purple .node-type { background: var(--purple-dim); color: var(--purple); }
  .node.purple:hover { box-shadow: 0 8px 32px rgba(139,92,246,0.15); }

  .node.green { border-color: rgba(16,185,129,0.3);  }
  .node.green .node-type { background: var(--green-dim); color: var(--green); }
  .node.green:hover { box-shadow: 0 8px 32px rgba(16,185,129,0.15); }

  .node.orange { border-color: rgba(249,115,22,0.3); }
  .node.orange .node-type { background: var(--orange-dim); color: var(--orange); }
  .node.orange:hover { box-shadow: 0 8px 32px rgba(249,115,22,0.15); }

  /* Replica badge */
  .replica-badge {
    position: absolute;
    top: 12px;
    right: 12px;
    font-size: 10px;
    font-family: var(--mono);
    font-weight: 600;
    padding: 2px 8px;
    border-radius: 100px;
    background: rgba(255,255,255,0.06);
    color: var(--text2);
    border: 1px solid var(--border);
  }

  /* ── ARROWS ── */
  .arrow-zone {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 4px 0;
    gap: 48px;
    position: relative;
  }

  .arrow {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
  }
  .arrow-line {
    width: 2px;
    height: 36px;
    position: relative;
  }
  .arrow-line::before {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: 1px;
  }
  .arrow-line::after {
    content: '';
    position: absolute;
    bottom: -1px;
    left: 50%;
    transform: translateX(-50%);
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
  }
  .arrow.https .arrow-line::before { background: var(--orange); box-shadow: 0 0 8px var(--orange); }
  .arrow.https .arrow-line::after  { border-top: 7px solid var(--orange); }
  .arrow.http  .arrow-line::before { background: var(--blue); box-shadow: 0 0 8px var(--blue); }
  .arrow.http  .arrow-line::after  { border-top: 7px solid var(--blue); }
  .arrow.db    .arrow-line::before { background: var(--green); box-shadow: 0 0 8px var(--green); }
  .arrow.db    .arrow-line::after  { border-top: 7px solid var(--green); }
  .arrow.jwt   .arrow-line::before { background: var(--purple); box-shadow: 0 0 8px var(--purple); }
  .arrow.jwt   .arrow-line::after  { border-top: 7px solid var(--purple); }

  .arrow-label {
    font-size: 9px;
    font-family: var(--mono);
    font-weight: 600;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    padding: 2px 8px;
    border-radius: 100px;
    white-space: nowrap;
  }
  .arrow.https .arrow-label { background: rgba(249,115,22,0.15); color: var(--orange); }
  .arrow.http  .arrow-label { background: rgba(59,130,246,0.15);  color: var(--blue); }
  .arrow.db    .arrow-label { background: rgba(16,185,129,0.15);   color: var(--green); }
  .arrow.jwt   .arrow-label { background: rgba(139,92,246,0.15);   color: var(--purple); }

  /* ── CONFIG STRIP (Secrets + ConfigMap) ── */
  .config-strip {
    display: flex;
    gap: 12px;
    margin-top: 16px;
    flex-wrap: wrap;
  }
  .config-node {
    flex: 1;
    min-width: 200px;
    background: rgba(255,255,255,0.02);
    border: 1px dashed rgba(255,255,255,0.1);
    border-radius: 12px;
    padding: 14px 18px;
    display: flex;
    gap: 12px;
    align-items: flex-start;
  }
  .config-icon { font-size: 20px; flex-shrink: 0; }
  .config-name { font-size: 12px; font-weight: 600; margin-bottom: 4px; }
  .config-keys { font-size: 10px; font-family: var(--mono); color: var(--text3); line-height: 1.9; }
  .config-keys span { color: var(--text2); }

  /* ── HPA / AUTOSCALE STRIP ── */
  .hpa-strip {
    display: flex;
    gap: 12px;
    margin-top: 16px;
    flex-wrap: wrap;
  }
  .hpa-node {
    flex: 1;
    min-width: 160px;
    background: rgba(251,191,36,0.04);
    border: 1px solid rgba(251,191,36,0.15);
    border-radius: 12px;
    padding: 12px 16px;
    display: flex;
    gap: 10px;
    align-items: center;
  }
  .hpa-icon { font-size: 18px; flex-shrink: 0; }
  .hpa-name { font-size: 11px; font-weight: 600; color: #fbbf24; }
  .hpa-range { font-size: 10px; font-family: var(--mono); color: var(--text3); margin-top: 2px; }

  /* ── LEGEND ── */
  .legend {
    margin-top: 48px;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
  }
  .legend-title {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: var(--text3);
    grid-column: 1 / -1;
    margin-bottom: 4px;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 10px 14px;
    background: rgba(255,255,255,0.02);
    border: 1px solid var(--border);
    border-radius: 10px;
  }
  .legend-dot {
    width: 10px; height: 10px;
    border-radius: 50%;
    flex-shrink: 0;
  }
  .legend-item-name { font-size: 12px; font-weight: 600; }
  .legend-item-desc { font-size: 10px; color: var(--text3); margin-top: 1px; }

  /* ── STORAGE VISUAL ── */
  .storage-visual {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 10px;
    flex-wrap: wrap;
  }
  .storage-block {
    font-size: 9px;
    font-family: var(--mono);
    padding: 3px 8px;
    border-radius: 6px;
    font-weight: 600;
  }
  .arrow-right {
    color: var(--text3);
    font-size: 12px;
  }

  /* Animated pulse on cluster border */
  @keyframes borderPulse {
    0%, 100% { border-color: rgba(59,130,246,0.2); }
    50%       { border-color: rgba(59,130,246,0.45); }
  }
  .layer-cluster { animation: borderPulse 4s ease-in-out infinite; }

  /* Animated data flow dots */
  .flow-dot {
    width: 6px; height: 6px;
    border-radius: 50%;
    display: inline-block;
    animation: flowAnim 2s ease-in-out infinite;
  }
  @keyframes flowAnim {
    0%,100% { opacity: 0.2; transform: scale(0.8); }
    50%      { opacity: 1;   transform: scale(1.2); }
  }
  .flow-dot:nth-child(2) { animation-delay: 0.4s; }
  .flow-dot:nth-child(3) { animation-delay: 0.8s; }
</style>
</head>
<body>
<div class="page">

  <!-- HEADER -->
  <div class="header">
    <div class="logo-row">
      <div class="logo-icon">🏦</div>
      <div class="logo-text">Fin<em>Vault</em></div>
    </div>
    <div class="tagline">Kubernetes Microservices Architecture</div>
    <div class="pill">namespace: finvault &nbsp;·&nbsp; 3 microservices &nbsp;·&nbsp; 1 statefulset &nbsp;·&nbsp; HPA enabled</div>
  </div>

  <div class="arch-wrap">

    <!-- ══ LAYER 1: INTERNET ══ -->
    <div class="layer layer-internet">
      <div class="layer-label">🌐 External Traffic</div>
      <div class="nodes-row" style="justify-content:center; gap:32px;">
        <div class="node orange" style="max-width:220px; text-align:center;">
          <span class="node-icon">👤</span>
          <div class="node-name">End User</div>
          <div class="node-type">Browser / Mobile</div>
          <div class="node-detail">
            Sends HTTPS requests<br>
            <span>https://finvault.yourdomain.com</span>
          </div>
        </div>
        <div class="node orange" style="max-width:220px; text-align:center;">
          <span class="node-icon">🌍</span>
          <div class="node-name">DNS / CDN</div>
          <div class="node-type">Route 53 / Cloudflare</div>
          <div class="node-detail">
            Resolves domain to<br>LoadBalancer external IP
            <span>A record → 20.100.x.x</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Arrow -->
    <div class="arrow-zone">
      <div class="arrow https">
        <div class="arrow-label">HTTPS :443</div>
        <div class="arrow-line"></div>
      </div>
    </div>

    <!-- ══ LAYER 2: INGRESS / LOAD BALANCER ══ -->
    <div class="layer layer-ingress">
      <div class="layer-label">🔀 Ingress &amp; Load Balancer</div>
      <div class="nodes-row" style="justify-content:center; gap:16px;">

        <div class="node purple" style="max-width:240px;">
          <span class="node-icon">☁️</span>
          <div class="node-name">Cloud LoadBalancer</div>
          <div class="node-type">Service: LoadBalancer</div>
          <div class="node-detail">
            Provisioned by cloud provider<br>
            External IP assigned automatically
            <span>port: 80 / 443</span>
          </div>
        </div>

        <div class="node purple" style="max-width:260px;">
          <span class="node-icon">🔀</span>
          <div class="node-name">Nginx Ingress Controller</div>
          <div class="node-type">ingress.networking.k8s.io/v1</div>
          <div class="node-detail">
            TLS termination (HTTPS → HTTP)<br>
            Path-based routing rules<br>
            Rate limiting: 100 req/min
            <span>/ → frontend-service:80</span>
          </div>
        </div>

        <div class="node purple" style="max-width:200px;">
          <span class="node-icon">🔒</span>
          <div class="node-name">TLS Secret</div>
          <div class="node-type">Secret (tls)</div>
          <div class="node-detail">
            SSL certificate + private key<br>
            Mounted by Ingress controller
            <span>finvault-tls</span>
          </div>
        </div>

      </div>
    </div>

    <!-- Arrow -->
    <div class="arrow-zone">
      <div class="arrow http">
        <div class="arrow-label">HTTP :80</div>
        <div class="arrow-line"></div>
      </div>
    </div>

    <!-- ══ LAYER 3: K8s NAMESPACE ══ -->
    <div class="layer layer-cluster">
      <div class="layer-label">☸️ Kubernetes Namespace — finvault</div>

      <!-- CONFIG + SECRETS -->
      <div class="config-strip">
        <div class="config-node">
          <div class="config-icon">⚙️</div>
          <div>
            <div class="config-name">ConfigMap: finvault-config</div>
            <div class="config-keys">
              DB_HOST <span>postgres-service</span><br>
              DB_PORT <span>5432</span><br>
              AUTH_SERVICE_URL <span>http://auth-service:3001</span><br>
              NODE_ENV <span>production</span>
            </div>
          </div>
        </div>
        <div class="config-node">
          <div class="config-icon">🔑</div>
          <div>
            <div class="config-name">Secret: finvault-secrets</div>
            <div class="config-keys">
              DB_PASSWORD <span>●●●●●●●●●</span><br>
              JWT_SECRET <span>●●●●●●●●●●●●●●●</span><br>
              POSTGRES_USER <span>●●●●●●●●</span><br>
              POSTGRES_PASSWORD <span>●●●●●●●●●</span>
            </div>
          </div>
        </div>
      </div>

      <!-- SERVICE ROW -->
      <div style="margin-top: 24px;">
        <div class="layer layer-services" style="margin-bottom:0;">
          <div class="layer-label">🚀 Stateless Microservices (Deployments)</div>
          <div class="nodes-row">

            <!-- FRONTEND -->
            <div class="node blue">
              <div class="replica-badge">×2 pods</div>
              <span class="node-icon">🖥️</span>
              <div class="node-name">Frontend</div>
              <div class="node-type">Deployment</div>
              <div class="node-detail">
                React SPA + Nginx<br>
                Reverse proxy to services<br>
                <span>image: finvault-frontend</span>
                <span>port: 80 → containerPort 80</span>
                <span>svc: frontend-service (LB)</span>
                <span>cpu: 50m–200m · mem: 64–128Mi</span>
              </div>
            </div>

            <!-- AUTH -->
            <div class="node blue">
              <div class="replica-badge">×2 pods</div>
              <span class="node-icon">🔐</span>
              <div class="node-name">Auth Service</div>
              <div class="node-type">Deployment</div>
              <div class="node-detail">
                Node.js + Express<br>
                Register / Login / Verify JWT<br>
                <span>image: finvault-auth</span>
                <span>port: 3001 (ClusterIP)</span>
                <span>svc: auth-service</span>
                <span>cpu: 100m–300m · mem: 128–256Mi</span>
              </div>
            </div>

            <!-- ACCOUNT -->
            <div class="node blue">
              <div class="replica-badge">×2 pods</div>
              <span class="node-icon">🏦</span>
              <div class="node-name">Account Service</div>
              <div class="node-type">Deployment</div>
              <div class="node-detail">
                Node.js + Express<br>
                Accounts · Deposit · Withdraw<br>
                Transactions (ACID)
                <span>image: finvault-account</span>
                <span>port: 3002 (ClusterIP)</span>
                <span>svc: account-service</span>
                <span>cpu: 100m–300m · mem: 128–256Mi</span>
              </div>
            </div>

          </div>

          <!-- HPA ROW -->
          <div class="hpa-strip">
            <div class="hpa-node">
              <div class="hpa-icon">📈</div>
              <div>
                <div class="hpa-name">HPA: frontend</div>
                <div class="hpa-range">min 2 → max 6 · CPU &gt;60%</div>
              </div>
            </div>
            <div class="hpa-node">
              <div class="hpa-icon">📈</div>
              <div>
                <div class="hpa-name">HPA: auth-service</div>
                <div class="hpa-range">min 2 → max 8 · CPU &gt;70%</div>
              </div>
            </div>
            <div class="hpa-node">
              <div class="hpa-icon">📈</div>
              <div>
                <div class="hpa-name">HPA: account-service</div>
                <div class="hpa-range">min 2 → max 10 · CPU &gt;70%</div>
              </div>
            </div>
          </div>

        </div>
      </div>

      <!-- Arrows down to DB -->
      <div class="arrow-zone" style="margin-top:8px;">
        <div class="arrow db">
          <div class="arrow-label">SQL :5432</div>
          <div class="arrow-line"></div>
        </div>
        <div class="arrow jwt">
          <div class="arrow-label">JWT verify</div>
          <div class="arrow-line"></div>
        </div>
        <div class="arrow db">
          <div class="arrow-label">SQL :5432</div>
          <div class="arrow-line"></div>
        </div>
      </div>

      <!-- DATA LAYER -->
      <div class="layer layer-data" style="margin-bottom:0;">
        <div class="layer-label">🗄️ Persistent Data Layer</div>
        <div class="nodes-row" style="justify-content: center;">

          <div class="node green" style="max-width:260px;">
            <span class="node-icon">🐘</span>
            <div class="node-name">PostgreSQL</div>
            <div class="node-type">StatefulSet</div>
            <div class="node-detail">
              Stable pod name: postgres-0<br>
              Ordered startup / shutdown<br>
              Tables: users, accounts, transactions
              <span>image: postgres:15-alpine</span>
              <span>port: 5432 (ClusterIP)</span>
              <span>svc: postgres-service</span>
              <span>cpu: 250m–500m · mem: 256–512Mi</span>
            </div>
          </div>

          <div class="node green" style="max-width:220px;">
            <span class="node-icon">💾</span>
            <div class="node-name">PersistentVolumeClaim</div>
            <div class="node-type">PVC → PV</div>
            <div class="node-detail">
              postgres-pvc bound to postgres-pv<br>
              Data survives pod restarts
              <span>capacity: 5Gi</span>
              <span>accessMode: ReadWriteOnce</span>
              <span>reclaimPolicy: Retain</span>
            </div>
            <div class="storage-visual">
              <div class="storage-block" style="background:rgba(16,185,129,0.12);color:var(--green);">PVC</div>
              <div class="arrow-right">→</div>
              <div class="storage-block" style="background:rgba(16,185,129,0.06);color:var(--text2);">PV</div>
              <div class="arrow-right">→</div>
              <div class="storage-block" style="background:rgba(255,255,255,0.04);color:var(--text3);">HostPath</div>
            </div>
          </div>

        </div>
      </div>

    </div><!-- end cluster -->

  </div><!-- end arch-wrap -->

  <!-- ── LEGEND ── -->
  <div class="legend">
    <div class="legend-title">K8s Object Reference</div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--orange);box-shadow:0 0 6px var(--orange);"></div>
      <div>
        <div class="legend-item-name">Namespace</div>
        <div class="legend-item-desc">Resource isolation boundary</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--purple);box-shadow:0 0 6px var(--purple);"></div>
      <div>
        <div class="legend-item-name">Ingress + LoadBalancer</div>
        <div class="legend-item-desc">External entry, TLS, routing</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--blue);box-shadow:0 0 6px var(--blue);"></div>
      <div>
        <div class="legend-item-name">Deployment</div>
        <div class="legend-item-desc">Stateless pods + rolling updates</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--green);box-shadow:0 0 6px var(--green);"></div>
      <div>
        <div class="legend-item-name">StatefulSet + PVC</div>
        <div class="legend-item-desc">Postgres with persistent disk</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:#fbbf24;box-shadow:0 0 6px #fbbf24;"></div>
      <div>
        <div class="legend-item-name">HPA</div>
        <div class="legend-item-desc">Auto-scales on CPU / memory</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--teal);box-shadow:0 0 6px var(--teal);"></div>
      <div>
        <div class="legend-item-name">ConfigMap + Secret</div>
        <div class="legend-item-desc">Env config and credentials</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:var(--slate);"></div>
      <div>
        <div class="legend-item-name">ClusterIP Service</div>
        <div class="legend-item-desc">Internal pod-to-pod DNS routing</div>
      </div>
    </div>

    <div class="legend-item">
      <div class="legend-dot" style="background:#f43f5e;box-shadow:0 0 6px #f43f5e;"></div>
      <div>
        <div class="legend-item-name">initContainers</div>
        <div class="legend-item-desc">Wait for deps before startup</div>
      </div>
    </div>

  </div>

  <!-- Footer -->
  <div style="text-align:center; margin-top:48px; color:var(--text3); font-size:11px; font-family:var(--mono);">
    FinVault · Kubernetes Architecture · namespace: finvault · 3 services · 1 database · HPA + Ingress + PVC
  </div>

</div>
</body>
</html>
