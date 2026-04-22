<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EstoqueApp</title>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,300;0,9..144,600;0,9..144,700;1,9..144,300&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
:root {
  --bg:        #f5f3ef;
  --surface:   #ffffff;
  --surface2:  #faf9f7;
  --border:    #e8e4dd;
  --border2:   #d4cfc6;
  --accent:    #2d6a4f;
  --accent-lt: #d8f3dc;
  --accent2:   #e07a2f;
  --accent2-lt:#fde8d4;
  --danger:    #c0392b;
  --danger-lt: #fdecea;
  --info:      #1a6faf;
  --info-lt:   #dceefb;
  --text:      #1c1a17;
  --text-2:    #5a5550;
  --text-3:    #9e9890;
  --shadow-sm: 0 1px 3px rgba(0,0,0,.06), 0 1px 2px rgba(0,0,0,.04);
  --shadow-md: 0 4px 16px rgba(0,0,0,.08), 0 1px 4px rgba(0,0,0,.04);
  --shadow-lg: 0 12px 40px rgba(0,0,0,.10), 0 2px 8px rgba(0,0,0,.05);
  --radius:    12px;
  --radius-sm: 8px;
  --font-disp: 'Fraunces', Georgia, serif;
  --font-body: 'DM Sans', sans-serif;
}

*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

body {
  background: var(--bg);
  color: var(--text);
  font-family: var(--font-body);
  min-height: 100vh;
  font-size: 15px;
  line-height: 1.5;
}

/* subtle dot pattern */
body::before {
  content:'';
  position:fixed; inset:0;
  background-image: radial-gradient(circle, #c8c2b840 1px, transparent 1px);
  background-size: 28px 28px;
  pointer-events:none;
  z-index:0;
}

.container {
  max-width: 980px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
  position: relative;
  z-index: 1;
}

/* ── Header ── */
header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  margin-bottom: 2.5rem;
  padding-bottom: 2rem;
  border-bottom: 1.5px solid var(--border);
  animation: slideDown .55s cubic-bezier(.22,.68,0,1.2) both;
}

.header-brand { display:flex; flex-direction:column; gap:.2rem; }

.brand-eyebrow {
  font-size: .7rem;
  letter-spacing: .18em;
  text-transform: uppercase;
  color: var(--accent);
  font-weight: 500;
}

.brand-title {
  font-family: var(--font-disp);
  font-size: 2.6rem;
  font-weight: 700;
  line-height: 1;
  color: var(--text);
  letter-spacing: -.02em;
}

.brand-title em {
  font-style: italic;
  font-weight: 300;
  color: var(--accent);
}

/* stat cards */
.stats-row {
  display: flex;
  gap: .75rem;
}

.stat-card {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-sm);
  padding: .75rem 1.1rem;
  text-align: right;
  box-shadow: var(--shadow-sm);
  min-width: 80px;
  transition: box-shadow .2s, transform .2s;
}
.stat-card:hover { box-shadow: var(--shadow-md); transform: translateY(-1px); }

.stat-num {
  font-family: var(--font-disp);
  font-size: 1.8rem;
  font-weight: 600;
  line-height: 1;
  color: var(--text);
}

.stat-lbl {
  font-size: .65rem;
  letter-spacing: .1em;
  text-transform: uppercase;
  color: var(--text-3);
  margin-top: .15rem;
}

/* ── Tabs ── */
.tabs {
  display: flex;
  gap: .25rem;
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius);
  padding: .3rem;
  margin-bottom: 1.75rem;
  width: fit-content;
  box-shadow: var(--shadow-sm);
  animation: slideDown .55s .1s cubic-bezier(.22,.68,0,1.2) both;
}

.tab {
  font-family: var(--font-body);
  font-size: .8rem;
  font-weight: 500;
  padding: .55rem 1.25rem;
  border-radius: calc(var(--radius) - 4px);
  border: none;
  cursor: pointer;
  color: var(--text-2);
  background: transparent;
  transition: all .2s;
  letter-spacing: .01em;
  white-space: nowrap;
}
.tab:hover { color: var(--text); background: var(--bg); }
.tab.active {
  background: var(--accent);
  color: #fff;
  box-shadow: 0 2px 8px rgba(45,106,79,.35);
}

