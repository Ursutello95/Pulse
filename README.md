<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pulse ⚡ — Flujo de ejecución</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@3.19.0/dist/tabler-icons.min.css" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: #0f172a;
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      min-height: 100vh;
      display: flex;
      align-items: flex-start;
      justify-content: center;
      padding: 40px 20px;
      color: #e2e8f0;
    }

    .container {
      max-width: 580px;
      width: 100%;
    }

    .page-title {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 28px;
    }
    .page-title .icon {
      width: 40px; height: 40px;
      border-radius: 12px;
      background: rgba(99,102,241,0.15);
      border: 1px solid rgba(99,102,241,0.3);
      display: flex; align-items: center; justify-content: center;
      font-size: 20px;
    }
    .page-title h1 { font-size: 20px; font-weight: 600; color: #f1f5f9; }
    .page-title p  { font-size: 12px; color: #64748b; margin-top: 2px; }

    /* Steps */
    .flow { display: flex; flex-direction: column; gap: 0; }

    .step { display: flex; gap: 16px; align-items: stretch; opacity: 0; transform: translateY(8px); transition: opacity .4s, transform .4s; }
    .step.visible { opacity: 1; transform: translateY(0); }

    .step-left { display: flex; flex-direction: column; align-items: center; width: 38px; flex-shrink: 0; }

    .step-dot {
      width: 38px; height: 38px; border-radius: 50%;
      border: 2px solid #334155;
      background: #1e293b;
      display: flex; align-items: center; justify-content: center;
      flex-shrink: 0; z-index: 1;
      transition: border-color .4s, background .4s;
    }
    .step-dot i { font-size: 17px; color: #475569; transition: color .4s; }
    .step-dot.active  { border-color: #6366f1; background: rgba(99,102,241,0.12); }
    .step-dot.active i { color: #818cf8; }
    .step-dot.done    { border-color: #22c55e; background: rgba(34,197,94,0.1); }
    .step-dot.done i  { color: #4ade80; }
    .step-dot.fail    { border-color: #f59e0b; background: rgba(245,158,11,0.1); }
    .step-dot.fail i  { color: #fbbf24; }

    .step-line {
      flex: 1; width: 2px;
      background: #1e293b;
      margin: 4px 0;
      min-height: 20px;
      position: relative; overflow: hidden;
    }
    .step-line-fill {
      position: absolute; top: 0; left: 0;
      width: 100%; height: 0%;
      background: #22c55e;
      transition: height .6s cubic-bezier(.4,0,.2,1);
    }

    .step-body { flex: 1; padding-bottom: 18px; }

    .step-card {
      background: #1e293b;
      border: 1px solid #334155;
      border-radius: 12px;
      padding: 12px 16px;
      transition: border-color .4s;
    }
    .step-card.active { border-color: #6366f1; }
    .step-card.done   { border-color: #22c55e; }
    .step-card.fail   { border-color: #f59e0b; }

    .step-head {
      display: flex; align-items: center;
      justify-content: space-between;
      margin-bottom: 4px;
    }
    .step-name { font-size: 13px; font-weight: 600; color: #f1f5f9; }

    .step-status {
      font-size: 11px;
      padding: 2px 9px;
      border-radius: 20px;
      background: #0f172a;
      color: #475569;
      border: 1px solid #334155;
    }
    .step-status.running { background: rgba(99,102,241,0.15); color: #818cf8; border-color: rgba(99,102,241,0.4); }
    .step-status.done    { background: rgba(34,197,94,0.1);   color: #4ade80; border-color: rgba(34,197,94,0.3); }
    .step-status.fail    { background: rgba(245,158,11,0.1);  color: #fbbf24; border-color: rgba(245,158,11,0.3); }

    .step-desc { font-size: 12px; color: #64748b; line-height: 1.5; }

    .step-detail {
      display: none;
      margin-top: 10px;
      padding: 9px 12px;
      background: #0f172a;
      border-radius: 8px;
      font-size: 11.5px;
      color: #94a3b8;
      line-height: 1.8;
      font-family: 'Cascadia Code', 'Fira Code', monospace;
    }
    .step-detail.show { display: block; }

    .tag {
      display: inline-block;
      font-size: 10px;
      padding: 2px 8px;
      border-radius: 20px;
      margin: 2px 3px 2px 0;
      font-family: 'Segoe UI', sans-serif;
    }
    .tag-ok   { background: rgba(34,197,94,0.12);  color: #4ade80; }
    .tag-warn { background: rgba(245,158,11,0.12); color: #fbbf24; }
    .tag-err  { background: rgba(239,68,68,0.12);  color: #f87171; }
    .tag-info { background: rgba(99,102,241,0.15); color: #818cf8; }

    /* Final message preview */
    .final-msg {
      display: none;
      margin-top: 10px;
      background: #0f172a;
      border: 1px solid #1e293b;
      border-radius: 10px;
      padding: 12px 14px;
      font-size: 12px;
      line-height: 1.8;
      color: #cbd5e1;
    }
    .final-msg.show { display: block; }
    .final-msg .msg-header {
      font-size: 13px; font-weight: 600; color: #f1f5f9;
      margin-bottom: 4px;
      display: flex; align-items: center; gap: 6px;
    }
    .final-msg .msg-sub { color: #475569; font-size: 11px; margin-bottom: 10px; }
    .final-msg .msg-footer {
      margin-top: 10px;
      padding-top: 10px;
      border-top: 1px solid #1e293b;
      font-size: 11px;
      color: #475569;
      display: flex; align-items: center; gap: 6px;
    }
    .msg-footer i { color: #4ade80; }

    /* Run button */
    .run-btn {
      margin-top: 20px;
      width: 100%;
      padding: 11px;
      font-size: 13px;
      font-weight: 600;
      border-radius: 10px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      background: rgba(99,102,241,0.15);
      color: #818cf8;
      border: 1px solid rgba(99,102,241,0.35);
      transition: background .15s, border-color .15s;
    }
    .run-btn:hover:not(:disabled) {
      background: rgba(99,102,241,0.25);
      border-color: rgba(99,102,241,0.6);
    }
    .run-btn:disabled { opacity: 0.6; cursor: not-allowed; }
  </style>
</head>
<body>
<div class="container">

  <div class="page-title">
    <div class="icon">⚡</div>
    <div>
      <h1>Pulse</h1>
      <p>Flujo de ejecución · de Qualy a Slack</p>
    </div>
  </div>

  <div class="flow" id="flow">

    <!-- STEP 0 -->
    <div class="step visible" id="step-0">
      <div class="step-left">
        <div class="step-dot" id="dot-0"><i class="ti ti-clock"></i></div>
        <div class="step-line"><div class="step-line-fill" id="line-0"></div></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-0">
          <div class="step-head">
            <span class="step-name">Trigger — Viernes 17hs</span>
            <span class="step-status" id="st-0">En espera</span>
          </div>
          <div class="step-desc">El task programado se activa automáticamente según el cron <code>0 17 * * 5</code></div>
        </div>
      </div>
    </div>

    <!-- STEP 1 -->
    <div class="step visible" id="step-1">
      <div class="step-left">
        <div class="step-dot" id="dot-1"><i class="ti ti-mail"></i></div>
        <div class="step-line"><div class="step-line-fill" id="line-1"></div></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-1">
          <div class="step-head">
            <span class="step-name">Gmail — leer Qualy</span>
            <span class="step-status" id="st-1">Pendiente</span>
          </div>
          <div class="step-desc">Busca el último email de nora.caceres@modo.com.ar con asunto "feedback de calidad"</div>
          <div class="step-detail" id="detail-1">
            from: nora.caceres@modo.com.ar<br>
            subject: "📊 Tu feedback de calidad · 18/05 al 24/05/2026"<br>
            → 20 casos evaluados encontrados
          </div>
        </div>
      </div>
    </div>

    <!-- STEP 2 -->
    <div class="step visible" id="step-2">
      <div class="step-left">
        <div class="step-dot" id="dot-2"><i class="ti ti-file-text"></i></div>
        <div class="step-line"><div class="step-line-fill" id="line-2"></div></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-2">
          <div class="step-head">
            <span class="step-name">Extraer datos del reporte</span>
            <span class="step-status" id="st-2">Pendiente</span>
          </div>
          <div class="step-desc">Identifica fortalezas, oportunidades de mejora con frecuencia y tags HX</div>
          <div class="step-detail" id="detail-2">
            <span class="tag tag-ok">✓ Apertura y tono</span>
            <span class="tag tag-ok">✓ CX Steps</span>
            <span class="tag tag-ok">✓ Guía</span><br>
            <span class="tag tag-warn">⚠ Personalizar explicación — 7/20</span>
            <span class="tag tag-warn">⚠ Evitar pedir datos — 4/20</span>
          </div>
        </div>
      </div>
    </div>

    <!-- STEP 3 -->
    <div class="step visible" id="step-3">
      <div class="step-left">
        <div class="step-dot" id="dot-3"><i class="ti ti-cloud"></i></div>
        <div class="step-line"><div class="step-line-fill" id="line-3"></div></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-3">
          <div class="step-head">
            <span class="step-name">Salesforce — CSAT</span>
            <span class="step-status" id="st-3">Pendiente</span>
          </div>
          <div class="step-desc">Consulta el CSAT de la semana del experto via Zapier → Salesforce</div>
          <div class="step-detail" id="detail-3">
            <span class="tag tag-err">✗ Auth requerida</span><br>
            → Fallback: "pendiente de Salesforce"
          </div>
        </div>
      </div>
    </div>

    <!-- STEP 4 -->
    <div class="step visible" id="step-4">
      <div class="step-left">
        <div class="step-dot" id="dot-4"><i class="ti ti-pencil"></i></div>
        <div class="step-line"><div class="step-line-fill" id="line-4"></div></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-4">
          <div class="step-head">
            <span class="step-name">Redactar mensaje Pulse</span>
            <span class="step-status" id="st-4">Pendiente</span>
          </div>
          <div class="step-desc">Arma el mensaje en formato hamburguesa con voz cálida, argentina y profesional</div>
          <div class="step-detail" id="detail-4">
            <span class="tag tag-ok">🟢 Fortalezas</span>
            <span class="tag tag-warn">🎯 Foco</span>
            <span class="tag tag-ok">💪 Refuerzo</span>
            <span class="tag tag-info">📌 Plan</span>
            <span class="tag tag-info">⭐ CSAT</span>
          </div>
        </div>
      </div>
    </div>

    <!-- STEP 5 -->
    <div class="step visible" id="step-5">
      <div class="step-left">
        <div class="step-dot" id="dot-5"><i class="ti ti-brand-slack"></i></div>
      </div>
      <div class="step-body">
        <div class="step-card" id="card-5">
          <div class="step-head">
            <span class="step-name">Slack — enviar al experto</span>
            <span class="step-status" id="st-5">Pendiente</span>
          </div>
          <div class="step-desc">Publica en <code>#cowork · C08HR97K2Q0</code> · visible solo para Ursula</div>
          <div class="final-msg" id="final-msg">
            <div class="msg-header">⚡ Pulse · Semana 18/05 al 24/05</div>
            <div class="msg-sub">Ursula, cerraste una semana con 20 casos auditados...</div>
            <span class="tag tag-ok">🟢 Apertura</span>
            <span class="tag tag-ok">🟢 CX Steps</span>
            <span class="tag tag-ok">🟢 Guía</span><br>
            <span class="tag tag-warn" style="margin-top:4px;">🎯 Personalizar explicación — 7/20</span><br>
            <span class="tag tag-info" style="margin-top:4px;">📌 Plan de acción incluido</span><br>
            <span class="tag" style="margin-top:4px; background:rgba(100,116,139,0.15); color:#64748b;">⭐ CSAT: pendiente · 📋 20 casos</span>
            <div class="msg-footer">
              <i class="ti ti-check"></i> Entregado · Solo vos podés ver este mensaje 🔒
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>

  <button class="run-btn" id="run-btn" onclick="runFlow()">
    <i class="ti ti-player-play"></i>
    Simular ejecución de Pulse
  </button>

</div>

<script>
  const steps = [
    { delay: 0,    duration: 600,  status: 'Ejecutando…',  doneStatus: 'Activado',    doneClass: 'done', detail: false },
    { delay: 800,  duration: 1000, status: 'Leyendo…',     doneStatus: 'Leído',       doneClass: 'done', detail: true  },
    { delay: 2000, duration: 900,  status: 'Procesando…',  doneStatus: 'Extraído',    doneClass: 'done', detail: true  },
    { delay: 3100, duration: 800,  status: 'Consultando…', doneStatus: 'Fallback ⚠',  doneClass: 'fail', detail: true  },
    { delay: 4100, duration: 1100, status: 'Redactando…',  doneStatus: 'Listo',       doneClass: 'done', detail: true  },
    { delay: 5400, duration: 700,  status: 'Enviando…',    doneStatus: 'Entregado ✓', doneClass: 'done', detail: false, final: true }
  ];

  const icons = ['ti-clock','ti-mail','ti-file-text','ti-cloud','ti-pencil','ti-brand-slack'];
  let running = false;

  function runFlow() {
    if (running) return;
    running = true;
    const btn = document.getElementById('run-btn');
    btn.disabled = true;
    btn.innerHTML = '<i class="ti ti-loader"></i> Ejecutando…';

    steps.forEach((s, i) => {
      setTimeout(() => {
        const dot  = document.getElementById('dot-'  + i);
        const card = document.getElementById('card-' + i);
        const st   = document.getElementById('st-'   + i);
        const line = document.getElementById('line-' + i);

        dot.className  = 'step-dot active';
        card.className = 'step-card active';
        st.className   = 'step-status running';
        st.textContent = s.status;

        setTimeout(() => {
          dot.className  = 'step-dot ' + s.doneClass;
          dot.innerHTML  = s.doneClass === 'fail'
            ? '<i class="ti ti-alert-triangle"></i>'
            : '<i class="ti ti-check"></i>';
          card.className = 'step-card ' + s.doneClass;
          st.className   = 'step-status ' + s.doneClass;
          st.textContent = s.doneStatus;

          if (s.detail) {
            const d = document.getElementById('detail-' + i);
            if (d) d.classList.add('show');
          }
          if (line) line.style.height = '100%';
          if (s.final) {
            document.getElementById('final-msg').classList.add('show');
            btn.innerHTML  = '<i class="ti ti-refresh"></i> Volver a simular';
            btn.disabled   = false;
            btn.onclick    = resetFlow;
            running = false;
          }
        }, s.duration);
      }, s.delay);
    });
  }

  function resetFlow() {
    running = false;
    steps.forEach((s, i) => {
      const dot  = document.getElementById('dot-'  + i);
      const card = document.getElementById('card-' + i);
      const st   = document.getElementById('st-'   + i);
      const line = document.getElementById('line-' + i);
      dot.className  = 'step-dot';
      dot.innerHTML  = '<i class="ti ' + icons[i] + '"></i>';
      card.className = 'step-card';
      st.className   = 'step-status';
      st.textContent = i === 0 ? 'En espera' : 'Pendiente';
      if (line) line.style.height = '0%';
      const d = document.getElementById('detail-' + i);
      if (d) d.classList.remove('show');
    });
    document.getElementById('final-msg').classList.remove('show');
    const btn = document.getElementById('run-btn');
    btn.innerHTML = '<i class="ti ti-player-play"></i> Simular ejecución de Pulse';
    btn.onclick   = runFlow;
  }
</script>
</body>
</html>
