<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>USB Key Manager</title>
<style>
  /* Offline-safe fonts — no external requests */

  :root {
    --g: #00ff88;
    --g2: #00cc66;
    --gb: #003322;
    --bg: #070d0a;
    --bg2: #0c1510;
    --bg3: #111a14;
    --border: #1a3a22;
    --dim: #3a6a4a;
    --red: #ff4444;
    --blue: #00ccff;
    --text: #c8ffc8;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    font-family: 'Share Tech Mono', 'Lucida Console', 'Courier New', monospace;
    color: var(--text);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding: 30px 16px 50px;
  }

  /* Scanlines */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,255,100,0.012) 2px, rgba(0,255,100,0.012) 4px);
    pointer-events: none; z-index: 9999;
  }

  /* Glow bg */
  body::after {
    content: '';
    position: fixed; inset: 0;
    background:
      radial-gradient(ellipse 50% 40% at 20% 15%, rgba(0,80,40,0.25) 0%, transparent 70%),
      radial-gradient(ellipse 40% 50% at 80% 85%, rgba(0,40,80,0.2) 0%, transparent 70%);
    pointer-events: none; z-index: 0;
  }

  .wrap { position: relative; z-index: 1; width: 100%; max-width: 520px; }

  /* HEADER */
  .hdr {
    text-align: center;
    margin-bottom: 28px;
  }
  .hdr-icon {
    font-size: 40px;
    display: block;
    margin-bottom: 10px;
    filter: drop-shadow(0 0 14px var(--g));
    animation: pulse 3s ease-in-out infinite;
  }
  @keyframes pulse {
    0%,100% { filter: drop-shadow(0 0 14px var(--g)); }
    50%      { filter: drop-shadow(0 0 28px var(--g)); }
  }
  .hdr-title {
    font-family: 'Orbitron', 'Trebuchet MS', 'Arial Narrow', monospace;
    font-size: 22px;
    font-weight: 900;
    letter-spacing: 5px;
    color: var(--g);
    text-shadow: 0 0 30px rgba(0,255,136,0.4);
  }
  .hdr-sub {
    font-size: 10px;
    color: var(--dim);
    letter-spacing: 6px;
    margin-top: 4px;
  }

  /* CARD */
  .card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 28px 30px;
    box-shadow: 0 0 60px rgba(0,255,136,0.04), inset 0 1px 0 #1e3e28;
  }

  /* SCREEN TITLE */
  .screen-title {
    font-family: 'Orbitron', 'Trebuchet MS', 'Arial Narrow', monospace;
    font-size: 11px;
    letter-spacing: 4px;
    color: var(--dim);
    margin-bottom: 18px;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .screen-title span { color: var(--g); }

  /* BUTTONS */
  .btn {
    display: flex; align-items: center; justify-content: center; gap: 8px;
    width: 100%;
    padding: 12px 18px;
    border-radius: 4px;
    font-family: 'Share Tech Mono', 'Lucida Console', 'Courier New', monospace;
    font-size: 13px;
    letter-spacing: 2px;
    cursor: pointer;
    transition: all .15s;
    border: 1px solid;
  }
  .btn:active { transform: translateY(1px); }
  .btn-green {
    background: rgba(0,255,136,0.07);
    border-color: rgba(0,255,136,0.4);
    color: var(--g);
  }
  .btn-green:hover { background: rgba(0,255,136,0.14); box-shadow: 0 0 20px rgba(0,255,136,0.15); }
  .btn-blue {
    background: rgba(0,204,255,0.07);
    border-color: rgba(0,204,255,0.35);
    color: var(--blue);
  }
  .btn-blue:hover { background: rgba(0,204,255,0.12); }
  .btn-red {
    background: rgba(255,68,68,0.06);
    border-color: rgba(255,68,68,0.3);
    color: var(--red);
    font-size: 11px;
    padding: 7px 14px;
    width: auto;
  }
  .btn-ghost {
    background: none;
    border: 1px solid var(--border);
    color: var(--dim);
    font-size: 12px;
  }
  .btn-ghost:hover { border-color: var(--g); color: var(--text); }
  .btn-sm { padding: 8px 14px; font-size: 11px; letter-spacing: 1px; }
  .btn:disabled { opacity: .4; cursor: not-allowed; }
  .btn-row { display: flex; gap: 10px; margin-top: 16px; }
  .btn-row .btn { flex: 1; }

  /* BACK LINK */
  .back {
    background: none; border: none; color: var(--dim);
    cursor: pointer; font-family: inherit; font-size: 11px;
    margin-top: 14px; display: inline-block;
    letter-spacing: 1px;
  }
  .back:hover { color: var(--g); }

  /* INFO BOX */
  .info {
    background: rgba(0,255,136,0.04);
    border: 1px solid rgba(0,255,136,0.15);
    border-left: 3px solid var(--g2);
    border-radius: 3px;
    padding: 12px 14px;
    font-size: 12px;
    color: #7ac07a;
    line-height: 1.8;
    margin-bottom: 22px;
  }

  /* ERROR */
  .err {
    background: rgba(255,40,40,0.07);
    border: 1px solid rgba(255,40,40,0.3);
    color: #ff8888;
    padding: 10px 14px;
    border-radius: 3px;
    font-size: 12px;
    margin-bottom: 16px;
  }

  /* KEY DISPLAY */
  .key-box {
    background: #050f08;
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 14px;
    margin-bottom: 16px;
    position: relative;
  }
  .key-label {
    font-size: 9px; letter-spacing: 3px;
    color: var(--dim); margin-bottom: 6px;
  }
  .key-val {
    font-size: 10px; color: var(--g);
    word-break: break-all; line-height: 1.7;
    text-shadow: 0 0 6px rgba(0,255,136,0.25);
  }
  .key-id {
    font-size: 10px; color: var(--dim);
    margin-top: 8px;
  }

  /* DROP ZONE */
  .dropzone {
    border: 2px dashed var(--border);
    border-radius: 5px;
    padding: 36px 20px;
    text-align: center;
    cursor: pointer;
    transition: all .2s;
    margin-bottom: 16px;
    position: relative;
  }
  .dropzone:hover, .dropzone.over {
    border-color: var(--g);
    background: rgba(0,255,136,0.04);
  }
  .dropzone input[type=file] {
    position: absolute; inset: 0; opacity: 0; cursor: pointer; width: 100%; height: 100%;
  }
  .dz-icon { font-size: 36px; display: block; margin-bottom: 10px; }
  .dz-text { font-size: 13px; color: var(--dim); }
  .dz-hint { font-size: 11px; color: #2a4a2a; margin-top: 5px; }

  /* VAULT LIST */
  .vault-header {
    display: flex; justify-content: space-between;
    align-items: center; margin-bottom: 16px;
  }
  .vault-count {
    font-size: 11px; color: var(--dim); letter-spacing: 2px;
  }
  .vault-count span { color: var(--g); }

  .accounts { max-height: 340px; overflow-y: auto; margin-bottom: 14px; }
  .accounts::-webkit-scrollbar { width: 4px; }
  .accounts::-webkit-scrollbar-track { background: var(--bg); }
  .accounts::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  .acc-item {
    background: #050f08;
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 12px 14px;
    margin-bottom: 8px;
    transition: border-color .15s;
    position: relative;
  }
  .acc-item:hover { border-color: #2a5a32; }
  .acc-name {
    font-family: 'Orbitron', 'Trebuchet MS', 'Arial Narrow', monospace;
    font-size: 11px; font-weight: 700;
    color: var(--g); margin-bottom: 3px;
  }
  .acc-url { font-size: 10px; color: #2a5a32; margin-bottom: 8px; }
  .acc-row {
    display: flex; align-items: center; gap: 8px; margin-bottom: 4px;
  }
  .acc-label { font-size: 9px; color: var(--dim); letter-spacing: 2px; min-width: 60px; }
  .acc-val { font-size: 12px; color: #9adaaa; flex: 1; }
  .acc-val.hidden { letter-spacing: 2px; color: var(--dim); }
  .icon-btn {
    background: none; border: none; cursor: pointer;
    color: var(--dim); padding: 2px 5px;
    font-size: 13px; line-height: 1;
    transition: color .1s;
  }
  .icon-btn:hover { color: var(--g); }
  .icon-btn.del:hover { color: var(--red); }
  .acc-actions {
    position: absolute; top: 10px; right: 10px;
    display: flex; gap: 4px;
  }
  .copied { color: var(--g) !important; }

  /* FORM */
  .form-group { margin-bottom: 16px; }
  .form-label {
    display: block; font-size: 10px;
    letter-spacing: 3px; color: var(--dim);
    margin-bottom: 6px;
  }
  .form-input {
    width: 100%;
    background: #050f08;
    border: 1px solid var(--border);
    border-radius: 3px;
    color: var(--text);
    padding: 10px 12px;
    font-family: 'Share Tech Mono', 'Lucida Console', 'Courier New', monospace;
    font-size: 13px;
    outline: none;
    transition: border-color .15s;
  }
  .form-input:focus { border-color: var(--g2); }
  .form-input::placeholder { color: #2a4a2a; }

  /* EMPTY STATE */
  .empty {
    text-align: center; padding: 24px;
    color: #2a4a2a; font-size: 12px; line-height: 2;
  }

  /* LOADER */
  .loader {
    text-align: center; color: var(--g);
    font-size: 13px; letter-spacing: 3px;
    padding: 10px;
    animation: blink 1s step-end infinite;
  }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.3} }

  /* FOOTER */
  .footer {
    margin-top: 20px; text-align: center;
    font-size: 9px; color: #1a3a22;
    letter-spacing: 4px; line-height: 2;
  }

  /* PASSWORD GEN */
  .pw-gen {
    display: flex; gap: 8px; margin-top: 6px;
  }
  .pw-gen .form-input { flex: 1; }

  /* STATUS BAR */
  .status-bar {
    display: flex; align-items: center; gap: 8px;
    margin-bottom: 18px; padding: 8px 12px;
    background: rgba(0,255,136,0.04);
    border: 1px solid rgba(0,255,136,0.12);
    border-radius: 3px; font-size: 11px;
  }
  .status-dot {
    width: 7px; height: 7px; border-radius: 50%;
    background: var(--g);
    box-shadow: 0 0 8px var(--g);
    animation: pulse-dot 2s ease-in-out infinite;
  }
  @keyframes pulse-dot { 0%,100%{opacity:1} 50%{opacity:.3} }
  .status-bar span { color: var(--dim); }
  .status-bar strong { color: var(--g); margin-left: auto; }

  .tag {
    display: inline-block; font-size: 9px;
    letter-spacing: 2px; padding: 2px 7px;
    border-radius: 2px; margin-left: 8px;
    background: rgba(0,255,136,0.1);
    border: 1px solid rgba(0,255,136,0.2);
    color: var(--g2);
    vertical-align: middle;
  }
</style>
</head>
<body>
<div class="wrap">

  <!-- HEADER -->
  <div class="hdr">
    <span class="hdr-icon">🔐</span>
    <div class="hdr-title">USB KEY MANAGER</div>
    <div class="hdr-sub">CIFRADO AES-256-GCM · 100% LOCAL</div>
  </div>

  <!-- SCREENS container -->
  <div id="app"></div>

  <div class="footer">
    NINGÚN DATO SALE DE TU DISPOSITIVO<br/>
    ABRE ESTE ARCHIVO DESDE TU USB PARA MAYOR SEGURIDAD
  </div>
</div>

<script>
// ============================================================
//  CRYPTO ENGINE
// ============================================================
const Crypto = {
  async generateKey() {
    const key = await crypto.subtle.generateKey({ name:'AES-GCM', length:256 }, true, ['encrypt','decrypt']);
    const raw = await crypto.subtle.exportKey('raw', key);
    return Array.from(new Uint8Array(raw)).map(b=>b.toString(16).padStart(2,'0')).join('');
  },
  async importKey(hex) {
    const bytes = new Uint8Array(hex.match(/.{1,2}/g).map(b=>parseInt(b,16)));
    return crypto.subtle.importKey('raw', bytes, {name:'AES-GCM'}, false, ['encrypt','decrypt']);
  },
  async encrypt(data, hexKey) {
    const key = await this.importKey(hexKey);
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const enc = await crypto.subtle.encrypt({name:'AES-GCM',iv}, key, new TextEncoder().encode(JSON.stringify(data)));
    return {
      iv: [...iv].map(b=>b.toString(16).padStart(2,'0')).join(''),
      data: [...new Uint8Array(enc)].map(b=>b.toString(16).padStart(2,'0')).join(''),
      v: 1
    };
  },
  async decrypt(blob, hexKey) {
    const key = await this.importKey(hexKey);
    const iv = new Uint8Array(blob.iv.match(/.{1,2}/g).map(b=>parseInt(b,16)));
    const data = new Uint8Array(blob.data.match(/.{1,2}/g).map(b=>parseInt(b,16)));
    const dec = await crypto.subtle.decrypt({name:'AES-GCM',iv}, key, data);
    return JSON.parse(new TextDecoder().decode(dec));
  }
};

// ============================================================
//  STATE
// ============================================================
const State = {
  screen: 'home',
  masterKey: null,
  keyId: null,
  accounts: [],
  loading: false,
  error: '',
  newKey: null,
  showPw: {},
  editId: null,
  form: { name:'', url:'', username:'', password:'' }
};

const VAULT_KEY = 'usb_vault_v1';

// ============================================================
//  UTILS
// ============================================================
function uid() { return Date.now().toString(36) + Math.random().toString(36).slice(2); }
function shortId(hex) { return hex.slice(0,8).toUpperCase() + '…' + hex.slice(-8).toUpperCase(); }

function generatePassword(len=16) {
  const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()_+-=';
  const arr = crypto.getRandomValues(new Uint8Array(len));
  return Array.from(arr).map(b=>chars[b%chars.length]).join('');
}

function downloadFile(name, content, type='application/json') {
  const blob = new Blob([content], {type});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = name;
  a.click();
  setTimeout(()=>URL.revokeObjectURL(a.href), 1000);
}

function copyText(text, btnEl) {
  navigator.clipboard.writeText(text).then(()=>{
    const orig = btnEl.textContent;
    btnEl.textContent = '✓';
    btnEl.classList.add('copied');
    setTimeout(()=>{ btnEl.textContent=orig; btnEl.classList.remove('copied'); }, 1500);
  });
}

function getStorage() {
  try { return JSON.parse(localStorage.getItem(VAULT_KEY) || 'null'); } catch { return null; }
}
function setStorage(val) {
  try { localStorage.setItem(VAULT_KEY, JSON.stringify(val)); } catch {}
}

// ============================================================
//  VAULT OPS
// ============================================================
async function loadVault(hexKey) {
  State.loading = true; State.error = ''; render();
  try {
    const stored = getStorage();
    if (!stored) {
      State.masterKey = hexKey;
      State.keyId = shortId(hexKey);
      State.accounts = [];
      State.screen = 'vault';
    } else {
      const data = await Crypto.decrypt(stored, hexKey);
      State.masterKey = hexKey;
      State.keyId = shortId(hexKey);
      State.accounts = data;
      State.screen = 'vault';
    }
  } catch {
    State.error = '❌ Llave incorrecta o archivo dañado. Intenta de nuevo.';
  }
  State.loading = false; render();
}

async function saveVault() {
  const enc = await Crypto.encrypt(State.accounts, State.masterKey);
  setStorage(enc);
}

// ============================================================
//  RENDER ENGINE
// ============================================================
const app = document.getElementById('app');

function render() {
  const s = State;
  let html = '';

  if (s.error) html += `<div class="err">${s.error}</div>`;
  if (s.loading) html += `<div class="card"><div class="loader">PROCESANDO...</div></div>`;

  if (!s.loading) {
    switch(s.screen) {
      case 'home': html += renderHome(); break;
      case 'create': html += renderCreate(); break;
      case 'unlock': html += renderUnlock(); break;
      case 'vault': html += renderVault(); break;
      case 'add': html += renderAdd(); break;
    }
  }

  app.innerHTML = html;
  bindEvents();
}

// ---- HOME ----
function renderHome() {
  return `
  <div class="card">
    <div class="screen-title">⬡ INICIO</div>
    <div class="info">
      Convierte tu USB en una llave física de acceso.<br/>
      Genera una <strong style="color:var(--g)">llave AES-256</strong> única, guárdala en tu USB<br/>
      y úsala para cifrar todas tus credenciales.<br/>
      Sin la llave = datos ilegibles.
    </div>
    <div style="display:flex;flex-direction:column;gap:10px">
      <button class="btn btn-green" data-action="go-create">
        ⚡ &nbsp;GENERAR NUEVA LLAVE USB
      </button>
      <button class="btn btn-blue" data-action="go-unlock">
        🔓 &nbsp;USAR LLAVE EXISTENTE
      </button>
    </div>
  </div>`;
}

// ---- CREATE ----
function renderCreate() {
  const k = State.newKey;
  return `
  <div class="card">
    <div class="screen-title">⚡ <span>CREAR LLAVE</span></div>
    <div class="info">
      Paso 1: Genera tu llave maestra única.<br/>
      Paso 2: Descarga el archivo <strong style="color:var(--g)">llave-usb.json</strong><br/>
      Paso 3: Cópialo a tu USB física. ¡Nunca pierdas ese archivo!
    </div>
    ${!k ? `
      <button class="btn btn-green" data-action="gen-key">
        ⚡ GENERAR LLAVE MAESTRA
      </button>
    ` : `
      <div class="key-box">
        <div class="key-label">🔑 LLAVE GENERADA — AES-256</div>
        <div class="key-val">${k.slice(0,32)}<br/>${k.slice(32,64)}</div>
        <div class="key-id" style="margin-top:10px;font-size:9px;">
          ID: <span style="color:var(--g)">${shortId(k)}</span>
          <span class="tag">NUEVA</span>
        </div>
      </div>
      <div style="display:flex;flex-direction:column;gap:10px">
        <button class="btn btn-green" data-action="download-key">
          💾 &nbsp;DESCARGAR llave-usb.json (para tu USB)
        </button>
        <button class="btn btn-blue" data-action="use-new-key">
          🔓 &nbsp;ABRIR VAULT CON ESTA LLAVE
        </button>
      </div>
    `}
    <button class="back" data-action="go-home">← volver al inicio</button>
  </div>`;
}

// ---- UNLOCK ----
function renderUnlock() {
  return `
  <div class="card">
    <div class="screen-title">🔓 <span>DESBLOQUEAR VAULT</span></div>
    <div class="info">
      Conecta tu USB y carga el archivo<br/>
      <strong style="color:var(--g)">llave-usb.json</strong> para acceder a tus cuentas.
    </div>
    <div class="dropzone" id="dz">
      <input type="file" accept=".json" id="key-file-input"/>
      <span class="dz-icon">📁</span>
      <div class="dz-text">Clic aquí o arrastra tu llave</div>
      <div class="dz-hint">llave-usb.json</div>
    </div>
    <div style="text-align:center;color:var(--dim);font-size:11px;margin-bottom:12px">— o pega la llave manualmente —</div>
    <div class="form-group">
      <label class="form-label">PEGAR LLAVE HEX (64 caracteres)</label>
      <input class="form-input" id="manual-key" placeholder="0123abcd..." maxlength="64"/>
    </div>
    <button class="btn btn-blue btn-sm" data-action="unlock-manual" style="width:auto;padding:9px 20px;">
      🔓 DESBLOQUEAR
    </button>
    <button class="back" data-action="go-home">← volver al inicio</button>
  </div>`;
}

// ---- VAULT ----
function renderVault() {
  const accs = State.accounts;
  return `
  <div class="card">
    <div class="status-bar">
      <div class="status-dot"></div>
      <span>VAULT ACTIVO</span>
      <strong>ID: ${State.keyId}</strong>
      <button class="btn btn-red btn-sm" data-action="lock" style="margin-left:8px">🔒 CERRAR</button>
    </div>

    <div class="vault-header">
      <div class="vault-count">CUENTAS: <span>${accs.length}</span></div>
      <button class="btn btn-green btn-sm" data-action="go-add" style="width:auto;padding:8px 16px;font-size:11px">
        + AGREGAR
      </button>
    </div>

    <div class="accounts">
      ${accs.length===0 ? `
        <div class="empty">
          🔐 Vault vacío<br/>
          Agrega tu primera cuenta
        </div>
      ` : accs.map(acc=>`
        <div class="acc-item" id="acc-${acc.id}">
          <div class="acc-actions">
            <button class="icon-btn" data-action="copy-user" data-id="${acc.id}" title="Copiar usuario">📋</button>
            <button class="icon-btn" data-action="copy-pw" data-id="${acc.id}" title="Copiar contraseña">🔑</button>
            <button class="icon-btn" data-action="toggle-pw" data-id="${acc.id}" title="Ver contraseña">👁</button>
            <button class="icon-btn del" data-action="delete-acc" data-id="${acc.id}" title="Eliminar">🗑</button>
          </div>
          <div class="acc-name">${esc(acc.name)}</div>
          ${acc.url ? `<div class="acc-url">${esc(acc.url)}</div>` : ''}
          <div class="acc-row">
            <span class="acc-label">USUARIO</span>
            <span class="acc-val">${esc(acc.username)}</span>
          </div>
          <div class="acc-row">
            <span class="acc-label">CLAVE</span>
            <span class="acc-val ${State.showPw[acc.id]?'':'hidden'}">
              ${State.showPw[acc.id] ? esc(acc.password) : '••••••••••'}
            </span>
          </div>
          ${acc.notes ? `<div class="acc-row"><span class="acc-label">NOTAS</span><span class="acc-val" style="font-size:11px;color:var(--dim)">${esc(acc.notes)}</span></div>` : ''}
        </div>
      `).join('')}
    </div>

    <button class="btn btn-ghost btn-sm" data-action="export-vault">📤 Exportar backup cifrado</button>
  </div>`;
}

// ---- ADD ----
function renderAdd() {
  const f = State.form;
  return `
  <div class="card">
    <div class="screen-title">+ <span>NUEVA CUENTA</span></div>
    <div class="form-group">
      <label class="form-label">NOMBRE / SERVICIO *</label>
      <input class="form-input" id="f-name" placeholder="Gmail, Netflix, banco…" value="${esc(f.name)}"/>
    </div>
    <div class="form-group">
      <label class="form-label">URL (opcional)</label>
      <input class="form-input" id="f-url" placeholder="https://…" value="${esc(f.url)}"/>
    </div>
    <div class="form-group">
      <label class="form-label">USUARIO / EMAIL *</label>
      <input class="form-input" id="f-username" placeholder="usuario@correo.com" value="${esc(f.username)}"/>
    </div>
    <div class="form-group">
      <label class="form-label">CONTRASEÑA *</label>
      <div class="pw-gen">
        <input class="form-input" id="f-password" type="text" placeholder="contraseña" value="${esc(f.password)}"/>
        <button class="btn btn-ghost btn-sm" data-action="gen-pw" style="width:auto;white-space:nowrap">⚡ Generar</button>
      </div>
    </div>
    <div class="form-group">
      <label class="form-label">NOTAS (opcional)</label>
      <input class="form-input" id="f-notes" placeholder="PIN, pregunta secreta…" value="${esc(f.notes||'')}"/>
    </div>
    <div class="btn-row">
      <button class="btn btn-green" data-action="save-acc">💾 GUARDAR</button>
      <button class="btn btn-ghost" data-action="go-vault">CANCELAR</button>
    </div>
  </div>`;
}

function esc(s='') {
  return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

// ============================================================
//  EVENT BINDING
// ============================================================
function bindEvents() {
  // Data-action buttons
  document.querySelectorAll('[data-action]').forEach(el=>{
    el.addEventListener('click', handleAction);
  });

  // Dropzone
  const dz = document.getElementById('dz');
  const fi = document.getElementById('key-file-input');
  if (dz && fi) {
    fi.addEventListener('change', e=>{ loadKeyFile(e.target.files[0]); });
    dz.addEventListener('dragover', e=>{ e.preventDefault(); dz.classList.add('over'); });
    dz.addEventListener('dragleave', ()=>dz.classList.remove('over'));
    dz.addEventListener('drop', e=>{
      e.preventDefault(); dz.classList.remove('over');
      loadKeyFile(e.dataTransfer.files[0]);
    });
  }
}

async function handleAction(e) {
  const a = e.currentTarget.dataset.action;
  const id = e.currentTarget.dataset.id;
  State.error = '';

  switch(a) {
    case 'go-home':   State.screen='home'; render(); break;
    case 'go-create': State.screen='create'; State.newKey=null; render(); break;
    case 'go-unlock': State.screen='unlock'; render(); break;
    case 'go-add':    State.form={name:'',url:'',username:'',password:'',notes:''}; State.screen='add'; render(); break;
    case 'go-vault':  State.screen='vault'; render(); break;

    case 'gen-key':
      State.loading=true; render();
      State.newKey = await Crypto.generateKey();
      State.loading=false; render();
      break;

    case 'download-key':
      downloadFile('llave-usb.json', JSON.stringify({
        usbKey: State.newKey,
        id: shortId(State.newKey),
        created: new Date().toISOString(),
        app: 'USB Key Manager v1',
        note: 'Guarda este archivo en tu USB. Sin él no puedes acceder al vault.'
      }, null, 2));
      break;

    case 'use-new-key':
      await loadVault(State.newKey);
      break;

    case 'unlock-manual': {
      const k = (document.getElementById('manual-key')?.value||'').trim();
      if (k.length !== 64) { State.error='La llave debe tener exactamente 64 caracteres hex.'; render(); return; }
      await loadVault(k);
      break;
    }

    case 'lock':
      State.masterKey=null; State.accounts=[]; State.screen='home'; State.showPw={}; render();
      break;

    case 'toggle-pw':
      State.showPw[id] = !State.showPw[id]; render();
      break;

    case 'copy-user': {
      const acc = State.accounts.find(a=>a.id===id);
      if(acc) copyText(acc.username, e.currentTarget);
      break;
    }
    case 'copy-pw': {
      const acc = State.accounts.find(a=>a.id===id);
      if(acc) copyText(acc.password, e.currentTarget);
      break;
    }

    case 'delete-acc': {
      if (!confirm('¿Eliminar esta cuenta? Esta acción no se puede deshacer.')) break;
      State.accounts = State.accounts.filter(a=>a.id!==id);
      await saveVault();
      render();
      break;
    }

    case 'gen-pw': {
      const pw = generatePassword(18);
      const el = document.getElementById('f-password');
      if(el) { el.value=pw; el.focus(); }
      break;
    }

    case 'save-acc': {
      const name     = document.getElementById('f-name')?.value?.trim()||'';
      const url      = document.getElementById('f-url')?.value?.trim()||'';
      const username = document.getElementById('f-username')?.value?.trim()||'';
      const password = document.getElementById('f-password')?.value||'';
      const notes    = document.getElementById('f-notes')?.value?.trim()||'';
      if(!name||!username||!password) {
        State.error='⚠ Nombre, usuario y contraseña son requeridos.'; render(); return;
      }
      State.loading=true; render();
      State.accounts.push({ id:uid(), name, url, username, password, notes, added:new Date().toISOString() });
      await saveVault();
      State.loading=false;
      State.screen='vault'; render();
      break;
    }

    case 'export-vault': {
      const stored = getStorage();
      if(!stored) { alert('No hay datos para exportar.'); return; }
      downloadFile(`vault-backup-${Date.now()}.json`, JSON.stringify(stored, null, 2));
      break;
    }
  }
}

async function loadKeyFile(file) {
  if(!file) return;
  try {
    const text = await file.text();
    const obj = JSON.parse(text);
    if(!obj.usbKey || obj.usbKey.length!==64) throw new Error('invalid');
    await loadVault(obj.usbKey);
  } catch {
    State.error = '⚠ Archivo inválido. Asegúrate de usar el archivo llave-usb.json correcto.';
    render();
  }
}

// ============================================================
//  BOOT
// ============================================================
render();
</script>
</body>
</html>