/* ── Panels ── */
.panel { display: none; }
.panel.active { display: block; animation: fadeUp .3s ease both; }

/* ── Card container ── */
.card {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.4rem;
  border-bottom: 1.5px solid var(--border);
  background: var(--surface2);
}

.card-title {
  font-size: .7rem;
  font-weight: 600;
  letter-spacing: .13em;
  text-transform: uppercase;
  color: var(--text-2);
}

.card-body { padding: 1.25rem 1.4rem; }

/* ── Form ── */
.form-grid { display: grid; grid-template-columns: 1fr 160px 110px auto; gap: .75rem; align-items: end; }
.form-grid-cat { display: grid; grid-template-columns: 1fr auto; gap: .75rem; align-items: end; }
.field { display: flex; flex-direction: column; gap: .4rem; }

label {
  font-size: .7rem;
  font-weight: 500;
  letter-spacing: .05em;
  color: var(--text-2);
}

input[type="text"],
input[type="number"],
select {
  background: var(--bg);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text);
  font-family: var(--font-body);
  font-size: .9rem;
  padding: .65rem .9rem;
  outline: none;
  transition: border-color .2s, box-shadow .2s;
  width: 100%;
  -moz-appearance: textfield;
}
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button { -webkit-appearance: none; }
input:focus, select:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(45,106,79,.12);
  background: #fff;
}
select { cursor: pointer; }

.btn-primary {
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: var(--radius-sm);
  font-family: var(--font-body);
  font-weight: 600;
  font-size: .82rem;
  padding: .7rem 1.4rem;
  cursor: pointer;
  white-space: nowrap;
  transition: all .2s;
  box-shadow: 0 2px 8px rgba(45,106,79,.25);
  height: fit-content;
  letter-spacing: .02em;
}
.btn-primary:hover { background: #235c40; box-shadow: 0 4px 14px rgba(45,106,79,.35); transform: translateY(-1px); }
.btn-primary:active { transform: translateY(0); box-shadow: none; }

/* ── Toolbar ── */
.toolbar {
  display: flex;
  gap: .6rem;
  margin-bottom: 1rem;
  align-items: center;
  flex-wrap: wrap;
}

.search-box {
  position: relative;
  flex: 1;
  min-width: 160px;
}
.search-icon {
  position: absolute;
  left: .9rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-3);
  font-size: .95rem;
  pointer-events: none;
}
.search-box input { padding-left: 2.4rem; }

.btn-ghost {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text-2);
  font-family: var(--font-body);
  font-size: .78rem;
  font-weight: 500;
  padding: .6rem 1rem;
  cursor: pointer;
  transition: all .2s;
  white-space: nowrap;
}
.btn-ghost:hover { border-color: var(--accent); color: var(--accent); background: var(--accent-lt); }
.btn-ghost.danger:hover { border-color: var(--danger); color: var(--danger); background: var(--danger-lt); }

/* ── Filter chips ── */
.chips { display: flex; gap: .5rem; flex-wrap: wrap; margin-bottom: 1rem; }
.chip {
  font-size: .72rem;
  font-weight: 500;
  padding: .35rem .85rem;
  border-radius: 99px;
  border: 1.5px solid var(--border);
  cursor: pointer;
  color: var(--text-2);
  background: var(--surface);
  transition: all .15s;
}
.chip:hover { border-color: var(--border2); color: var(--text); }
.chip.active { border-color: var(--accent); color: var(--accent); background: var(--accent-lt); font-weight: 600; }

/* ── Table ── */
.tbl-head {
  display: grid;
  grid-template-columns: 1fr 140px 100px 140px 52px;
  padding: .55rem 1.2rem;
  background: var(--surface2);
  border-bottom: 1.5px solid var(--border);
}
.th {
  font-size: .65rem;
  font-weight: 600;
  letter-spacing: .1em;
  text-transform: uppercase;
  color: var(--text-3);
}
.th:not(:first-child) { text-align: center; }

