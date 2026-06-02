const STORAGE_KEY      = "lub-records";
const PENDING_KEY      = "lub-pending-sync";
const ORDER_SEQ_KEY    = "lub-order-seq";

const SB_URL = window.SUPABASE_URL || "https://azhpxhrwhegfysoeqmft.supabase.co";
const SB_KEY = window.SUPABASE_ANON_KEY || "";
const SB_HEADERS = {
  "Content-Type":  "application/json",
  "apikey":        SB_KEY,
  "Authorization": "Bearer " + SB_KEY,
  "Prefer":        "return=minimal"
};

// DOM
const form             = document.getElementById("lub-form");
const connectionStatus = document.getElementById("connection-status");
const dbSyncStatus     = document.getElementById("db-sync-status");
const recentList       = document.getElementById("recent-list");
const lubDateTime      = document.getElementById("lubDateTime");
const hourmeterAtual   = document.getElementById("hourmeterAtual");
const hourmeterProx    = document.getElementById("hourmeterProx");
const intervaloPreview = document.getElementById("intervalo-preview");
const obsField         = document.getElementById("observation");
const obsLabel         = document.getElementById("obs-required-label");
const labelProxTroca   = document.getElementById("label-prox-troca");
const sectionFiltro    = document.getElementById("section-filtro");

let tipoServico = "preventiva";

// ── Tipo de serviço ───────────────────────────────────────────
const tipoDesc = {
  preventiva:      "Troca normal de oleo e/ou filtro conforme plano de manutencao.",
  corretiva:       "Corretiva: completar oleo por vazamento ou falha. Observacao obrigatoria.",
  completar_nivel: "Completar nivel de oleo. Observacao obrigatoria."
};

["preventiva","corretiva","completar"].forEach(t => {
  document.getElementById(`tipo-${t}`).addEventListener("click", () => {
    const val = t === "completar" ? "completar_nivel" : t;
    setTipo(val);
  });
});

function setTipo(val) {
  tipoServico = val;
  ["preventiva","corretiva","completar"].forEach(t => {
    document.getElementById(`tipo-${t}`).classList.remove("active");
  });
  const btnId = val === "completar_nivel" ? "tipo-completar" : `tipo-${val}`;
  document.getElementById(btnId).classList.add("active");
  document.getElementById("tipo-desc").textContent = tipoDesc[val];

  const needsObs = val === "corretiva" || val === "completar_nivel";
  obsField.required = needsObs;
  obsLabel.textContent = needsObs ? "(obrigatoria)" : "(opcional)";

  // Proxima troca só aparece na preventiva
  labelProxTroca.style.display = val === "preventiva" ? "" : "none";
  if (val !== "preventiva") hourmeterProx.value = "";

  // Filtro só aparece na preventiva
  sectionFiltro.style.display = val === "preventiva" ? "" : "none";
}

// ── Calculo intervalo ─────────────────────────────────────────
function calcIntervalo() {
  const atual = parseFloat(hourmeterAtual.value);
  const prox  = parseFloat(hourmeterProx.value);
  if (!isNaN(atual) && !isNaN(prox) && prox > atual) {
    intervaloPreview.textContent = `Intervalo de troca: ${(prox - atual).toFixed(0)} horas`;
    intervaloPreview.className = "connection-status online";
  } else if (!isNaN(atual) && !isNaN(prox) && prox <= atual) {
    intervaloPreview.textContent = "Horimetro da proxima troca deve ser maior que o atual.";
    intervaloPreview.className = "connection-status offline";
  } else {
    intervaloPreview.textContent = "";
  }
}
hourmeterAtual.addEventListener("input", calcIntervalo);
hourmeterProx.addEventListener("input",  calcIntervalo);

