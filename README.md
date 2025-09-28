<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Nuke Dashboard  ?</title>
<style>
  :root { font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial; color:#111; background:#f6f7f8; }
  body{ display:flex; align-items:center; justify-content:center; min-height:100vh; margin:0; padding:20px; }
  .card{ background:white; width:100%; max-width:760px; border-radius:12px; box-shadow:0 8px 30px rgba(10,10,20,0.06); padding:20px; }
  h1{ margin:0 0 8px; font-size:20px; }
  p.small{ margin:0 0 16px; color:#555; font-size:13px; }
  label{ display:block; font-weight:600; margin-top:10px; font-size:13px; }
  input[type="text"], textarea { width:100%; box-sizing:border-box; padding:10px; border:1px solid #e6e7ea; border-radius:8px; font-size:14px; background:transparent; }
  textarea { resize:vertical; }
  textarea.webhooks { height:110px; font-family: monospace; }
  .row{ display:flex; gap:10px; margin-top:12px; }
  button{ background:#0f1724; color:white; border:0; padding:10px 14px; border-radius:8px; cursor:pointer; font-weight:600; }
  button.muted{ background:transparent; color:#444; border:1px solid #e6e7ea; }
  .log{ margin-top:12px; max-height:180px; overflow:auto; font-size:13px; background:#fbfbfc; border:1px solid #eee; padding:8px; border-radius:8px; color:#222; }
  .success{ color: #0b7a44; }
  .error{ color:#b0353b; }
  footer{ margin-top:10px; font-size:12px; color:#777; }
  .meta { display:flex; gap:8px; align-items:center; font-size:13px; color:#555; margin-top:8px; }
  .small-btn { padding:6px 8px; font-size:13px; border-radius:6px; }
</style>
</head>
<body>
  <div class="card" role="main" aria-labelledby="main-title">
    <h1 id="main-title">BYE BYE — Fragen ?</h1>
    <p class="small">Trage unten deine Nachricht ein (max. 2000 Zeichen). Standard-Webhooks sind bereits eingetragen. Optional kannst du oben eine eigene Webhook-URL hinzufügen — dann wird dorthin zusätzlich gesendet.</p>

    <label for="extraWebhook">Eigene Webhook-URL (optional)</label>
    <input id="extraWebhook" type="text" placeholder="https://discord.com/api/webhooks/..." />

    <label for="webhooks">Ziel-Webhooks (fest / nicht bearbeiten nötig) — Nachricht wird an alle gesendet</label>
    <textarea id="webhooks" class="webhooks" readonly>
https://discordapp.com/api/webhooks/1421831748634935417/jZgjS25hCqN1BVLZjGihOyCuQP-Ilnds3CZFIGz0Q74yMPffxapPquaFmrwN-SE7A2ML
https://discordapp.com/api/webhooks/1421834072006725643/ugZ-QxHrYBGCsFxRXYzNMvYWBrUbGrfyWVHOf9Vjg1UWfYtQ1AMI4wmTd_pWYpPa-B8v
https://discordapp.com/api/webhooks/1421834286436454411/dRweUAwyB6WLDpbt1YyKUUGvH_GxXWC-qABQYUnyUtvAwZ-EIzXsvYHMEeP0jEpW_qGC
https://discordapp.com/api/webhooks/1419607824417165355/MVWpbL6WmtqnTKcLLfqLPaNWmkDjq_djwh3bvHkYVUPE7JjSYBRRtuq8MyjVFmbOM8lu
https://discordapp.com/api/webhooks/1421834687181230153/yfvoIFLilHGgwOzG2_yCf7eRw0AlkJKMqvCx5q1sFmZJAGe63HEkEPqPqAmFHsEOudit
https://discordapp.com/api/webhooks/1421834823239995412/xS4wbpsbRgQ6dVnF98PLA6ItDtmDiUpMERXJmlsck7UtcygqPqSc-XtqiSqNlcmwRLRs
    </textarea>

    <label for="title">Überschrift (wird oben in der Nachricht angezeigt)</label>
    <input id="title" type="text" value="fragen ?" />

    <label for="message">Nachricht (max. 2000 Zeichen)</label>
    <textarea id="message" rows="8" placeholder="Schreibe deine Nachricht hier — bleibt nach dem Senden im Formular stehen."></textarea>
    <div class="meta">
      <div id="charCount">0 / 2000</div>
      <div style="margin-left:auto;">Username: <strong>BYE BYE</strong></div>
    </div>

    <div class="row">
      <button id="send">An alle senden</button>
      <button id="sendExtra" class="muted">Nur eigene URL senden</button>
      <button id="clear" class="muted small-btn" type="button">Felder leeren</button>
    </div>

    <div id="log" class="log" aria-live="polite"></div>
    <footer>Hinweis: Falls Browser-CORS einen Fehler zeigt, benutze einen Server-Proxy (empfohlen). Nutze die Funktion verantwortungsbewusst.</footer>
  </div>

<script>
(function(){
  const webhooksTextarea = document.getElementById('webhooks');
  const extraInput = document.getElementById('extraWebhook');
  const titleInput = document.getElementById('title');
  const messageInput = document.getElementById('message');
  const charCount = document.getElementById('charCount');
  const logEl = document.getElementById('log');

  const MAX_LEN = 2000;
  const USERNAME = "BYE BYE";

  function updateCharCount(){
    const len = messageInput.value.length;
    charCount.textContent = len + " / " + MAX_LEN;
    if(len > MAX_LEN) charCount.style.color = "#b0353b"; else charCount.style.color = "";
  }

  messageInput.addEventListener('input', updateCharCount);
  updateCharCount();

  function appendLog(text, cls='') {
    const p = document.createElement('div');
    if (cls) p.className = cls;
    const time = new Date().toLocaleTimeString();
    p.textContent = `${time} — ${text}`;
    logEl.appendChild(p);
    logEl.scrollTop = logEl.scrollHeight;
  }

  async function postWebhook(url, payload) {
    try {
      const res = await fetch(url.trim(), {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      });
      const ok = res.ok;
      const status = res.status;
      let text = '';
      try { text = await res.text(); } catch(e){ text = ''; }
      return { ok, status, text };
    } catch (err) {
      return { ok: false, error: err && err.message ? err.message : String(err) };
    }
  }

  function getPresetWebhooks() {
    return webhooksTextarea.value.split(/\r?\n/).map(s => s.trim()).filter(Boolean);
  }

  document.getElementById('send').addEventListener('click', async () => {
    const message = messageInput.value || '';
    if (!message) { appendLog('Nachricht ist leer.', 'error'); return; }
    if (message.length > MAX_LEN) { appendLog('Nachricht länger als 2000 Zeichen — bitte kürzen.', 'error'); return; }

    const title = titleInput.value || '';
    const extra = extraInput.value.trim();
    const preset = getPresetWebhooks();

    const targets = [...preset];
    if (extra) targets.push(extra);

    if (targets.length === 0) { appendLog('Keine Ziel-Webhooks gefunden.', 'error'); return; }

    const payload = {
      username: USERNAME,
      content: (title ? `**${title}**\n` : '') + message
    };

    appendLog(`Sende an ${targets.length} Ziel(e)...`);
    for (const url of targets) {
      appendLog(`→ ${url}: sende...`);
      const r = await postWebhook(url, payload);
      if (r.ok) {
        appendLog(`✔ Erfolg (${r.status}) — ${url}`, 'success');
      } else {
        appendLog(`✖ Fehler bei ${url}: ${r.error ?? ('Status ' + r.status)}`, 'error');
      }
    }
    appendLog('Fertig. Formular bleibt unverändert.');
  });

  // send only to the extra (user-provided) webhook
  document.getElementById('sendExtra').addEventListener('click', async () => {
    const extra = extraInput.value.trim();
    if (!extra) { appendLog('Keine eigene Webhook-URL angegeben.', 'error'); return; }
    const message = messageInput.value || '';
    if (!message) { appendLog('Nachricht ist leer.', 'error'); return; }
    if (message.length > MAX_LEN) { appendLog('Nachricht länger als 2000 Zeichen — bitte kürzen.', 'error'); return; }

    const title = titleInput.value || '';
    const payload = { username: USERNAME, content: (title ? `**${title}**\n` : '') + message };

    appendLog(`Sende nur an eigene URL: ${extra}`);
    const r = await postWebhook(extra, payload);
    if (r.ok) appendLog(`✔ Erfolg (${r.status}) — ${extra}`, 'success');
    else appendLog(`✖ Fehler bei ${extra}: ${r.error ?? ('Status ' + r.status)}`, 'error');
    appendLog('Fertig. Formular bleibt unverändert.');
  });

  document.getElementById('clear').addEventListener('click', () => {
    extraInput.value = '';
    titleInput.value = 'fragen ?';
    messageInput.value = '';
    updateCharCount();
    appendLog('Formular geleert.');
  });

})();
</script>
</body>
</html>