.product-list { min-height: 140px; }

.empty-state {
  padding: 3.5rem 2rem;
  text-align: center;
  display: none;
  border-top: 1.5px solid var(--border);
}
.empty-state.visible { display: block; }
.empty-icon { font-size: 2.5rem; margin-bottom: .75rem; opacity: .3; }
.empty-text { font-size: .82rem; color: var(--text-3); letter-spacing: .05em; }

.product-row {
  display: grid;
  grid-template-columns: 1fr 140px 100px 140px 52px;
  padding: .85rem 1.2rem;
  border-top: 1.5px solid var(--border);
  align-items: center;
  transition: background .15s;
  animation: rowIn .25s ease both;
}
.product-row:hover { background: var(--surface2); }

.product-name { font-weight: 500; font-size: .92rem; }

.cat-cell { display: flex; justify-content: center; }
.badge {
  font-size: .65rem;
  font-weight: 600;
  letter-spacing: .06em;
  text-transform: uppercase;
  padding: .25rem .7rem;
  border-radius: 99px;
}

.product-qty-cell { text-align: center; }
.qty-num {
  font-family: var(--font-disp);
  font-size: 1.2rem;
  font-weight: 600;
}
.qty-zero { color: var(--danger); }
.qty-low  { color: var(--accent2); }
.qty-ok   { color: var(--accent); }

.qty-controls { display: flex; align-items: center; justify-content: center; gap: .5rem; }
.qty-btn {
  width: 28px; height: 28px;
  border-radius: 50%;
  border: 1.5px solid var(--border);
  background: var(--surface);
  color: var(--text-2);
  font-size: 1rem;
  cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: all .15s;
  box-shadow: var(--shadow-sm);
  line-height: 1;
  padding: 0;
}
.qty-btn:hover { border-color: var(--accent); color: var(--accent); background: var(--accent-lt); }
.qty-btn.minus:hover { border-color: var(--danger); color: var(--danger); background: var(--danger-lt); }

.qty-val {
  font-family: var(--font-disp);
  font-weight: 600;
  font-size: .95rem;
  min-width: 2.2rem;
  text-align: center;
}

.btn-del {
  background: transparent; border: none; cursor: pointer;
  color: var(--text-3); font-size: .85rem;
  width: 30px; height: 30px; border-radius: 6px;
  display: flex; align-items: center; justify-content: center;
  margin: 0 auto; transition: all .15s;
}
.btn-del:hover { background: var(--danger-lt); color: var(--danger); }

/* ── Categorias ── */
.cat-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(175px, 1fr)); gap: 1rem; }
.cat-card {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius);
  padding: 1.15rem 1.2rem 1rem;
  position: relative;
  box-shadow: var(--shadow-sm);
  animation: rowIn .25s ease both;
  transition: box-shadow .2s, transform .2s;
  overflow: hidden;
}
.cat-card:hover { box-shadow: var(--shadow-md); transform: translateY(-2px); }
.cat-card::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 4px;
}
.cat-name { font-weight: 600; font-size: .9rem; margin-bottom: .25rem; }
.cat-count { font-size: .75rem; color: var(--text-3); }
.cat-del { position: absolute; top: .6rem; right: .6rem; }

/* ── Histórico ── */
.hist-head {
  display: grid;
  grid-template-columns: 140px 1fr 100px 80px;
  padding: .55rem 1.2rem;
  background: var(--surface2);
  border-bottom: 1.5px solid var(--border);
}
.hist-row {
  display: grid;
  grid-template-columns: 140px 1fr 100px 80px;
  padding: .8rem 1.2rem;
  border-top: 1.5px solid var(--border);
  align-items: center;
  transition: background .15s;
  animation: rowIn .2s ease both;
}
.hist-row:first-child { border-top: none; }
.hist-row:hover { background: var(--surface2); }
.hist-time { font-size: .7rem; color: var(--text-3); line-height: 1.6; }
.hist-prod { font-weight: 500; font-size: .88rem; }
.hist-tipo { text-align: center; }
.tipo-pill {
  display: inline-block;
  font-size: .62rem;
  font-weight: 600;
  letter-spacing: .08em;
  text-transform: uppercase;
  padding: .22rem .65rem;
  border-radius: 99px;
}
.hist-delta { font-family: var(--font-disp); font-weight: 600; font-size: 1rem; text-align: center; }
.d-pos { color: var(--accent); }
.d-neg { color: var(--danger); }
.d-new { color: var(--info); }
.hist-empty {
  padding: 3rem;
  text-align: center;
  font-size: .82rem;
  color: var(--text-3);
}