// ── Carregar insumos do Supabase ──────────────────────────────
async function loadInsumos() {
  try {
    const r = await fetch(
      SB_URL + "/rest/v1/dim_insumo?select=id_insumo,nome,categoria&order=categoria,nome",
      { headers: { "apikey": SB_KEY, "Authorization": "Bearer " + SB_KEY } }
    );
    if (!r.ok) return;
    const data = await r.json();
    const oleos   = data.filter(i => i.categoria === "LUBRIFICANTE");
    const filtros = data.filter(i => i.categoria === "FILTRO");

    ["oilType1","oilType2","oilType3"].forEach(id => {
      const sel = document.getElementById(id);
      oleos.forEach(o => {
        const opt = document.createElement("option");
        opt.value = o.nome;
        opt.textContent = o.nome;
        sel.appendChild(opt);
      });
    });

    ["filterType1","filterType2"].forEach(id => {
      const sel = document.getElementById(id);
      filtros.forEach(f => {
        const opt = document.createElement("option");
        opt.value = f.nome;
        opt.textContent = f.nome;
        sel.appendChild(opt);
      });
    });
  } catch(e) {
    console.warn("Nao foi possivel carregar insumos:", e);
  }
}

// ── Helpers ───────────────────────────────────────────────────
function makeId() {
  return typeof crypto !== "undefined" && crypto.randomUUID
    ? crypto.randomUUID()
    : `${Date.now()}-${Math.floor(Math.random()*1e6)}`;
}

function getNow() {
  const now = new Date();
  return new Date(now.getTime() - now.getTimezoneOffset()*60000).toISOString().slice(0,16);
}

function toIso(val) {
  if (!val) return new Date().toISOString();
  const d = new Date(val);
  return isNaN(d.getTime()) ? new Date().toISOString() : d.toISOString();
}

function nextOrder() {
  const n = Number(localStorage.getItem(ORDER_SEQ_KEY) || "0") + 1;
  localStorage.setItem(ORDER_SEQ_KEY, String(n));
  return `LUB-${String(n).padStart(5,"0")}`;
}

function getRecords() {
  try { return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]"); } catch { return []; }
}
function saveRecords(r) { localStorage.setItem(STORAGE_KEY, JSON.stringify(r)); }
function getPending()   { try { return JSON.parse(localStorage.getItem(PENDING_KEY) || "[]"); } catch { return []; } }
function savePending(q) { localStorage.setItem(PENDING_KEY, JSON.stringify(q)); }

function escapeHtml(t) { const d = document.createElement("div"); d.textContent = t; return d.innerHTML; }

// ── Status ────────────────────────────────────────────────────
function updateConnectionStatus() {
  connectionStatus.textContent = navigator.onLine ? "Online" : "Offline - dados salvos localmente";
  connectionStatus.className = "connection-status " + (navigator.onLine ? "online" : "offline");
}

function updateSyncStatus() {
  const n = getPending().length;
  if (n === 0) {
    dbSyncStatus.textContent = "Sincronizacao com banco em dia.";
    dbSyncStatus.className = "connection-status online";
  } else {
    dbSyncStatus.textContent = `${n} lancamento(s) aguardando envio ao banco.`;
    dbSyncStatus.className = "connection-status offline";
  }
}

// ── Sync Supabase ─────────────────────────────────────────────
async function syncToSupabase(record) {
  try {
    const r = await fetch(SB_URL + "/rest/v1/lubrificacao_v2", {
      method: "POST",
      headers: SB_HEADERS,
      body: JSON.stringify({
        order_number:    record.orderNumber,
        created_at:      record.createdAt,
        vehicle:         record.vehicle         || null,
        operator:        record.operator        || null,
        location:        record.location        || null,
        tipo_servico:    record.tipoServico     || null,
        hourmeter_atual: record.hourmeterAtual  || null,
        hourmeter_prox:  record.hourmeterProx   || null,
        oil_type_1:      record.oilType1        || null,
        oil_qty_1:       record.oilQty1         || null,
        oil_type_2:      record.oilType2        || null,
        oil_qty_2:       record.oilQty2         || null,
        oil_type_3:      record.oilType3        || null,
        oil_qty_3:       record.oilQty3         || null,
        filter_type_1:   record.filterType1     || null,
        filter_qty_1:    record.filterQty1      || null,
        filter_type_2:   record.filterType2     || null,
        filter_qty_2:    record.filterQty2      || null,
        observation:     record.observation     || null
      })
    });
    return r.ok || r.status === 201;
  } catch(e) {
    console.error("Sync error:", e);
    return false;
  }
}

