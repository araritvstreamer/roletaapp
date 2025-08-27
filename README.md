<!doctype html>
<html lang="pt-br">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Explorador Roleta — 1 arquivo (v2)</title>
  <!-- CDN libs -->
  <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <style>
*{box-sizing:border-box}body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Helvetica,Arial,sans-serif;margin:0;color:#111;background:#fff}
header{padding:16px 20px;border-bottom:1px solid #eee}
h1{margin:0 0 6px 0;font-size:22px}h2{margin:0 0 8px 0;font-size:18px}h3{margin:12px 0 8px 0;font-size:16px}
.muted{color:#666}.small{font-size:12px}.mt{margin-top:8px}
code{background:#f5f5f5;padding:2px 6px;border-radius:6px}
.tabs{display:flex;gap:8px;padding:10px 20px;border-bottom:1px solid #eee;background:#fafafa;position:sticky;top:0;z-index:5}
.tab{padding:8px 12px;border:1px solid #ddd;background:#fff;border-radius:10px;cursor:pointer}
.tab.active{background:#111;color:#fff;border-color:#111}
main{padding:16px 20px}
.panel{display:none}.panel.active{display:block}
.grid-3{display:grid;grid-template-columns:1fr;gap:16px}@media(min-width:980px){.grid-3{grid-template-columns:1fr 1fr 1fr}}
.card{background:#fff;border:1px solid #eee;border-radius:14px;padding:16px;box-shadow:0 6px 16px rgba(0,0,0,.04);margin-bottom:16px}
.row{display:flex;align-items:center}.gap{gap:8px}
.table-wrap{overflow:auto;margin-top:12px}
table{border-collapse:collapse;width:100%;font-size:14px}
th,td{padding:8px;border-top:1px solid #eee;text-align:left}
th{background:#fafafa}
textarea{width:100%;padding:8px;border:1px solid #ddd;border-radius:10px;font:inherit}
input{padding:8px;border:1px solid #ddd;border-radius:10px;font:inherit}
button{padding:8px 12px;border:1px solid #111;background:#111;color:#fff;border-radius:10px;cursor:pointer}
button.ghost{background:#fff;color:#111;border-color:#ddd}
footer{padding:12px 20px;border-top:1px solid #eee;color:#666;font-size:12px}
canvas{max-height:480px}
/* chips horizontais */
.chips{display:flex;flex-wrap:wrap;gap:6px}
.chip{padding:6px 10px;border-radius:999px;color:#fff;font-weight:600;font-size:13px}
.chip.A{background:#2563eb} .chip.B{background:#16a34a} .chip.C{background:#eab308;color:#111}
.chip.fora{background:#6b7280}
.sticky-bar{position:sticky;top:56px;background:#fff;padding:8px 0;z-index:4}
  </style>
</head>
<body>
  <header>
    <h1>Explorador Roleta — 1 arquivo (v2)</h1>
    <p class="muted">Tempo real com <b>3 regiões</b> + CSV (<code>numero,fonte,sheet</code>). Este arquivo único já funciona em hospedagem estática.</p>
  </header>

  <nav class="tabs">
    <button class="tab active" data-tab="realtime">Tempo real</button>
    <button class="tab" data-tab="csv">CSV</button>
    <button class="tab" data-tab="sobre">Sobre</button>
  </nav>

  <main>
    <section id="tab-realtime" class="panel active">
      <div class="grid-3">
        <div class="card">
          <h2>Configurar 3 regiões</h2>
          <label>Região A — números</label>
          <textarea id="regionA" rows="3" placeholder="ex.: 1 2 3 4 5"></textarea>
          <label>Região B — números</label>
          <textarea id="regionB" rows="3" placeholder="ex.: 6 7 8 9 10"></textarea>
          <label>Região C — números</label>
          <textarea id="regionC" rows="3" placeholder="ex.: 11 12 13 ..."></textarea>
          <div class="row gap">
            <button id="saveRegions">Salvar regiões</button>
            <button id="clearRegions" class="ghost">Limpar</button>
          </div>
          <p class="muted small">Mudanças nas caixas já atualizam os gráficos. Botão “Salvar” grava no navegador.</p>
        </div>

        <div class="card">
          <h2>Entrada de giros</h2>
          <div class="row gap">
            <input id="liveNum" placeholder="0–36">
            <button id="addSpin">Adicionar</button>
            <button id="clearSpins" class="ghost">Zerar</button>
            <button id="exportSpins" class="ghost">Exportar CSV</button>
          </div>
          <p class="muted small">Dica: digite o número e tecle <b>Enter</b>.</p>
          <h3 class="mt">Probabilidade atual por região</h3>
          <canvas id="chartProb"></canvas>
        </div>

        <div class="card">
          <h2>Rolling (janela = 30 giros)</h2>
          <canvas id="chartRolling"></canvas>
        </div>
      </div>

      <div class="card sticky-bar">
        <h2>Sequência (horizontal, últimos 60)</h2>
        <div class="chips" id="chips"></div>
      </div>

      <div class="card">
        <h2>Histórico (últimos 200 giros)</h2>
        <div class="table-wrap" id="tableLive"></div>
      </div>
    </section>

    <section id="tab-csv" class="panel">
      <div class="card">
        <h2>Carregar CSV</h2>
        <div class="row gap">
          <input id="csvInput" type="file" accept=".csv">
          <button id="demoBtn">Usar exemplo</button>
          <span id="csvStatus" class="muted"></span>
        </div>
      </div>

      <div class="card">
        <h2>Frequência por número (total)</h2>
        <canvas id="chartGeral"></canvas>
        <div class="table-wrap" id="tableGeral"></div>
      </div>

      <div class="card">
        <h2>Frequência por número — por fonte</h2>
        <canvas id="chartFonte"></canvas>
        <div class="table-wrap" id="tableFonte"></div>
      </div>

      <div class="card">
        <h2>Frequência por número — por sheet</h2>
        <canvas id="chartSheet"></canvas>
        <div class="table-wrap" id="tableSheet"></div>
      </div>
    </section>

    <section id="tab-sobre" class="panel">
      <div class="card">
        <h2>Sobre</h2>
        <p class="muted">[Não verificado] Ferramenta de análise/visualização dos seus dados. Não garante ganhos.</p>
      </div>
    </section>
  </main>

  <footer>
    <p class="muted small">Defina as regiões, digite os giros e use CSV para histórico. Use dados representativos e valide fora da amostra.</p>
  </footer>

  <script>
// Tabs
document.querySelectorAll('.tab').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
    btn.classList.add('active');
    document.querySelector('#tab-' + btn.dataset.tab).classList.add('active');
  });
});

// Utils
const parseNumber = (v) => {
  if (v === null || v === undefined) return null;
  const n = Number(v);
  if (Number.isFinite(n)) return Math.trunc(n);
  const m = String(v).match(/\b\d{1,2}\b/);
  return m ? Number(m[0]) : null;
};
const txtToSet = (s) => new Set(String(s).split(/[^0-9]+/).filter(Boolean).map(n=>Number(n)).filter(n=>Number.isInteger(n)&&n>=0&&n<=36));
const saveLS = (k,v) => { try { localStorage.setItem(k, JSON.stringify(v)); } catch {} };
const loadLS = (k,fallback) => { try { const x=localStorage.getItem(k); return x?JSON.parse(x):fallback; } catch { return fallback; } };

// Regions + live spins
let liveSpins = loadLS('liveSpins', []);
const regionAEl = document.getElementById('regionA');
const regionBEl = document.getElementById('regionB');
const regionCEl = document.getElementById('regionC');
regionAEl.value = loadLS('regionA_txt',''); regionBEl.value = loadLS('regionB_txt',''); regionCEl.value = loadLS('regionC_txt','');

const toRegion = (n) => {
  const A = txtToSet(regionAEl.value), B = txtToSet(regionBEl.value), C = txtToSet(regionCEl.value);
  if (A.has(n)) return 'A';
  if (B.has(n)) return 'B';
  if (C.has(n)) return 'C';
  return '(fora)';
};

// Charts
let chartProb, chartRolling, chartGeral, chartFonte, chartSheet;
// bar 0..1 (prob)
function renderBar01(ctx, labels, data, label) {
  if (ctx.__chart) ctx.__chart.destroy();
  ctx.__chart = new Chart(ctx, { type:'bar', data:{ labels: labels.map(String), datasets:[{ label, data, borderWidth:1 }] }, options:{ responsive:true, scales:{ y:{ beginAtZero:true, max:1 }}, plugins:{ legend:{ display:false }}}});
  return ctx.__chart;
}
// bar contagens
function renderBarCount(ctx, labels, data, label) {
  if (ctx.__chart) ctx.__chart.destroy();
  ctx.__chart = new Chart(ctx, { type:'bar', data:{ labels: labels.map(String), datasets:[{ label, data, borderWidth:1 }] }, options:{ responsive:true, scales:{ y:{ beginAtZero:true }}, plugins:{ legend:{ display:false }}}});
  return ctx.__chart;
}
function renderLineMulti(ctx, labels, series, names) {
  if (ctx.__chart) ctx.__chart.destroy();
  ctx.__chart = new Chart(ctx, { type:'line', data:{ labels, datasets: series.map((d,i)=>({ label:names[i], data:d, borderWidth:2, pointRadius:0 })) }, options:{ responsive:true, scales:{ y:{ beginAtZero:true, max:1 }}}});
  return ctx.__chart;
}
function renderStacked(ctx, labelsX, categories, seriesRows, rowNames, title) {
  if (ctx.__chart) ctx.__chart.destroy();
  const datasets = categories.map((cat)=>({ label:String(cat), data: seriesRows.map(row=>row[cat]||0), borderWidth:1, stack: 'stack-0' }));
  ctx.__chart = new Chart(ctx, { type:'bar', data:{ labels:rowNames, datasets }, options:{ responsive:true, scales:{ x:{ stacked:true }, y:{ stacked:true, beginAtZero:true }}, plugins:{ title:{ display:true, text:title }}}});
  return ctx.__chart;
}

function updateLive() {
  // anotados
  const annotated = liveSpins.map((n,i)=>({ idx:i+1, numero:n, regiao: toRegion(n) }));
  // probs
  const counts = { 'A':0, 'B':0, 'C':0, '(fora)':0 };
  annotated.forEach(r => counts[r.regiao] = (counts[r.regiao]||0) + 1);
  const total = Math.max(1, annotated.length);
  const probs = ['A','B','C','(fora)'].map(k => counts[k]/total);
  renderBar01(document.getElementById('chartProb'), ['A','B','C','(fora)'], probs, 'Probabilidade');
  // rolling 30
  const win=30; let a=0,b=0,c=0; const q=[]; const labels=[], sA=[], sB=[], sC=[];
  annotated.forEach((r,i)=>{ q.push(r.regiao); if(r.regiao==='A') a++; else if(r.regiao==='B') b++; else if(r.regiao==='C') c++; if(q.length>win){ const out=q.shift(); if(out==='A') a--; else if(out==='B') b--; else if(out==='C') c--; } const d=Math.min(i+1,win); labels.push(i+1); sA.push(a/d); sB.push(b/d); sC.push(c/d); });
  renderLineMulti(document.getElementById('chartRolling'), labels, [sA,sB,sC], ['A','B','C']);
  // chips horizontais (últimos 60, mais recentes à direita)
  const chipsEl = document.getElementById('chips'); chipsEl.innerHTML='';
  annotated.slice(-60).forEach(r=>{ const span=document.createElement('span'); span.className='chip ' + (r.regiao==='(fora)'?'fora':r.regiao); span.textContent = r.numero; chipsEl.appendChild(span); });
  // tabela
  const wrap=document.getElementById('tableLive'); wrap.innerHTML='';
  const table=document.createElement('table'); table.innerHTML='<thead><tr><th>#</th><th>Número</th><th>Região</th></tr></thead>';
  const tbody=document.createElement('tbody');
  annotated.slice(-200).reverse().forEach(r=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td>${r.idx}</td><td>${r.numero}</td><td>${r.regiao}</td>`; tbody.appendChild(tr); });
  table.appendChild(tbody); wrap.appendChild(table);
}

// salvar/limpar regiões
document.getElementById('saveRegions').addEventListener('click', ()=>{
  saveLS('regionA_txt', regionAEl.value);
  saveLS('regionB_txt', regionBEl.value);
  saveLS('regionC_txt', regionCEl.value);
  updateLive();
});
document.getElementById('clearRegions').addEventListener('click', ()=>{
  regionAEl.value=''; regionBEl.value=''; regionCEl.value='';
  saveLS('regionA_txt',''); saveLS('regionB_txt',''); saveLS('regionC_txt','');
  updateLive();
});
// atualiza enquanto digita (feedback imediato)
['input','change'].forEach(ev=>{
  regionAEl.addEventListener(ev, updateLive);
  regionBEl.addEventListener(ev, updateLive);
  regionCEl.addEventListener(ev, updateLive);
});

// live input
const liveNumEl = document.getElementById('liveNum');
document.getElementById('addSpin').addEventListener('click', ()=>{
  const n=parseNumber(liveNumEl.value);
  if(n!==null && n>=0 && n<=36){ liveSpins.push(n); saveLS('liveSpins', liveSpins); liveNumEl.value=''; updateLive(); }
});
liveNumEl.addEventListener('keydown', (e)=>{ if(e.key==='Enter'){ e.preventDefault(); document.getElementById('addSpin').click(); }});
document.getElementById('clearSpins').addEventListener('click', ()=>{ liveSpins=[]; saveLS('liveSpins', liveSpins); updateLive(); });
document.getElementById('exportSpins').addEventListener('click', ()=>{
  const csv='idx,numero,regiao\n'+ liveSpins.map((n,i)=>`${i+1},${n},${toRegion(n)}`).join('\n');
  const blob=new Blob([csv],{type:'text/csv;charset=utf-8;'}); const url=URL.createObjectURL(blob);
  const a=document.createElement('a'); a.href=url; a.download='live_spins.csv'; a.click(); URL.revokeObjectURL(url);
});
updateLive();

// CSV analysis
function fromCSVText(text){ const parsed = Papa.parse(text,{ header:true, skipEmptyLines:true }); const rows=(parsed.data||[]).map(r=>({ numero:r.numero,fonte:r.fonte,sheet:r.sheet })); return rows.filter(r=>r.numero!==undefined && r.numero!==''); }
function aggGeral(rows){ const m=new Map(); rows.forEach(r=>{ const n=parseNumber(r.numero); if(n===null||n<0||n>36) return; m.set(n,(m.get(n)||0)+1); }); return Array.from(m.entries()).map(([numero,contagem])=>({numero,contagem})).sort((a,b)=>b.contagem-a.contagem); }
function aggGroup(rows,key){ const numerosSet=new Set(); const groups=new Map(); rows.forEach(r=>{ const n=parseNumber(r.numero); if(n===null||n<0||n>36) return; numerosSet.add(n); const g=String(r[key]||`(sem ${key})`); if(!groups.has(g)) groups.set(g,new Map()); const inner=groups.get(g); inner.set(n,(inner.get(n)||0)+1); }); const numeros=Array.from(numerosSet).sort((a,b)=>a-b); const series=Array.from(groups.entries()).map(([name,inner])=>{ const row={name}; numeros.forEach(n=>row[n]=inner.get(n)||0); return row; }); return { numeros, series }; }
function renderTable(container, rows){ container.innerHTML=''; const table=document.createElement('table'); table.innerHTML='<thead><tr><th>Número</th><th>Contagem</th></tr></thead>'; const tbody=document.createElement('tbody'); rows.forEach(r=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td>${r.numero}</td><td>${r.contagem}</td>`; tbody.appendChild(tr); }); table.appendChild(tbody); container.appendChild(table); }
function renderTableGrouped(container, numeros, series, firstColLabel){ container.innerHTML=''; const table=document.createElement('table'); const thead=document.createElement('thead'); const headRow=document.createElement('tr'); const th0=document.createElement('th'); th0.textContent=firstColLabel; headRow.appendChild(th0); numeros.forEach(n=>{ const th=document.createElement('th'); th.textContent=String(n); th.style.textAlign='right'; headRow.appendChild(th); }); thead.appendChild(headRow); table.appendChild(thead); const tbody=document.createElement('tbody'); series.forEach(row=>{ const tr=document.createElement('tr'); const td=document.createElement('td'); td.textContent=row.name; tr.appendChild(td); numeros.forEach(n=>{ const td2=document.createElement('td'); td2.textContent=String(row[n]||0); td2.style.textAlign='right'; tr.appendChild(td2); }); tbody.appendChild(tr); }); table.appendChild(tbody); container.appendChild(table); }

const csvStatus=document.getElementById('csvStatus');
document.getElementById('csvInput').addEventListener('change',(e)=>{ const file=e.target.files[0]; if(!file) return; const reader=new FileReader(); reader.onload=()=>{ try{ const rows=fromCSVText(reader.result); const geral=aggGeral(rows); renderBarCount(document.getElementById('chartGeral'), geral.map(r=>r.numero), geral.map(r=>r.contagem), 'Frequência total'); renderTable(document.getElementById('tableGeral'), geral); const porFonte=aggGroup(rows,'fonte'); renderStacked(document.getElementById('chartFonte'), porFonte.numeros, porFonte.numeros, porFonte.series, porFonte.series.map(s=>s.name), 'Por fonte'); renderTableGrouped(document.getElementById('tableFonte'), porFonte.numeros, porFonte.series, 'Fonte'); const porSheet=aggGroup(rows,'sheet'); renderStacked(document.getElementById('chartSheet'), porSheet.numeros, porSheet.numeros, porSheet.series, porSheet.series.map(s=>s.name), 'Por sheet'); renderTableGrouped(document.getElementById('tableSheet'), porSheet.numeros, porSheet.series, 'Sheet'); csvStatus.textContent=`Linhas: ${rows.length}`; } catch(err){ csvStatus.textContent=`Erro: ${err}`; } }; reader.readAsText(file,'utf-8'); });
document.getElementById('demoBtn').addEventListener('click',()=>{ const demo=`numero,fonte,sheet\n30,exemplo.xlsx,Plan1\n9,exemplo.xlsx,Plan1\n11,exemplo.xlsx,Plan2\n24,exemplo2.xlsx,PlanA\n2,exemplo2.xlsx,PlanA\n3,exemplo2.xlsx,PlanB\n7,exemplo3.xlsx,Plan1\n8,exemplo3.xlsx,Plan1\n10,exemplo3.xlsx,Plan2\n27,exemplo3.xlsx,Plan2\n28,exemplo3.xlsx,Plan3\n5,exemplo4.xlsx,PlanZ\n16,exemplo4.xlsx,PlanZ\n4,exemplo4.xlsx,PlanZ\n33,exemplo4.xlsx,PlanZ\n`; const rows=fromCSVText(demo); const geral=aggGeral(rows); renderBarCount(document.getElementById('chartGeral'), geral.map(r=>r.numero), geral.map(r=>r.contagem), 'Frequência total'); renderTable(document.getElementById('tableGeral'), geral); const porFonte=aggGroup(rows,'fonte'); renderStacked(document.getElementById('chartFonte'), porFonte.numeros, porFonte.numeros, porFonte.series, porFonte.series.map(s=>s.name), 'Por fonte'); renderTableGrouped(document.getElementById('tableFonte'), porFonte.numeros, porFonte.series, 'Fonte'); const porSheet=aggGroup(rows,'sheet'); renderStacked(document.getElementById('chartSheet'), porSheet.numeros, porSheet.numeros, porSheet.series, porSheet.series.map(s=>s.name), 'Por sheet'); renderTableGrouped(document.getElementById('tableSheet'), porSheet.numeros, porSheet.series, 'Sheet'); csvStatus.textContent=`Demo carregado: ${rows.length} linhas`; });
  </script>
</body>
</html>