/* ── Toast ── */
.toast {
  position: fixed; bottom: 2rem; right: 2rem;
  background: var(--text);
  color: #fff;
  font-size: .78rem;
  font-weight: 500;
  padding: .8rem 1.3rem;
  border-radius: var(--radius-sm);
  box-shadow: var(--shadow-lg);
  z-index: 999;
  transform: translateY(80px) scale(.95);
  opacity: 0;
  transition: all .3s cubic-bezier(.34,1.56,.64,1);
  max-width: 300px;
}
.toast.show { transform: translateY(0) scale(1); opacity: 1; }
.toast.error { background: var(--danger); }
.toast.success { background: var(--accent); }

/* ── Animations ── */
@keyframes slideDown { from { opacity:0; transform:translateY(-20px); } to { opacity:1; transform:translateY(0); } }
@keyframes fadeUp    { from { opacity:0; transform:translateY(12px);  } to { opacity:1; transform:translateY(0); } }
@keyframes rowIn     { from { opacity:0; transform:translateX(-8px);  } to { opacity:1; transform:translateX(0); } }

/* ── Responsive ── */
@media (max-width: 680px) {
  .brand-title { font-size: 2rem; }
  .stats-row { gap: .5rem; }
  .stat-num { font-size: 1.4rem; }
  .form-grid { grid-template-columns: 1fr; }
  .tbl-head, .product-row { grid-template-columns: 1fr 90px 40px; }
  .th:nth-child(2), .th:nth-child(3), .cat-cell, .product-qty-cell { display: none; }
  .hist-head, .hist-row { grid-template-columns: 110px 1fr 80px; }
  .hist-delta { display: none; }
}
</style>
</head>
<body>
<div class="container">

  <!-- Header -->
  <header>
    <div class="header-brand">
      <span class="brand-eyebrow">Controle de Inventário</span>
      <h1 class="brand-title">Estoque<em>App</em></h1>
    </div>
    <div class="stats-row">
      <div class="stat-card">
        <div class="stat-num" id="statProdutos">0</div>
        <div class="stat-lbl">Produtos</div>
      </div>
      <div class="stat-card">
        <div class="stat-num" id="statUnidades">0</div>
        <div class="stat-lbl">Unidades</div>
      </div>
      <div class="stat-card">
        <div class="stat-num" id="statZero" style="color:var(--danger)">0</div>
        <div class="stat-lbl">Zerados</div>
      </div>
    </div>
  </header>

  <!-- Tabs -->
  <div class="tabs">
    <button class="tab active" onclick="mudarTab('estoque')">📦 Estoque</button>
    <button class="tab"        onclick="mudarTab('categorias')">🏷️ Categorias</button>
    <button class="tab"        onclick="mudarTab('historico')">📋 Histórico</button>
  </div>

  <!-- ══ ESTOQUE ══ -->
  <div class="panel active" id="panel-estoque">

    <div class="card" style="margin-bottom:1.25rem;">
      <div class="card-header">
        <span class="card-title">Adicionar produto</span>
      </div>
      <div class="card-body">
        <div class="form-grid">
          <div class="field">
            <label>Nome do Produto</label>
            <input type="text" id="nomeProduto" placeholder="Ex: Caneta azul, Camiseta P…" autocomplete="off">
          </div>
          <div class="field">
            <label>Categoria</label>
            <select id="catProduto"><option value="">— Sem categoria —</option></select>
          </div>
          <div class="field">
            <label>Quantidade</label>
            <input type="number" id="qtdProduto" placeholder="0" min="0">
          </div>
          <button class="btn-primary" onclick="adicionarProduto()">+ Adicionar</button>
        </div>
      </div>
    </div>

    <div class="toolbar">
      <div class="search-box">
        <span class="search-icon">🔍</span>
        <input type="text" id="searchInput" placeholder="Buscar produto…" oninput="renderEstoque()">
      </div>
      <button class="btn-ghost" onclick="exportarCSV()">↓ Exportar CSV</button>
      <button class="btn-ghost danger" onclick="limparTudo()">✕ Limpar tudo</button>
    </div>

    <div class="chips" id="filterChips"></div>

    <div class="card" style="overflow:hidden;">
      <div class="tbl-head">
        <span class="th">Produto</span>
        <span class="th">Categoria</span>
        <span class="th">Qtd</span>
        <span class="th">Ajustar</span>
        <span class="th">Del</span>
      </div>
      <div class="product-list" id="productList"></div>
      <div class="empty-state" id="emptyState">
        <div class="empty-icon">📭</div>
        <div class="empty-text">Nenhum produto no estoque ainda.<br>Adicione o primeiro acima!</div>
      </div>
    </div>
  </div>

  <!-- ══ CATEGORIAS ══ -->
  <div class="panel" id="panel-categorias">
    <div class="card" style="margin-bottom:1.5rem;">
      <div class="card-header"><span class="card-title">Nova Categoria</span></div>
      <div class="card-body">
        <div class="form-grid-cat">
          <div class="field">
            <label>Nome da Categoria</label>
            <input type="text" id="nomeCat" placeholder="Ex: Alimentos, Eletrônicos, Roupas…" autocomplete="off">
          </div>
          <button class="btn-primary" onclick="adicionarCategoria()">+ Criar</button>
        </div>
      </div>
    </div>
    <div class="cat-grid" id="catGrid"></div>
    <div class="empty-state" id="emptyCat" style="display:none; padding:3rem; text-align:center; border:1.5px dashed var(--border); border-radius:var(--radius); margin-top:1rem;">
      <div class="empty-icon">🏷️</div>
      <div class="empty-text">Nenhuma categoria criada ainda.</div>
    </div>
  </div>

  <!-- ══ HISTÓRICO ══ -->
  <div class="panel" id="panel-historico">
    <div class="toolbar">
      <div class="search-box">
        <span class="search-icon">🔍</span>
        <input type="text" id="searchHist" placeholder="Buscar no histórico…" oninput="renderHistorico()">
      </div>
      <button class="btn-ghost danger" onclick="limparHistorico()">✕ Limpar histórico</button>
    </div>
    <div class="chips" id="histChips"></div>
    <div class="card" style="overflow:hidden;">
      <div class="hist-head">
        <span class="th">Data / Hora</span>
        <span class="th">Produto</span>
        <span class="th">Tipo</span>
        <span class="th">Δ Qtd</span>
      </div>
      <div id="histList"></div>
    </div>
  </div>