async function processQueue() {
  if (!navigator.onLine) { updateSyncStatus(); return; }
  let queue = getPending();
  while (queue.length) {
    const ok = await syncToSupabase(queue[0]);
    if (!ok) break;
    queue = queue.slice(1);
    savePending(queue);
  }
  updateSyncStatus();
}

function enqueue(record) {
  const q = getPending();
  q.push(record);
  savePending(q);
  updateSyncStatus();
  processQueue();
}

// ── Render recentes ───────────────────────────────────────────
function renderRecent() {
  const records = getRecords().slice(-5).reverse();
  recentList.innerHTML = "";
  if (!records.length) {
    recentList.innerHTML = "<li class='recent-item recent-empty'><span class='recent-cell'>Nenhum servico ainda.</span></li>";
    return;
  }
  records.forEach(r => {
    const li = document.createElement("li");
    li.className = "recent-item";
    li.setAttribute("role","row");
    const data = new Date(r.createdAt).toLocaleDateString("pt-BR");
    li.innerHTML = `
      <span class="recent-cell">${escapeHtml(r.vehicle||"-")}</span>
      <span class="recent-cell">${escapeHtml(r.tipoServico||"-")}</span>
      <span class="recent-cell">${data}</span>
    `;
    recentList.appendChild(li);
  });
}

// ── Submit ────────────────────────────────────────────────────
form.addEventListener("submit", (e) => {
  e.preventDefault();
  const fd = new FormData(form);

  const needsObs = tipoServico === "corretiva" || tipoServico === "completar_nivel";
  const obs = String(fd.get("observation") || "").trim();
  if (needsObs && !obs) {
    alert("Observacao e obrigatoria para este tipo de servico.");
    return;
  }

  const record = {
    id:            makeId(),
    orderNumber:   nextOrder(),
    createdAt:     toIso(String(fd.get("lubDateTime") || "")),
    vehicle:       String(fd.get("vehicle")       || "").trim().toUpperCase(),
    operator:      String(fd.get("operator")      || "").trim().toUpperCase(),
    location:      String(fd.get("location")      || "").trim().toUpperCase(),
    tipoServico:   tipoServico,
    hourmeterAtual: parseFloat(fd.get("hourmeterAtual")) || null,
    hourmeterProx:  parseFloat(fd.get("hourmeterProx"))  || null,
    oilType1:      String(fd.get("oilType1")      || "").trim() || null,
    oilQty1:       parseFloat(fd.get("oilQty1"))  || null,
    oilType2:      String(fd.get("oilType2")      || "").trim() || null,
    oilQty2:       parseFloat(fd.get("oilQty2"))  || null,
    oilType3:      String(fd.get("oilType3")      || "").trim() || null,
    oilQty3:       parseFloat(fd.get("oilQty3"))  || null,
    filterType1:   String(fd.get("filterType1")   || "").trim() || null,
    filterQty1:    parseFloat(fd.get("filterQty1")) || null,
    filterType2:   String(fd.get("filterType2")   || "").trim() || null,
    filterQty2:    parseFloat(fd.get("filterQty2")) || null,
    observation:   obs || null
  };

  const records = getRecords();
  records.push(record);
  saveRecords(records);
  enqueue(record);

  form.reset();
  lubDateTime.value = getNow();
  setTipo("preventiva");
  intervaloPreview.textContent = "";
  renderRecent();
});

// ── Service Worker ────────────────────────────────────────────
if ("serviceWorker" in navigator) {
  window.addEventListener("load", async () => {
    try {
      const reg = await navigator.serviceWorker.register("./sw.js?v=1", { updateViaCache: "none" });
      reg.update();
    } catch(err) {
      console.error("SW error:", err);
    }
  });
}

// ── Init ──────────────────────────────────────────────────────
window.addEventListener("online",  updateConnectionStatus);
window.addEventListener("offline", updateConnectionStatus);
window.addEventListener("online",  processQueue);

lubDateTime.value = getNow();
setTipo("preventiva");
updateConnectionStatus();
updateSyncStatus();
processQueue();
renderRecent();
loadInsumos();
