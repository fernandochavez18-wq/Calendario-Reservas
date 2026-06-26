# Calendario-Reservas

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Agenda Tratamientos 2026–2027</title>
<style>
  :root {
    --bg:       #F8F4EF;
    --surface:  #FFFFFF;
    --border:   #E2D9CE;
    --text:     #2A2118;
    --muted:    #8C7B6E;
    --accent:   #D4834A;
    --accent2:  #C46A2E;
    --booked:   #D4834A;
    --booked-bg:#FEF0E6;
    --transport:#E84B3C;
    --transport-bg:#FDE8E6;
    --weekend-bg:#F2EDE7;
    --available:#FFFFFF;
    --header-bg:#2A2118;
    --shadow:   0 2px 12px rgba(42,33,24,.10);
    --radius:   10px;
  }
  * { box-sizing:border-box; margin:0; padding:0; }
  body { font-family:'Segoe UI',system-ui,sans-serif; background:var(--bg); color:var(--text); min-height:100vh; }

  /* ── TOPBAR ── */
  .topbar {
    background:var(--header-bg);
    color:#fff;
    padding:0 24px;
    display:flex; align-items:center; gap:16px;
    height:58px; position:sticky; top:0; z-index:100;
    box-shadow:0 2px 8px rgba(0,0,0,.3);
  }
  .topbar h1 { font-size:17px; font-weight:700; letter-spacing:.3px; flex:1; }
  .topbar h1 span { color:var(--accent); }
  .btn-nav {
    background:rgba(255,255,255,.12); border:none; color:#fff;
    padding:7px 14px; border-radius:6px; cursor:pointer; font-size:13px; font-weight:600;
    transition:background .15s;
  }
  .btn-nav:hover { background:rgba(255,255,255,.22); }
  .btn-nav.active { background:var(--accent); }
  .month-label { font-size:18px; font-weight:700; min-width:180px; text-align:center; letter-spacing:.2px; }

  /* ── LAYOUT ── */
  .container { max-width:1400px; margin:0 auto; padding:20px 16px; }

  /* ── MONTH GRID ── */
  .month-grid { display:grid; grid-template-columns:repeat(7,1fr); gap:4px; }
  .dow-header {
    text-align:center; font-size:11px; font-weight:700; color:var(--muted);
    letter-spacing:.8px; text-transform:uppercase; padding:8px 0;
  }
  .day-cell {
    background:var(--surface); border:1px solid var(--border); border-radius:8px;
    min-height:90px; padding:8px; cursor:pointer; transition:all .15s; position:relative;
  }
  .day-cell:hover { border-color:var(--accent); box-shadow:0 0 0 2px rgba(212,131,74,.15); }
  .day-cell.weekend { background:var(--weekend-bg); }
  .day-cell.other-month { opacity:.35; pointer-events:none; }
  .day-cell.today .day-num { background:var(--accent); color:#fff; }
  .day-num {
    font-size:13px; font-weight:700; width:26px; height:26px;
    border-radius:50%; display:flex; align-items:center; justify-content:center;
    margin-bottom:4px;
  }
  .day-dots { display:flex; flex-wrap:wrap; gap:3px; margin-top:4px; }
  .dot {
    width:8px; height:8px; border-radius:50%; background:var(--accent);
    title:attr(data-title);
  }
  .dot.transport { background:var(--transport); }

  /* ── DAY VIEW ── */
  .day-view { display:none; }
  .day-view.active { display:block; }
  .day-header-bar {
    background:var(--surface); border:1px solid var(--border); border-radius:var(--radius);
    padding:16px 20px; margin-bottom:16px;
    display:flex; align-items:center; justify-content:space-between;
    box-shadow:var(--shadow);
  }
  .day-title { font-size:22px; font-weight:700; }
  .day-subtitle { font-size:13px; color:var(--muted); margin-top:2px; }

  .slots-grid { display:flex; flex-direction:column; gap:3px; }
  .slot {
    display:grid;
    grid-template-columns:72px 1fr;
    border-radius:8px; overflow:hidden;
    border:1px solid var(--border);
    transition:all .12s;
    min-height:44px;
  }
  .slot:hover:not(.transport-slot) { box-shadow:0 0 0 2px rgba(212,131,74,.3); cursor:pointer; }
  .slot-time {
    background:#F5EDE3; display:flex; align-items:center; justify-content:center;
    font-size:13px; font-weight:700; color:var(--muted); border-right:1px solid var(--border);
    padding:0 8px;
  }
  .slot-body { background:var(--available); padding:8px 14px; display:flex; align-items:center; gap:10px; }

  .slot.booked .slot-time { background:#FAEBD7; color:var(--accent2); }
  .slot.booked .slot-body { background:var(--booked-bg); cursor:pointer; }
  .slot.booked:hover { box-shadow:0 0 0 2px rgba(212,131,74,.5); }

  .slot.transport-slot .slot-time { background:#FFEDED; color:#B03030; }
  .slot.transport-slot .slot-body { background:var(--transport-bg); cursor:default; }

  .slot.weekend .slot-time { background:#EDE8E2; }
  .slot.weekend .slot-body { background:var(--weekend-bg); }

  .slot-name { font-size:14px; font-weight:600; color:var(--text); }
  .slot-meta { font-size:11px; color:var(--muted); margin-top:2px; }
  .transport-label { font-size:12px; color:var(--transport); font-weight:600; display:flex; align-items:center; gap:5px; }
  .slot-add { font-size:12px; color:var(--muted); opacity:.6; }
  .slot:hover:not(.transport-slot):not(.booked) .slot-add { opacity:1; color:var(--accent); }

  /* ── MODAL ── */
  .overlay {
    display:none; position:fixed; inset:0; background:rgba(0,0,0,.45);
    z-index:200; align-items:center; justify-content:center;
    backdrop-filter:blur(2px);
  }
  .overlay.open { display:flex; }
  .modal {
    background:var(--surface); border-radius:14px; width:100%; max-width:480px;
    margin:16px; box-shadow:0 20px 60px rgba(0,0,0,.3);
    overflow:hidden;
  }
  .modal-header {
    background:var(--header-bg); color:#fff;
    padding:18px 22px; display:flex; align-items:center; justify-content:space-between;
  }
  .modal-header h3 { font-size:16px; font-weight:700; }
  .modal-header .slot-info { font-size:12px; color:rgba(255,255,255,.6); margin-top:3px; }
  .modal-close { background:none; border:none; color:#fff; font-size:20px; cursor:pointer; opacity:.7; padding:4px 8px; border-radius:4px; }
  .modal-close:hover { opacity:1; background:rgba(255,255,255,.1); }
  .modal-body { padding:22px; display:flex; flex-direction:column; gap:14px; }
  .field { display:flex; flex-direction:column; gap:5px; }
  .field label { font-size:12px; font-weight:700; color:var(--muted); text-transform:uppercase; letter-spacing:.5px; }
  .field input, .field select {
    border:1.5px solid var(--border); border-radius:7px; padding:10px 12px;
    font-size:14px; color:var(--text); background:var(--bg);
    transition:border-color .15s;
    font-family:inherit;
  }
  .field input:focus, .field select:focus { outline:none; border-color:var(--accent); background:#fff; }
  .modal-footer { padding:0 22px 22px; display:flex; gap:10px; }
  .btn-save {
    flex:1; background:var(--accent); color:#fff; border:none;
    padding:12px; border-radius:8px; font-size:14px; font-weight:700; cursor:pointer;
    transition:background .15s;
  }
  .btn-save:hover { background:var(--accent2); }
  .btn-delete {
    background:#FDE8E6; color:var(--transport); border:1.5px solid #FACBC7;
    padding:12px 16px; border-radius:8px; font-size:13px; font-weight:700; cursor:pointer;
    transition:all .15s;
  }
  .btn-delete:hover { background:#FACBC7; }
  .btn-cancel {
    background:var(--bg); color:var(--muted); border:1.5px solid var(--border);
    padding:12px 16px; border-radius:8px; font-size:13px; font-weight:700; cursor:pointer;
  }
  .btn-cancel:hover { background:var(--border); }

  /* ── LEGEND ── */
  .legend {
    display:flex; gap:16px; flex-wrap:wrap; align-items:center;
    background:var(--surface); border:1px solid var(--border);
    border-radius:var(--radius); padding:12px 18px; margin-bottom:16px;
    font-size:12px; color:var(--muted);
  }
  .leg-item { display:flex; align-items:center; gap:6px; }
  .leg-dot { width:12px; height:12px; border-radius:3px; flex-shrink:0; }

  /* ── BACK BTN ── */
  .btn-back {
    background:var(--bg); border:1.5px solid var(--border); color:var(--text);
    padding:7px 14px; border-radius:7px; cursor:pointer; font-size:13px; font-weight:600;
    display:flex; align-items:center; gap:6px; transition:all .15s;
  }
  .btn-back:hover { border-color:var(--accent); color:var(--accent); }

  /* ── TOAST ── */
  .toast {
    position:fixed; bottom:24px; right:24px; z-index:300;
    background:var(--header-bg); color:#fff;
    padding:12px 20px; border-radius:9px; font-size:13px; font-weight:600;
    transform:translateY(80px); opacity:0; transition:all .3s;
    pointer-events:none;
  }
  .toast.show { transform:translateY(0); opacity:1; }

  .count-badge {
    background:var(--accent); color:#fff; border-radius:10px;
    font-size:10px; font-weight:700; padding:1px 6px; margin-left:4px;
  }

  @media(max-width:640px){
    .topbar h1 { font-size:14px; }
    .month-label { font-size:15px; min-width:130px; }
    .day-cell { min-height:64px; }
    .month-grid { gap:2px; }
  }
</style>
</head>
<body>

<div class="topbar">
  <div style="display:flex;align-items:center;gap:12px;flex:1">
    <div id="backBtn" style="display:none">
      <button class="btn-back" onclick="showMonthView()">← Volver</button>
    </div>
    <div>
      <h1>✦ Agenda <span>Tratamientos</span></h1>
    </div>
  </div>
  <button class="btn-nav" onclick="prevMonth()">‹</button>
  <div class="month-label" id="monthLabel"></div>
  <button class="btn-nav" onclick="nextMonth()">›</button>
</div>

<div class="container">

  <!-- MONTH VIEW -->
  <div id="monthView">
    <div class="legend">
      <div class="leg-item"><div class="leg-dot" style="background:var(--booked-bg);border:1.5px solid var(--accent)"></div> Reservado</div>
      <div class="leg-item"><div class="leg-dot" style="background:var(--transport-bg);border:1.5px solid var(--transport)"></div> Movilización</div>
      <div class="leg-item"><div class="leg-dot" style="background:var(--weekend-bg);border:1.5px solid var(--border)"></div> Fin de semana</div>
      <div class="leg-item"><div class="leg-dot" style="background:var(--surface);border:1.5px solid var(--border)"></div> Disponible</div>
    </div>
    <div class="month-grid" id="monthGrid"></div>
  </div>

  <!-- DAY VIEW -->
  <div id="dayView" class="day-view">
    <div class="day-header-bar">
      <div>
        <div class="day-title" id="dayTitle"></div>
        <div class="day-subtitle" id="daySubtitle"></div>
      </div>
      <div id="dayStats" style="text-align:right;font-size:13px;color:var(--muted)"></div>
    </div>
    <div class="legend">
      <div class="leg-item"><div class="leg-dot" style="background:var(--booked-bg);border:1.5px solid var(--accent)"></div> Reservado — haz clic para editar</div>
      <div class="leg-item"><div class="leg-dot" style="background:var(--transport-bg);border:1.5px solid var(--transport)"></div> 🚗 Movilización (30 min — automático)</div>
      <div class="leg-item"><div class="leg-dot" style="background:#F5EDE3;border:1.5px solid var(--border)"></div> Disponible — haz clic para agendar</div>
    </div>
    <div class="slots-grid" id="slotsGrid"></div>
  </div>

</div>

<!-- MODAL -->
<div class="overlay" id="overlay">
  <div class="modal">
    <div class="modal-header">
      <div>
        <h3 id="modalTitle">Nueva reserva</h3>
        <div class="slot-info" id="modalSlotInfo"></div>
      </div>
      <button class="modal-close" onclick="closeModal()">✕</button>
    </div>
    <div class="modal-body">
      <div class="field">
        <label>Nombre / Cliente</label>
        <input id="fNombre" type="text" placeholder="Nombre completo" autocomplete="off">
      </div>
      <div class="field">
        <label>Dirección</label>
        <input id="fDir" type="text" placeholder="Calle, número, depto." autocomplete="off">
      </div>
      <div class="field" style="display:grid;grid-template-columns:1fr 1fr;gap:12px">
        <div class="field" style="gap:5px">
          <label>Teléfono</label>
          <input id="fTel" type="tel" placeholder="+56 9 XXXX XXXX" autocomplete="off">
        </div>
        <div class="field" style="gap:5px">
          <label>Comuna</label>
          <select id="fComuna">
            <option value="">Seleccionar...</option>
          </select>
        </div>
      </div>
      <div class="field">
        <label>Notas</label>
        <input id="fNotas" type="text" placeholder="Observaciones adicionales" autocomplete="off">
      </div>
    </div>
    <div class="modal-footer">
      <button class="btn-cancel" onclick="closeModal()">Cancelar</button>
      <button class="btn-delete" id="btnDelete" style="display:none" onclick="deleteBooking()">Eliminar</button>
      <button class="btn-save" onclick="saveBooking()">Guardar reserva</button>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
// ── DATA ────────────────────────────────────────────────────────────────────
const COMUNAS = [
  "Cerrillos","Cerro Navia","Conchalí","El Bosque","Estación Central",
  "Huechuraba","Independencia","La Cisterna","La Florida","La Granja",
  "La Pintana","La Reina","Las Condes","Lo Barnechea","Lo Espejo",
  "Lo Prado","Macul","Maipú","Ñuñoa","Padre Hurtado","Pedro Aguirre Cerda",
  "Peñaflor","Peñalolén","Providencia","Pudahuel","Puente Alto","Quilicura",
  "Quinta Normal","Recoleta","Renca","San Bernardo","San Joaquín",
  "San Miguel","San Ramón","Santiago Centro","Vitacura","Talagante",
  "Melipilla","Buin","Colina","Lampa","Til Til","Pirque","San José de Maipo",
  "Alhué","Calera de Tango","Curacaví","El Monte","Isla de Maipo",
  "María Pinto","Paine"
].sort();

const DIAS  = ["Dom","Lun","Mar","Mié","Jue","Vie","Sáb"];
const MESES = ["Enero","Febrero","Marzo","Abril","Mayo","Junio",
               "Julio","Agosto","Septiembre","Octubre","Noviembre","Diciembre"];
const DIAS_FULL = ["Domingo","Lunes","Martes","Miércoles","Jueves","Viernes","Sábado"];

// Slots 09:00–19:30 (30 min, 22 bloques)
function genSlots() {
  const s = [];
  for(let h=9;h<20;h++) for(let m of[0,30]) s.push(`${String(h).padStart(2,'0')}:${m===0?'00':'30'}`);
  return s;
}
const SLOTS = genSlots();

// Storage
const KEY = 'agenda_tratamientos_v2';
let bookings = {};

function loadData() {
  try { bookings = JSON.parse(localStorage.getItem(KEY)||'{}'); } catch(e){ bookings={}; }
}
function saveData() {
  localStorage.setItem(KEY, JSON.stringify(bookings));
}
function bookingKey(y,m,d,slot) { return `${y}-${String(m+1).padStart(2,'0')}-${String(d).padStart(2,'0')}_${slot}`; }

// ── STATE ───────────────────────────────────────────────────────────────────
let currentYear  = 2026;
let currentMonth = new Date().getMonth(); // 0-based
let currentDay   = null;
let editingKey   = null;
let modalSlot    = null;

// Clamp to valid range (Jul 2026 – Dec 2027)
function clampMonth() {
  if(currentYear<2026||(currentYear===2026&&currentMonth<6)) { currentYear=2026; currentMonth=6; }
  if(currentYear>2027||(currentYear===2027&&currentMonth>11)) { currentYear=2027; currentMonth=11; }
}

// ── POPULATE SELECT ──────────────────────────────────────────────────────────
(function(){
  const sel = document.getElementById('fComuna');
  COMUNAS.forEach(c => { const o=document.createElement('option'); o.value=c; o.textContent=c; sel.appendChild(o); });
})();

// ── MONTH VIEW ───────────────────────────────────────────────────────────────
function renderMonth() {
  clampMonth();
  document.getElementById('monthLabel').textContent = `${MESES[currentMonth]} ${currentYear}`;
  const grid = document.getElementById('monthGrid');
  grid.innerHTML = '';

  DIAS.forEach(d => {
    const h = document.createElement('div');
    h.className='dow-header'; h.textContent=d; grid.appendChild(h);
  });

  const first = new Date(currentYear, currentMonth, 1).getDay(); // 0=Sun
  const days  = new Date(currentYear, currentMonth+1, 0).getDate();
  const today = new Date();

  // Blank cells before first
  for(let i=0;i<first;i++) {
    const blank=document.createElement('div'); blank.className='day-cell other-month'; grid.appendChild(blank);
  }

  for(let d=1;d<=days;d++) {
    const date = new Date(currentYear,currentMonth,d);
    const isWeekend = date.getDay()===0||date.getDay()===6;
    const isToday   = today.getFullYear()===currentYear&&today.getMonth()===currentMonth&&today.getDate()===d;

    const cell = document.createElement('div');
    cell.className='day-cell'+(isWeekend?' weekend':'')+(isToday?' today':'');
    cell.onclick = ()=>showDayView(d);

    const num = document.createElement('div');
    num.className='day-num'; num.textContent=d; cell.appendChild(num);

    // Dots for bookings
    const dots = document.createElement('div'); dots.className='day-dots';
    SLOTS.forEach(slot => {
      const k=bookingKey(currentYear,currentMonth,d,slot);
      if(bookings[k]&&bookings[k].nombre) {
        const dot=document.createElement('div'); dot.className='dot'; dots.appendChild(dot);
      }
    });
    cell.appendChild(dots);
    grid.appendChild(cell);
  }
}

// ── DAY VIEW ─────────────────────────────────────────────────────────────────
function showDayView(day) {
  currentDay = day;
  document.getElementById('monthView').style.display='none';
  const dv = document.getElementById('dayView'); dv.classList.add('active');
  document.getElementById('backBtn').style.display='block';

  const date = new Date(currentYear,currentMonth,day);
  const dow  = DIAS_FULL[date.getDay()];
  document.getElementById('dayTitle').textContent = `${dow} ${day} de ${MESES[currentMonth]} ${currentYear}`;

  renderSlots(day);
}

function renderSlots(day) {
  const grid = document.getElementById('slotsGrid');
  grid.innerHTML='';

  let bookedCount=0;
  SLOTS.forEach((slot,idx)=>{
    const k = bookingKey(currentYear,currentMonth,day,slot);
    const bk = bookings[k];
    if(bk&&bk.nombre) bookedCount++;
  });

  document.getElementById('daySubtitle').textContent = `${SLOTS.length} bloques de 30 min · 09:00 → 19:30`;
  document.getElementById('dayStats').innerHTML = bookedCount>0
    ? `<span style="color:var(--accent);font-weight:700">${bookedCount}</span> reserva${bookedCount>1?'s':''} este día`
    : `<span style="color:var(--muted)">Sin reservas</span>`;

  const date = new Date(currentYear,currentMonth,day);
  const isWeekend = date.getDay()===0||date.getDay()===6;

  SLOTS.forEach((slot,idx)=>{
    const k   = bookingKey(currentYear,currentMonth,day,slot);
    const bk  = bookings[k];
    const prevK = idx>0 ? bookingKey(currentYear,currentMonth,day,SLOTS[idx-1]) : null;
    const prevBk = prevK ? bookings[prevK] : null;
    const isTransport = prevBk&&prevBk.nombre&&!bk?.nombre;

    const row = document.createElement('div');
    row.className='slot'+(isWeekend?' weekend':'')+(bk&&bk.nombre?' booked':'')+(isTransport?' transport-slot':'');

    const timeDiv=document.createElement('div'); timeDiv.className='slot-time'; timeDiv.textContent=slot;
    const bodyDiv=document.createElement('div'); bodyDiv.className='slot-body';

    if(isTransport) {
      bodyDiv.innerHTML='<span class="transport-label">🚗 Movilización · 30 min</span>';
      row.title='Bloque reservado para traslado al siguiente cliente';
    } else if(bk&&bk.nombre) {
      bodyDiv.innerHTML=`
        <div style="flex:1">
          <div class="slot-name">${escHtml(bk.nombre)}</div>
          <div class="slot-meta">${[bk.comuna,bk.direccion].filter(Boolean).map(escHtml).join(' · ')}</div>
          ${bk.telefono?`<div class="slot-meta" style="margin-top:2px">📞 ${escHtml(bk.telefono)}</div>`:''}
          ${bk.notas?`<div class="slot-meta" style="margin-top:2px;font-style:italic">${escHtml(bk.notas)}</div>`:''}
        </div>
        <div style="color:var(--accent);font-size:18px;opacity:.6">✎</div>`;
      row.onclick=()=>openModal(slot,k);
    } else {
      bodyDiv.innerHTML='<span class="slot-add">+ Agregar reserva</span>';
      row.onclick=()=>openModal(slot,null);
    }

    row.appendChild(timeDiv); row.appendChild(bodyDiv);
    grid.appendChild(row);
  });
}

function showMonthView() {
  document.getElementById('monthView').style.display='';
  document.getElementById('dayView').classList.remove('active');
  document.getElementById('backBtn').style.display='none';
  currentDay=null;
  renderMonth();
}

// ── MODAL ────────────────────────────────────────────────────────────────────
function openModal(slot, key) {
  editingKey = key;
  modalSlot  = slot;
  const bk   = key ? (bookings[key]||{}) : {};
  const date = new Date(currentYear,currentMonth,currentDay);
  const dow  = DIAS_FULL[date.getDay()];

  document.getElementById('modalTitle').textContent = key ? 'Editar reserva' : 'Nueva reserva';
  document.getElementById('modalSlotInfo').textContent = `${dow} ${currentDay} de ${MESES[currentMonth]} ${currentYear} · ${slot}`;
  document.getElementById('fNombre').value = bk.nombre||'';
  document.getElementById('fDir').value    = bk.direccion||'';
  document.getElementById('fTel').value    = bk.telefono||'';
  document.getElementById('fComuna').value = bk.comuna||'';
  document.getElementById('fNotas').value  = bk.notas||'';
  document.getElementById('btnDelete').style.display = key?'':'none';
  document.getElementById('overlay').classList.add('open');
  setTimeout(()=>document.getElementById('fNombre').focus(),80);
}

function closeModal() {
  document.getElementById('overlay').classList.remove('open');
  editingKey=null; modalSlot=null;
}

function saveBooking() {
  const nombre = document.getElementById('fNombre').value.trim();
  if(!nombre) { document.getElementById('fNombre').focus(); document.getElementById('fNombre').style.borderColor='var(--transport)'; return; }
  document.getElementById('fNombre').style.borderColor='';

  const k = editingKey || bookingKey(currentYear,currentMonth,currentDay,modalSlot);
  bookings[k] = {
    nombre,
    direccion: document.getElementById('fDir').value.trim(),
    telefono:  document.getElementById('fTel').value.trim(),
    comuna:    document.getElementById('fComuna').value,
    notas:     document.getElementById('fNotas').value.trim(),
  };
  saveData();
  closeModal();
  renderSlots(currentDay);
  showToast('✓ Reserva guardada');
}

function deleteBooking() {
  if(!editingKey) return;
  delete bookings[editingKey];
  saveData();
  closeModal();
  renderSlots(currentDay);
  showToast('Reserva eliminada');
}

// ── NAV ──────────────────────────────────────────────────────────────────────
function prevMonth() {
  currentMonth--; if(currentMonth<0){currentMonth=11;currentYear--;}
  clampMonth();
  if(currentDay) showMonthView(); else renderMonth();
}
function nextMonth() {
  currentMonth++; if(currentMonth>11){currentMonth=0;currentYear++;}
  clampMonth();
  if(currentDay) showMonthView(); else renderMonth();
}

// ── UTILS ────────────────────────────────────────────────────────────────────
function escHtml(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }

function showToast(msg) {
  const t=document.getElementById('toast'); t.textContent=msg;
  t.classList.add('show'); setTimeout(()=>t.classList.remove('show'),2400);
}

// Close overlay on background click
document.getElementById('overlay').addEventListener('click',e=>{ if(e.target===document.getElementById('overlay')) closeModal(); });

// Keyboard
document.addEventListener('keydown',e=>{ if(e.key==='Escape') closeModal(); });

// ── INIT ─────────────────────────────────────────────────────────────────────
loadData();
// Start at current month if within range, otherwise Jul 2026
const now=new Date();
currentYear=now.getFullYear(); currentMonth=now.getMonth();
clampMonth();
renderMonth();
</script>
</body>
</html>