</div><!-- /container -->
<div class="toast" id="toast"></div>

<script>
const PALETTE = [
  {bg:'#d8f3dc',text:'#1e7e4a'},
  {bg:'#fde8d4',text:'#c05a1a'},
  {bg:'#dceefb',text:'#1557a0'},
  {bg:'#fce4ec',text:'#b52150'},
  {bg:'#f3e5f5',text:'#7b1fa2'},
  {bg:'#fff9c4',text:'#8d6e00'},
  {bg:'#e0f7fa',text:'#00697a'},
  {bg:'#ede7f6',text:'#5e35b1'},
];
const SK = { p:'estq_v3_produtos', c:'estq_v3_cats', h:'estq_v3_hist' };

let produtos=[],categorias=[],historico=[];
let catFiltro='todos', histFiltro='todos';

function carregar(){
  try{produtos  =JSON.parse(localStorage.getItem(SK.p))||[];}catch{produtos=[];}
  try{categorias=JSON.parse(localStorage.getItem(SK.c))||[];}catch{categorias=[];}
  try{historico =JSON.parse(localStorage.getItem(SK.h))||[];}catch{historico=[];}
  renderTudo();
}
function salvar(){
  localStorage.setItem(SK.p,JSON.stringify(produtos));
  localStorage.setItem(SK.c,JSON.stringify(categorias));
  localStorage.setItem(SK.h,JSON.stringify(historico));
}
function reg(tipo,prod,delta){
  historico.unshift({id:Date.now(),ts:new Date().toISOString(),tipo,prodId:prod.id,prodNome:prod.nome,delta});
  if(historico.length>500)historico=historico.slice(0,500);
}

// ── Produtos ──
function adicionarProduto(){
  const nome=document.getElementById('nomeProduto').value.trim();
  const qtd=parseInt(document.getElementById('qtdProduto').value)||0;
  const catId=document.getElementById('catProduto').value||null;
  if(!nome){toast('Informe o nome do produto!','error');document.getElementById('nomeProduto').focus();return;}
  const existe=produtos.find(p=>p.nome.toLowerCase()===nome.toLowerCase());
  if(existe){
    existe.quantidade+=qtd;
    if(catId)existe.catId=catId;
    reg(qtd>=0?'entrada':'saida',existe,qtd);
    toast(`"${nome}" atualizado (${qtd>=0?'+':''}${qtd})`,'success');
  }else{
    const novo={id:Date.now(),nome,quantidade:qtd,catId};
    produtos.unshift(novo);
    reg('novo',novo,qtd);
    toast(`"${nome}" adicionado!`,'success');
  }
  document.getElementById('nomeProduto').value='';
  document.getElementById('qtdProduto').value='';
  document.getElementById('nomeProduto').focus();
  salvar();renderTudo();
}

function ajustarQtd(id,delta){
  const p=produtos.find(p=>p.id===id);
  if(!p)return;
  const antes=p.quantidade;
  p.quantidade=Math.max(0,p.quantidade+delta);
  const real=p.quantidade-antes;
  if(real!==0)reg(real>0?'entrada':'saida',p,real);
  salvar();renderTudo();
}

function deletar(id){
  const p=produtos.find(p=>p.id===id);
  if(!p||!confirm(`Remover "${p.nome}"?`))return;
  reg('removido',p,-p.quantidade);
  produtos=produtos.filter(x=>x.id!==id);
  salvar();renderTudo();
  toast(`"${p.nome}" removido`);
}

function limparTudo(){
  if(!produtos.length){toast('Estoque já está vazio!','error');return;}
  if(!confirm('Limpar todo o estoque?'))return;
  produtos=[];salvar();renderTudo();toast('Estoque limpo!');
}

// ── Categorias ──
function adicionarCategoria(){
  const nome=document.getElementById('nomeCat').value.trim();
  if(!nome){toast('Informe o nome!','error');return;}
  if(categorias.find(c=>c.nome.toLowerCase()===nome.toLowerCase())){toast('Já existe!','error');return;}
  const pal=PALETTE[categorias.length%PALETTE.length];
  categorias.push({id:Date.now(),nome,bg:pal.bg,color:pal.text});
  document.getElementById('nomeCat').value='';
  salvar();renderTudo();toast(`"${nome}" criada!`,'success');
}

function deletarCategoria(id){
  const c=categorias.find(c=>c.id===id);
  if(!c||!confirm(`Remover "${c.nome}"?`))return;
  categorias=categorias.filter(x=>x.id!==id);
  produtos.forEach(p=>{if(String(p.catId)===String(id))p.catId=null;});
  salvar();renderTudo();toast(`"${c.nome}" removida`);
}

// ── Tabs ──
function mudarTab(nome){
  const ns=['estoque','categorias','historico'];
  document.querySelectorAll('.tab').forEach((t,i)=>t.classList.toggle('active',ns[i]===nome));
  document.querySelectorAll('.panel').forEach(p=>p.classList.remove('active'));
  document.getElementById('panel-'+nome).classList.add('active');
  if(nome==='historico')renderHistorico();
}

// ── Render ──
function renderTudo(){atualizarStats();renderEstoque();renderCategorias();popularSelect();renderChips();}

function atualizarStats(){
  document.getElementById('statProdutos').textContent=produtos.length;
  document.getElementById('statUnidades').textContent=produtos.reduce((a,p)=>a+p.quantidade,0);
  document.getElementById('statZero').textContent=produtos.filter(p=>p.quantidade===0).length;
}

function renderEstoque(){
  const busca=document.getElementById('searchInput').value.toLowerCase();
  const lista=document.getElementById('productList');
  const empty=document.getElementById('emptyState');
  let vis=produtos.filter(p=>p.nome.toLowerCase().includes(busca));
  if(catFiltro==='sem')vis=vis.filter(p=>!p.catId);
  else if(catFiltro!=='todos')vis=vis.filter(p=>String(p.catId)===catFiltro);
  if(!vis.length){lista.innerHTML='';empty.classList.add('visible');return;}
  empty.classList.remove('visible');
  lista.innerHTML=vis.map(p=>{
    const cls=p.quantidade===0?'qty-zero':p.quantidade<=5?'qty-low':'qty-ok';
    const cat=categorias.find(c=>String(c.id)===String(p.catId));
    const badge=cat
      ?`<span class="badge" style="background:${cat.bg};color:${cat.color}">${esc(cat.nome)}</span>`
      :`<span class="badge" style="background:#f0ede8;color:#9e9890">—</span>`;
    return `<div class="product-row">
      <div class="product-name">${esc(p.nome)}</div>
      <div class="cat-cell">${badge}</div>
      <div class="product-qty-cell"><span class="qty-num ${cls}">${p.quantidade}</span></div>
      <div class="qty-controls">
        <button class="qty-btn minus" onclick="ajustarQtd(${p.id},-1)" title="−1">−</button>
        <span class="qty-val ${cls}">${p.quantidade}</span>
        <button class="qty-btn" onclick="ajustarQtd(${p.id},+1)" title="+1">+</button>
      </div>
      <button class="btn-del" onclick="deletar(${p.id})" title="Remover">✕</button>
    </div>`;
  }).join('');
}

function renderChips(){
  const wrap=document.getElementById('filterChips');
  const comCat=categorias.filter(c=>produtos.some(p=>String(p.catId)===String(c.id)));
  const semCat=produtos.some(p=>!p.catId);
  const lista=[{id:'todos',nome:'Todos',bg:'',color:''},...comCat,...(semCat?[{id:'sem',nome:'Sem categoria',bg:'#f0ede8',color:'#9e9890'}]:[])];
  wrap.innerHTML=lista.map(c=>{
    const a=catFiltro===String(c.id);
    const st=a&&c.bg?`background:${c.bg};color:${c.color};border-color:${c.color}`:'';
    return `<button class="chip ${a?'active':''}" style="${st}" onclick="setCatFiltro('${c.id}')">${esc(c.nome)}</button>`;
  }).join('');
}

function setCatFiltro(id){catFiltro=id;renderChips();renderEstoque();}

function renderCategorias(){
  const grid=document.getElementById('catGrid');
  const empty=document.getElementById('emptyCat');
  if(!categorias.length){grid.innerHTML='';empty.style.display='block';return;}
  empty.style.display='none';
  grid.innerHTML=categorias.map(c=>{
    const count=produtos.filter(p=>String(p.catId)===String(c.id)).length;
    return `<div class="cat-card">
      <div style="position:absolute;top:0;left:0;right:0;height:4px;background:${c.color};border-radius:${getComputedStyle(document.documentElement).getPropertyValue('--radius')} ${getComputedStyle(document.documentElement).getPropertyValue('--radius')} 0 0"></div>
      <div class="cat-name" style="color:${c.color};margin-top:.5rem">${esc(c.nome)}</div>
      <div class="cat-count">${count} produto${count!==1?'s':''}</div>
      <button class="btn-del cat-del" onclick="deletarCategoria(${c.id})" title="Remover">✕</button>
    </div>`;
  }).join('');
}

function popularSelect(){
  const sel=document.getElementById('catProduto');
  const val=sel.value;
  sel.innerHTML='<option value="">— Sem categoria —</option>'
    +categorias.map(c=>`<option value="${c.id}"${String(val)===String(c.id)?' selected':''}>${esc(c.nome)}</option>`).join('');
}

function renderHistorico(){
  const busca=document.getElementById('searchHist').value.toLowerCase();
  const wrap=document.getElementById('histList');
  const chips=document.getElementById('histChips');
  const tipos=[{id:'todos',l:'Todos'},{id:'novo',l:'Novo'},{id:'entrada',l:'Entrada'},{id:'saida',l:'Saída'},{id:'removido',l:'Removido'}];
  chips.innerHTML=tipos.map(t=>`<button class="chip ${histFiltro===t.id?'active':''}" onclick="setHistFiltro('${t.id}')">${t.l}</button>`).join('');
  let lista=historico.filter(h=>h.prodNome.toLowerCase().includes(busca));
  if(histFiltro!=='todos')lista=lista.filter(h=>h.tipo===histFiltro);
  if(!lista.length){wrap.innerHTML=`<div class="hist-empty">Nenhum registro encontrado.</div>`;return;}
  const tipoPill={
    novo:{bg:'#dceefb',color:'#1557a0',l:'Novo'},
    entrada:{bg:'#d8f3dc',color:'#1e7e4a',l:'Entrada'},
    saida:{bg:'#fde8d4',color:'#c05a1a',l:'Saída'},
    removido:{bg:'#fdecea',color:'#c0392b',l:'Removido'},
  };
  wrap.innerHTML=lista.map(h=>{
    const d=new Date(h.ts);
    const data=d.toLocaleDateString('pt-BR');
    const hora=d.toLocaleTimeString('pt-BR',{hour:'2-digit',minute:'2-digit'});
    const dcls=h.tipo==='novo'?'d-new':h.delta>0?'d-pos':'d-neg';
    const dstr=(h.delta>0?'+':'')+h.delta;
    const tp=tipoPill[h.tipo]||{bg:'#f0ede8',color:'#9e9890',l:h.tipo};
    return `<div class="hist-row">
      <div class="hist-time">${data}<br>${hora}</div>
      <div class="hist-prod">${esc(h.prodNome)}</div>
      <div class="hist-tipo"><span class="tipo-pill" style="background:${tp.bg};color:${tp.color}">${tp.l}</span></div>
      <div class="hist-delta ${dcls}">${dstr}</div>
    </div>`;
  }).join('');
}

function setHistFiltro(t){histFiltro=t;renderHistorico();}

function limparHistorico(){
  if(!historico.length){toast('Histórico já está vazio!','error');return;}
  if(!confirm('Limpar todo o histórico?'))return;
  historico=[];salvar();renderHistorico();toast('Histórico limpo!');
}

function exportarCSV(){
  if(!produtos.length){toast('Nada pra exportar!','error');return;}
  const linhas=['Produto,Categoria,Quantidade',...produtos.map(p=>{
    const cat=categorias.find(c=>String(c.id)===String(p.catId));
    return `"${p.nome}","${cat?cat.nome:''}",${p.quantidade}`;
  })];
  const blob=new Blob([linhas.join('\n')],{type:'text/csv;charset=utf-8;'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');a.href=url;a.download='estoque.csv';a.click();
  URL.revokeObjectURL(url);toast('CSV exportado!','success');
}

function esc(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');}

let toastTimer;
function toast(msg,tipo=''){
  const el=document.getElementById('toast');
  el.textContent=msg;
  el.className='toast show'+(tipo?' '+tipo:'');
  clearTimeout(toastTimer);
  toastTimer=setTimeout(()=>el.classList.remove('show'),2800);
}

document.addEventListener('keydown',e=>{
  if(e.key!=='Enter')return;
  const id=document.activeElement.id;
  if(['nomeProduto','qtdProduto','catProduto'].includes(id))adicionarProduto();
  if(id==='nomeCat')adicionarCategoria();
});

carregar();
</script>
</body>
</html>