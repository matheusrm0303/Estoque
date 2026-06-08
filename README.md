import { useState, useEffect } from "react";

const STORAGE_KEY = "diarias-app";
const MESES = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"];
const MESES_FULL = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
const TIPOS = [
  { tipo:"rua",      label:"Diária de Rua",     emoji:"🚶", valor:200, acento:"#38BDF8" },
  { tipo:"deposito", label:"Diária de Depósito", emoji:"🏭", valor:100, acento:"#38BDF8" },
];
const SC = {
  pendente: { label:"Pendente", cor:"#FACC15", bg:"rgba(250,204,21,.1)"  },
  pago:     { label:"Pago",     cor:"#22D3EE", bg:"rgba(34,211,238,.1)"  },
  cancelado:{ label:"Cancelado",cor:"#F87171", bg:"rgba(248,113,113,.1)" },
};

function uid()  { return Date.now().toString(36)+Math.random().toString(36).slice(2); }
function fm(v)  { return (v||0).toLocaleString("pt-BR",{style:"currency",currency:"BRL"}); }
function fd(iso){ if(!iso)return""; const[y,m,d]=iso.split("-"); return`${d}/${m}/${y}`; }
function hoje() { return new Date().toISOString().slice(0,10); }
function calcTotal(d) { return (d.valorDiaria||0)+(d.almoco||0)+(d.janta||0); }

function loadJsPDF(){
  return new Promise(res=>{
    if(window.jspdf){res(window.jspdf.jsPDF);return;}
    const s=document.createElement("script");
    s.src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js";
    s.onload=()=>res(window.jspdf.jsPDF);
    document.head.appendChild(s);
  });
}

function abrirBlob(doc, nome) {
  try {
    // Tenta download direto
    const blob = doc.output("blob");
    const url  = URL.createObjectURL(blob);
    const a    = document.createElement("a");
    a.href = url; a.download = nome; a.target = "_blank";
    document.body.appendChild(a); a.click();
    setTimeout(()=>{ document.body.removeChild(a); URL.revokeObjectURL(url); }, 2000);
  } catch(e) {
    // Fallback: abre como data URI em nova aba
    try {
      const uri = doc.output("datauristring");
      const win = window.open("", "_blank");
      if (win) {
        win.document.write(
          '<html><head><title>'+nome+'</title></head><body style="margin:0">' +
          '<iframe src="' + uri + '" style="width:100%;height:100vh;border:none"></iframe>' +
          '</body></html>'
        );
        win.document.close();
      } else {
        // Último recurso: link clicável
        const uri2 = doc.output("datauristring");
        const a = document.createElement("a");
        a.href = uri2; a.download = nome;
        document.body.appendChild(a); a.click();
        document.body.removeChild(a);
      }
    } catch(e2) {
      alert("Não foi possível abrir o PDF: " + e2.message);
    }
  }
}

async function exportPDF(diarias, movimentos, mes) {
  const jsPDF = await loadJsPDF();
  const doc = new jsPDF({ unit:"mm", format:"a4" });
  const W=210, M=14; let y=0;
  const INK=[9,11,20], PUR=[14,165,233], MINT=[74,222,128], YEL=[250,204,21],
        GRAY=[100,116,139], WHITE=[255,255,255], SKY=[56,189,248];

  const chk=(n=10)=>{ if(y+n>282){doc.addPage();doc.setFillColor(...INK);doc.rect(0,0,W,10,"F");y=18;} };
  const sec=(txt,cor)=>{ chk(14);doc.setFillColor(...cor);doc.rect(M,y,W-M*2,8,"F");doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...WHITE);doc.text(txt,M+4,y+5.5);y+=12; };

  const dF = diarias.filter(d=>!mes||d.data?.slice(0,7)===mes).sort((a,b)=>a.data.localeCompare(b.data));
  const totD  = dF.filter(d=>d.status!=="cancelado").reduce((s,d)=>s+calcTotal(d),0);
  const totA  = movimentos.filter(m=>m.tipo==="adiantamento").reduce((s,m)=>s+(m.valor||0),0);
  const totF2 = movimentos.filter(m=>m.tipo==="fechamento").reduce((s,m)=>s+(m.valor||0),0);
  const saldo = totD-(totA+totF2);

  doc.setFillColor(...INK);doc.rect(0,0,W,30,"F");
  doc.setFillColor(...PUR);doc.rect(0,0,5,30,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(17);doc.setTextColor(...WHITE);
  doc.text("RELATÓRIO DE DIÁRIAS",M+7,14);
  const periodo=mes?(()=>{const[yr,mo]=mes.split("-");return MESES_FULL[parseInt(mo)-1]+" "+yr;})():"Todos os períodos";
  doc.setFontSize(9);doc.setTextColor(...GRAY);
  doc.text(periodo,M+7,23);doc.text("Gerado: "+new Date().toLocaleDateString("pt-BR"),W-M,23,{align:"right"});
  y=38;

  const cw=(W-M*2-8)/3;
  [[fm(totD),"Total em Diárias",SKY],[fm(totA+totF2),"Total Recebido",MINT],[fm(saldo),"Saldo Pendente",saldo>0?YEL:MINT]].forEach(function(item,i){
    const val=item[0],lbl=item[1],cor=item[2];
    const x=M+i*(cw+4);
    doc.setFillColor(18,20,36);doc.roundedRect(x,y,cw,22,2,2,"F");
    doc.setFillColor(...cor);doc.rect(x,y,3,22,"F");
    doc.setFont("helvetica","normal");doc.setFontSize(7);doc.setTextColor(...GRAY);doc.text(lbl.toUpperCase(),x+6,y+8);
    doc.setFont("helvetica","bold");doc.setFontSize(11);doc.setTextColor(...WHITE);doc.text(val,x+6,y+17);
  });y+=28;

  sec("DIÁRIAS TRABALHADAS",PUR);
  const cols=[M,M+26,M+74,M+114,M+144,M+170];
  const thd=function(){ doc.setFillColor(18,20,36);doc.rect(M,y,W-M*2,7,"F");doc.setFont("helvetica","bold");doc.setFontSize(7.5);doc.setTextColor(...GRAY);["Data","Evento","Tipo","Total","Status","Obs"].forEach(function(h,i){doc.text(h,cols[i],y+5);}); y+=8; };
  thd();
  dF.forEach(function(d,i){
    chk(8);
    doc.setFillColor(i%2===0?14:11,i%2===0?16:14,i%2===0?30:24);doc.rect(M,y-1,W-M*2,7,"F");
    const sc=SC[d.status]||SC.pendente;
    const tipo=(TIPOS.find(function(t){return t.tipo===d.tipoDiaria;})||{}).label||"-";
    const tot=calcTotal(d);
    doc.setFont("helvetica","normal");doc.setFontSize(8);doc.setTextColor(...WHITE);
    doc.text(fd(d.data),cols[0],y+4);
    doc.text((d.evento||"-").slice(0,20),cols[1],y+4);
    doc.text(tipo.replace("Diária de ",""),cols[2],y+4);
    doc.text(fm(tot)+(d.dobra?" ⚡":""),cols[3],y+4);
    const rgb=sc.cor.slice(1);
    doc.setTextColor(parseInt(rgb.slice(0,2),16),parseInt(rgb.slice(2,4),16),parseInt(rgb.slice(4,6),16));
    doc.text(sc.label,cols[4],y+4);
    doc.setTextColor(...WHITE);doc.text((d.obs||"").slice(0,12),cols[5],y+4);
    y+=7;
  });
  if(!dF.length){doc.setFontSize(9);doc.setTextColor(...GRAY);doc.text("Nenhuma diária no período.",M+4,y+5);y+=10;}
  y+=2;doc.setFillColor(...PUR);doc.rect(M,y,W-M*2,8,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...WHITE);
  doc.text("TOTAL",cols[0],y+5.5);doc.text(dF.length+" diárias",cols[2],y+5.5);doc.text(fm(totD),cols[3],y+5.5);y+=14;

  chk(20);sec("MOVIMENTOS FINANCEIROS",MINT);
  const mc=[M,M+46,M+94,M+135];
  doc.setFillColor(18,20,36);doc.rect(M,y,W-M*2,7,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(7.5);doc.setTextColor(...GRAY);
  ["Data","Tipo","Valor","Observação"].forEach(function(h,i){doc.text(h,mc[i],y+5);});y+=8;
  movimentos.slice().sort(function(a,b){return a.data.localeCompare(b.data);}).forEach(function(m,i){
    chk(8);
    doc.setFillColor(i%2===0?14:11,i%2===0?16:14,i%2===0?30:24);doc.rect(M,y-1,W-M*2,7,"F");
    doc.setFont("helvetica","normal");doc.setFontSize(8);doc.setTextColor(...WHITE);doc.text(fd(m.data),mc[0],y+4);
    doc.setTextColor(...(m.tipo==="adiantamento"?SKY:MINT));doc.text(m.tipo==="adiantamento"?"Adiantamento":"Fechamento",mc[1],y+4);
    doc.setTextColor(...MINT);doc.text(fm(m.valor),mc[2],y+4);
    doc.setTextColor(...WHITE);doc.text((m.obs||"").slice(0,24),mc[3],y+4);y+=7;
  });
  if(!movimentos.length){doc.setFontSize(9);doc.setTextColor(...GRAY);doc.text("Nenhum movimento.",M+4,y+5);y+=10;}
  y+=2;doc.setFillColor(...MINT);doc.rect(M,y,W-M*2,8,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...INK);
  doc.text("TOTAL RECEBIDO",mc[0],y+5.5);doc.text(fm(totA+totF2),mc[2],y+5.5);y+=14;

  chk(28);doc.setFillColor(saldo>0?40:6,saldo>0?35:50,saldo>0?8:30);doc.roundedRect(M,y,W-M*2,24,3,3,"F");
  doc.setFillColor(...(saldo>0?YEL:MINT));doc.rect(M,y,5,24,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(11);doc.setTextColor(...WHITE);doc.text("SALDO PENDENTE",M+10,y+10);
  doc.setFontSize(16);doc.setTextColor(...(saldo>0?YEL:MINT));doc.text(fm(saldo),M+10,y+20);
  doc.setFontSize(8);doc.setTextColor(...GRAY);
  doc.text("Adiant.: "+fm(totA)+"  ·  Total: "+fm(totD),W-M,y+20,{align:"right"});

  const pages=doc.getNumberOfPages();
  for(let i=1;i<=pages;i++){doc.setPage(i);doc.setFillColor(...INK);doc.rect(0,288,W,9,"F");doc.setFont("helvetica","normal");doc.setFontSize(7);doc.setTextColor(...GRAY);doc.text("Relatório de Diárias",M,294);doc.text("Pág "+i+"/"+pages,W-M,294,{align:"right"});}

  const nome = mes ? (function(){ const parts=mes.split("-"); return "diarias_"+MESES[parseInt(parts[1])-1]+parts[0]+".pdf"; })() : "diarias.pdf";
  abrirBlob(doc, nome);
}

async function exportPDFFechamento(diarias, movimentos) {
  const jsPDF = await loadJsPDF();
  const doc = new jsPDF({ unit:"mm", format:"a4" });
  const W=210, M=14; let y=0;
  const INK=[9,11,20], PUR=[14,165,233], MINT=[74,222,128], YEL=[250,204,21],
        GRAY=[100,116,139], WHITE=[255,255,255], SKY=[56,189,248];

  const chk=(n=10)=>{ if(y+n>282){doc.addPage();doc.setFillColor(...INK);doc.rect(0,0,W,10,"F");y=18;} };
  const sec=(txt,cor)=>{ chk(14);doc.setFillColor(...cor);doc.rect(M,y,W-M*2,8,"F");doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...WHITE);doc.text(txt,M+4,y+5.5);y+=12; };

  const totD    = diarias.filter(function(d){return d.status!=="cancelado";}).reduce(function(s,d){return s+calcTotal(d);},0);
  const totA    = movimentos.filter(function(m){return m.tipo==="adiantamento";}).reduce(function(s,m){return s+(m.valor||0);},0);
  const saldo   = totD - totA;
  const dPend   = diarias.filter(function(d){return d.status==="pendente";});
  const dPagas  = diarias.filter(function(d){return d.status==="pago";});
  const totAlim = diarias.reduce(function(s,d){return s+(d.almoco||0)+(d.janta||0);},0);
  const totPend = dPend.reduce(function(s,d){return s+calcTotal(d);},0);
  const totPago = dPagas.reduce(function(s,d){return s+calcTotal(d);},0);

  doc.setFillColor(...INK);doc.rect(0,0,W,32,"F");
  doc.setFillColor(...MINT);doc.rect(0,0,5,32,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(18);doc.setTextColor(...WHITE);doc.text("FECHAMENTO",M+7,15);
  doc.setFont("helvetica","normal");doc.setFontSize(10);doc.setTextColor(...MINT);doc.text("Resumo completo de diárias e adiantamentos",M+7,24);
  doc.setFontSize(8);doc.setTextColor(...GRAY);doc.text("Gerado: "+new Date().toLocaleDateString("pt-BR"),W-M,24,{align:"right"});
  y=40;

  const cw=(W-M*2-8)/3;
  [[fm(totD),"Total em Diárias",SKY],[fm(totA),"Adiantamentos",YEL],[fm(saldo),"A Receber",saldo>0?MINT:YEL]].forEach(function(item,i){
    const val=item[0],lbl=item[1],cor=item[2];
    const x=M+i*(cw+4);
    doc.setFillColor(18,20,36);doc.roundedRect(x,y,cw,22,2,2,"F");
    doc.setFillColor(...cor);doc.rect(x,y,3,22,"F");
    doc.setFont("helvetica","normal");doc.setFontSize(7);doc.setTextColor(...GRAY);doc.text(lbl.toUpperCase(),x+6,y+8);
    doc.setFont("helvetica","bold");doc.setFontSize(11);doc.setTextColor(...WHITE);doc.text(val,x+6,y+17);
  });y+=28;

  sec("RESUMO GERAL",PUR);
  const rows=[
    ["Total em Diárias",fm(totD),WHITE],
    ["  Alimentação inclusa",fm(totAlim),[167,139,250]],
    ["Adiantamentos recebidos ("+movimentos.filter(function(m){return m.tipo==="adiantamento";}).length+"x)",fm(totA),SKY],
    ["Diárias já pagas ("+dPagas.length+")",fm(totPago),MINT],
    ["Diárias pendentes ("+dPend.length+")",fm(totPend),YEL],
  ];
  rows.forEach(function(r,i){
    chk(8);doc.setFillColor(14,16,28);doc.rect(M,y,W-M*2,8,"F");
    doc.setFont("helvetica","normal");doc.setFontSize(9);doc.setTextColor(...GRAY);doc.text(r[0],M+4,y+5.5);
    doc.setFont("helvetica","bold");doc.setTextColor(...r[2]);doc.text(r[1],W-M,y+5.5,{align:"right"});
    if(i<rows.length-1){doc.setDrawColor(20,22,38);doc.line(M,y+8,W-M,y+8);}
    y+=8;
  });
  y+=4;
  chk(18);doc.setFillColor(saldo>0?6:40,saldo>0?40:35,saldo>0?20:8);doc.roundedRect(M,y,W-M*2,16,3,3,"F");
  doc.setFillColor(...(saldo>0?MINT:YEL));doc.rect(M,y,5,16,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(10);doc.setTextColor(...WHITE);doc.text("VALOR FINAL A RECEBER",M+10,y+7);
  doc.setFontSize(14);doc.setTextColor(...(saldo>0?MINT:YEL));doc.text(fm(saldo),W-M,y+11,{align:"right"});
  y+=22;

  if(dPend.length>0){
    sec("DIÁRIAS PENDENTES",[180,130,0]);
    const cp=[M,M+28,M+84,M+124,M+148,M+172];
    doc.setFillColor(18,20,36);doc.rect(M,y,W-M*2,7,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(7.5);doc.setTextColor(...GRAY);
    ["Data","Evento","Tipo","Diária","Alim.","Total"].forEach(function(h,i){doc.text(h,cp[i],y+5);});y+=8;
    dPend.slice().sort(function(a,b){return a.data.localeCompare(b.data);}).forEach(function(d,i){
      chk(8);doc.setFillColor(i%2===0?14:11,i%2===0?16:14,i%2===0?30:24);doc.rect(M,y-1,W-M*2,7,"F");
      const tipo=((TIPOS.find(function(t){return t.tipo===d.tipoDiaria;})||{}).label||"-").replace("Diária de ","");
      const alim=(d.almoco||0)+(d.janta||0);
      const tot=calcTotal(d);
      doc.setFont("helvetica","normal");doc.setFontSize(8);doc.setTextColor(...WHITE);
      doc.text(fd(d.data),cp[0],y+4);
      doc.text((d.evento||"-").slice(0,18)+(d.dobra?" ⚡":""),cp[1],y+4);
      doc.text(tipo,cp[2],y+4);
      doc.text(fm(d.valorDiaria||0),cp[3],y+4);
      doc.setTextColor(...[167,139,250]);doc.text(alim>0?fm(alim):"-",cp[4],y+4);
      doc.setTextColor(...YEL);doc.text(fm(tot),cp[5],y+4);
      y+=7;
    });
    y+=2;doc.setFillColor(180,130,0);doc.rect(M,y,W-M*2,8,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...INK);
    doc.text("SUBTOTAL ("+dPend.length+" diárias)",M+4,y+5.5);doc.text(fm(totPend),W-M,y+5.5,{align:"right"});y+=14;
  }

  if(dPagas.length>0){
    sec("DIÁRIAS PAGAS",MINT);
    const cp=[M,M+28,M+84,M+124,M+148,M+172];
    doc.setFillColor(18,20,36);doc.rect(M,y,W-M*2,7,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(7.5);doc.setTextColor(...GRAY);
    ["Data","Evento","Tipo","Diária","Alim.","Total"].forEach(function(h,i){doc.text(h,cp[i],y+5);});y+=8;
    dPagas.slice().sort(function(a,b){return a.data.localeCompare(b.data);}).forEach(function(d,i){
      chk(8);doc.setFillColor(i%2===0?14:11,i%2===0?16:14,i%2===0?30:24);doc.rect(M,y-1,W-M*2,7,"F");
      const tipo=((TIPOS.find(function(t){return t.tipo===d.tipoDiaria;})||{}).label||"-").replace("Diária de ","");
      const alim=(d.almoco||0)+(d.janta||0);
      const tot=calcTotal(d);
      doc.setFont("helvetica","normal");doc.setFontSize(8);doc.setTextColor(...GRAY);
      doc.text(fd(d.data),cp[0],y+4);doc.text((d.evento||"-").slice(0,18),cp[1],y+4);doc.text(tipo,cp[2],y+4);doc.text(fm(d.valorDiaria||0),cp[3],y+4);
      doc.setTextColor(...[167,139,250]);doc.text(alim>0?fm(alim):"-",cp[4],y+4);
      doc.setTextColor(...MINT);doc.text(fm(tot),cp[5],y+4);y+=7;
    });
    y+=2;doc.setFillColor(...MINT);doc.rect(M,y,W-M*2,8,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...INK);
    doc.text("SUBTOTAL ("+dPagas.length+" diárias)",M+4,y+5.5);doc.text(fm(totPago),W-M,y+5.5,{align:"right"});y+=14;
  }

  const adiantos=movimentos.filter(function(m){return m.tipo==="adiantamento";});
  if(adiantos.length>0){
    chk(20);sec("ADIANTAMENTOS RECEBIDOS",SKY);
    const ma=[M,M+50,M+110];
    doc.setFillColor(18,20,36);doc.rect(M,y,W-M*2,7,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(7.5);doc.setTextColor(...GRAY);
    ["Data","Observação","Valor"].forEach(function(h,i){doc.text(h,ma[i],y+5);});y+=8;
    adiantos.slice().sort(function(a,b){return a.data.localeCompare(b.data);}).forEach(function(m,i){
      chk(8);doc.setFillColor(i%2===0?14:11,i%2===0?16:14,i%2===0?30:24);doc.rect(M,y-1,W-M*2,7,"F");
      doc.setFont("helvetica","normal");doc.setFontSize(8);doc.setTextColor(...WHITE);
      doc.text(fd(m.data),ma[0],y+4);doc.text((m.obs||"—").slice(0,30),ma[1],y+4);
      doc.setTextColor(...SKY);doc.text(fm(m.valor),ma[2],y+4);y+=7;
    });
    y+=2;doc.setFillColor(...SKY);doc.rect(M,y,W-M*2,8,"F");
    doc.setFont("helvetica","bold");doc.setFontSize(9);doc.setTextColor(...INK);
    doc.text("TOTAL ADIANTADO",M+4,y+5.5);doc.text(fm(totA),W-M,y+5.5,{align:"right"});y+=14;
  }

  chk(26);doc.setFillColor(saldo>0?6:40,saldo>0?40:35,saldo>0?20:8);doc.roundedRect(M,y,W-M*2,22,3,3,"F");
  doc.setFillColor(...(saldo>0?MINT:YEL));doc.rect(M,y,5,22,"F");
  doc.setFont("helvetica","bold");doc.setFontSize(11);doc.setTextColor(...WHITE);doc.text("VALOR FINAL A RECEBER",M+10,y+9);
  doc.setFontSize(16);doc.setTextColor(...(saldo>0?MINT:YEL));doc.text(fm(saldo),M+10,y+18);
  doc.setFontSize(8);doc.setTextColor(...GRAY);
  doc.text(diarias.length+" diárias  ·  "+fm(totA)+" adiantado  ·  total "+fm(totD),W-M,y+18,{align:"right"});

  const pages=doc.getNumberOfPages();
  for(let i=1;i<=pages;i++){doc.setPage(i);doc.setFillColor(...INK);doc.rect(0,288,W,9,"F");doc.setFont("helvetica","normal");doc.setFontSize(7);doc.setTextColor(...GRAY);doc.text("Fechamento · Relatório de Diárias",M,294);doc.text("Pág "+i+"/"+pages,W-M,294,{align:"right"});}

  const hoje2=new Date().toLocaleDateString("pt-BR").split("/").join("-");
  abrirBlob(doc, "fechamento_"+hoje2+".pdf");
}

export default function App() {
  const [diarias,    setDiarias]    = useState(()=>{try{return JSON.parse(localStorage.getItem(STORAGE_KEY)||"[]")}catch{return[]}});
  const [movimentos, setMovimentos] = useState(()=>{try{return JSON.parse(localStorage.getItem(STORAGE_KEY+"-mov")||"[]")}catch{return[]}});
  const [aba,        setAba]        = useState("diarias");
  const [tela,       setTela]       = useState("lista");
  const [editId,     setEditId]     = useState(null);
  const [filtroMes,  setFiltroMes]  = useState("");
  const [filtroSt,   setFiltroSt]   = useState("");
  const [busca,      setBusca]      = useState("");
  const [pdfMes,     setPdfMes]     = useState("");
  const [gerando,    setGerando]    = useState(false);
  const [gerandoF,   setGerandoF]   = useState(false);
  const [toast,      setToast]      = useState(null);
  const [confirmId,  setConfirmId]  = useState(null);
  const [form,  setForm]  = useState({data:hoje(),evento:"",tipoDiaria:"",valorDiaria:"",dobra:false,almoco:"",janta:"",obs:""});
  const [mForm, setMForm] = useState({tipo:"adiantamento",valor:"",data:hoje(),obs:""});
  const [mErro, setMErro] = useState("");

  useEffect(()=>{localStorage.setItem(STORAGE_KEY,JSON.stringify(diarias));},[diarias]);
  useEffect(()=>{localStorage.setItem(STORAGE_KEY+"-mov",JSON.stringify(movimentos));},[movimentos]);

  useEffect(()=>{
    const el=document.createElement("style");
    el.textContent="@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');" +
      "*{box-sizing:border-box;-webkit-tap-highlight-color:transparent;margin:0;padding:0;}" +
      "body{background:#040D1A;}" +
      "input,select,textarea,button{font-family:'Inter',sans-serif;}" +
      "input[type=date]::-webkit-calendar-picker-indicator{filter:invert(.4) sepia(1) saturate(2) hue-rotate(200deg);}" +
      "select option{background:#12141F;}" +
      "::-webkit-scrollbar{width:3px;}::-webkit-scrollbar-thumb{background:#1E2030;border-radius:2px;}" +
      ".press{transition:transform .12s ease,opacity .12s ease;cursor:pointer;}.press:active{transform:scale(.97);opacity:.85;}" +
      ".fadein{animation:fi .3s ease;}@keyframes fi{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}";
    document.head.appendChild(el);
    return()=>el.remove();
  },[]);

  const showToast=(msg,cor="#22D3EE")=>{setToast({msg,cor});setTimeout(()=>setToast(null),2400);};

  const totD  = diarias.filter(d=>d.status!=="cancelado").reduce((s,d)=>s+calcTotal(d),0);
  const totA  = movimentos.filter(m=>m.tipo==="adiantamento").reduce((s,m)=>s+(m.valor||0),0);
  const totF  = movimentos.filter(m=>m.tipo==="fechamento").reduce((s,m)=>s+(m.valor||0),0);
  const saldo = totD-(totA+totF);

  const dFilt=diarias.filter(d=>{
    if(filtroMes&&d.data?.slice(0,7)!==filtroMes)return false;
    if(filtroSt&&d.status!==filtroSt)return false;
    if(busca&&!d.evento?.toLowerCase().includes(busca.toLowerCase()))return false;
    return true;
  }).sort((a,b)=>b.data.localeCompare(a.data));

  const meses=[...new Set(diarias.map(d=>d.data?.slice(0,7)))].filter(Boolean).sort().reverse();

  const abrirForm=(d=null)=>{
    if(d){setForm({data:d.data,evento:d.evento||"",tipoDiaria:d.tipoDiaria||"",valorDiaria:d.valorDiaria||"",dobra:d.dobra||false,almoco:d.almoco||"",janta:d.janta||"",obs:d.obs||""});setEditId(d.id);}
    else{setForm({data:hoje(),evento:"",tipoDiaria:"",valorDiaria:"",dobra:false,almoco:"",janta:"",obs:""});setEditId(null);}
    setTela("form");
  };
  const salvarDiaria=()=>{
    if(!form.evento||!form.tipoDiaria){alert("Preencha Evento e Tipo.");return;}
    if(editId)setDiarias(d=>d.map(x=>x.id===editId?{...form,id:editId}:x));
    else setDiarias(d=>[...d,{...form,id:uid()}]);
    setTela("lista");showToast(editId?"Diária atualizada":"Diária registrada");
  };
  const marcarPago=(id)=>{setDiarias(d=>d.map(x=>x.id===id?{...x,status:"pago"}:x));showToast("Marcada como paga");};
  const voltarPendente=(id)=>{setDiarias(d=>d.map(x=>x.id===id?{...x,status:"pendente"}:x));showToast("Voltou para pendente","#FACC15");};
  const excluirDiaria=(id)=>{setDiarias(d=>d.filter(x=>x.id!==id));setConfirmId(null);if(tela==="form")setTela("lista");showToast("Diária removida","#F87171");};
  const salvarMov=()=>{
    const v=parseFloat(mForm.valor);
    if(!v||v<=0){setMErro("Informe um valor válido.");return;}
    setMovimentos(m=>[...m,{...mForm,valor:v,id:uid()}]);
    setMForm(f=>({...f,valor:"",obs:""}));setMErro("");
    showToast(mForm.tipo==="adiantamento"?"Adiantamento registrado":"Registrado");
  };
  const excluirMov=(id)=>{setMovimentos(m=>m.filter(x=>x.id!==id));showToast("Removido","#F87171");};
  const handlePDF=async()=>{setGerando(true);try{await exportPDF(diarias,movimentos,pdfMes);}catch(e){alert("Erro: "+e.message);}finally{setGerando(false);setTela("lista");}};
  const handlePDFF=async()=>{setGerandoF(true);try{await exportPDFFechamento(diarias,movimentos);}catch(e){alert("Erro: "+e.message);}finally{setGerandoF(false);}};

  const bg="  #09090F", surface="#071628", border="rgba(14,165,233,.12)", muted="#3D5A73", txt="#E8F4FD", acent="#0EA5E9", acent2="#38BDF8";
  const inp={background:"#0A1E35",border:"1px solid rgba(14,165,233,.12)",borderRadius:10,color:txt,padding:"13px 15px",fontSize:14,outline:"none",width:"100%",fontFamily:"'Inter',sans-serif"};
  const lbl={color:muted,fontSize:10,fontWeight:600,textTransform:"uppercase",letterSpacing:"1px",display:"block",marginBottom:7};
  const card={background:"#071628",border:"1px solid rgba(14,165,233,.12)",borderRadius:16};
  const HDR={display:"flex",alignItems:"center",justifyContent:"space-between",padding:"18px 20px 14px",position:"sticky",top:0,background:"rgba(4,13,26,.95)",backdropFilter:"blur(20px)",zIndex:10,borderBottom:"1px solid rgba(14,165,233,.12)"};

  const BtnPrimary=({children,onClick,style={},disabled=false})=>(
    <button className="press" onClick={onClick} disabled={disabled}
      style={{background:"linear-gradient(135deg,#0EA5E9,#0284C7)",border:"none",borderRadius:12,color:"#fff",fontWeight:700,fontSize:14,padding:"14px 22px",boxShadow:"0 4px 20px rgba(14,165,233,.4)",opacity:disabled?.55:1,fontFamily:"'Inter',sans-serif",...style}}>
      {children}
    </button>
  );

  const Modal=()=>{
    if(!confirmId)return null;
    const d=diarias.find(x=>x.id===confirmId);
    return(
      <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,.85)",zIndex:60,display:"flex",alignItems:"flex-end",justifyContent:"center",backdropFilter:"blur(6px)"}}>
        <div className="fadein" style={{...card,width:"100%",maxWidth:520,borderRadius:"20px 20px 0 0",padding:24,borderBottom:"none"}}>
          <div style={{width:36,height:4,background:border,borderRadius:2,margin:"0 auto 20px"}}/>
          <div style={{fontSize:11,fontWeight:700,color:"#F87171",textTransform:"uppercase",letterSpacing:1,marginBottom:8}}>Confirmar exclusão</div>
          <div style={{fontSize:17,fontWeight:600,color:txt,marginBottom:4}}>{d?.evento}</div>
          <div style={{fontSize:13,color:muted,marginBottom:24}}>{fd(d?.data)} · {fm(calcTotal(d||{}))}</div>
          <div style={{display:"flex",gap:10}}>
            <button className="press" style={{flex:1,padding:"13px",background:"#0A1E35",border:"1px solid rgba(14,165,233,.12)",borderRadius:12,color:muted,fontWeight:600,fontSize:14,fontFamily:"'Inter',sans-serif"}} onClick={()=>setConfirmId(null)}>Cancelar</button>
            <button className="press" style={{flex:1,padding:"13px",background:"rgba(248,113,113,.12)",border:"1px solid rgba(248,113,113,.2)",borderRadius:12,color:"#F87171",fontWeight:700,fontSize:14,fontFamily:"'Inter',sans-serif"}} onClick={()=>excluirDiaria(confirmId)}>Excluir</button>
          </div>
        </div>
      </div>
    );
  };

  // ── TELA PDF ─────────────────────────────────────────────────────────────
  if(tela==="pdf") return(
    <div style={{minHeight:"100vh",background:"#040D1A",color:txt,fontFamily:"'Inter',sans-serif",maxWidth:520,margin:"0 auto"}}>
      <div style={HDR}>
        <button className="press" onClick={()=>setTela("lista")} style={{background:"transparent",border:"1px solid rgba(14,165,233,.12)",color:muted,fontSize:13,fontWeight:500,padding:"8px 16px",borderRadius:20,fontFamily:"'Inter',sans-serif"}}>← Voltar</button>
        <span style={{fontSize:16,fontWeight:700,letterSpacing:"-.03em"}}>Exportar Relatório</span>
        <div style={{width:72}}/>
      </div>
      <div style={{padding:"24px 20px 100px",display:"flex",flexDirection:"column",gap:16}}>
        <div style={{textAlign:"center",paddingBottom:8}}>
          <div style={{width:64,height:64,background:"linear-gradient(135deg,#0EA5E9,#0369A1)",borderRadius:18,display:"flex",alignItems:"center",justifyContent:"center",fontSize:28,margin:"0 auto 14px",boxShadow:"0 8px 28px rgba(14,165,233,.35)"}}>📊</div>
          <div style={{fontSize:20,fontWeight:800,color:txt,letterSpacing:"-.04em"}}>Relatório em PDF</div>
          <div style={{color:muted,fontSize:13,marginTop:6}}>Exporta todas as diárias e movimentos</div>
        </div>
        <div style={{...card,padding:18}}>
          <label style={lbl}>Filtrar por período</label>
          <select style={inp} value={pdfMes} onChange={e=>setPdfMes(e.target.value)}>
            <option value="">Todos os períodos</option>
            {meses.map(m=>{const[y,mo]=m.split("-");return <option key={m} value={m}>{MESES_FULL[parseInt(mo)-1]} {y}</option>;})}
          </select>
        </div>
        <div style={{...card,padding:18}}>
          <div style={{fontSize:11,fontWeight:700,color:muted,textTransform:"uppercase",letterSpacing:1,marginBottom:14}}>Resumo</div>
          {[[diarias.filter(d=>!pdfMes||d.data?.slice(0,7)===pdfMes).length+" registros","Diárias",acent2],[fm(totD),"Total em Diárias",txt],[fm(totA),"Adiantamentos","#38BDF8"],[fm(saldo),"Saldo Pendente",saldo>0?"#FACC15":"#22D3EE"]].map(([v,l,c])=>(
            <div key={l} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"10px 0",borderBottom:"1px solid rgba(14,165,233,.07)"}}>
              <span style={{fontSize:13,color:muted,fontWeight:500}}>{l}</span>
              <span style={{fontSize:13,fontWeight:700,color:c}}>{v}</span>
            </div>
          ))}
        </div>
        <BtnPrimary onClick={handlePDF} disabled={gerando} style={{width:"100%",padding:15,fontSize:15}}>
          {gerando?"Gerando PDF…":"↓  Baixar PDF"}
        </BtnPrimary>
      </div>
    </div>
  );

  // ── TELA FORM ────────────────────────────────────────────────────────────
  if(tela==="form") return(
    <div style={{minHeight:"100vh",background:"#040D1A",color:txt,fontFamily:"'Inter',sans-serif",maxWidth:520,margin:"0 auto"}}>
      <div style={HDR}>
        <button className="press" onClick={()=>setTela("lista")} style={{background:"transparent",border:"1px solid rgba(14,165,233,.12)",color:muted,fontSize:13,fontWeight:500,padding:"8px 16px",borderRadius:20,fontFamily:"'Inter',sans-serif"}}>← Voltar</button>
        <span style={{fontSize:16,fontWeight:700,letterSpacing:"-.03em"}}>{editId?"Editar Diária":"Nova Diária"}</span>
        <div style={{width:72}}/>
      </div>
      <div style={{padding:"24px 20px 120px",display:"flex",flexDirection:"column",gap:14}} className="fadein">
        <div style={{...card,padding:18}}>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:14,marginBottom:14}}>
            <div><label style={lbl}>Data</label><input type="date" style={inp} value={form.data} onChange={e=>setForm(f=>({...f,data:e.target.value}))}/></div>
            <div>
              <label style={lbl}>Dia</label>
              <div style={{...inp,color:acent2,fontWeight:600,background:"rgba(14,165,233,.06)",border:"1px solid rgba(14,165,233,.15)"}}>
                {form.data?new Date(form.data+"T12:00").toLocaleDateString("pt-BR",{weekday:"short"}).replace(".","").replace(/^\w/,c=>c.toUpperCase()):"—"}
              </div>
            </div>
          </div>
          <div>
            <label style={lbl}>Evento *</label>
            <input style={inp} placeholder="Ex: Show do João, Casamento Silva…" value={form.evento} onChange={e=>setForm(f=>({...f,evento:e.target.value}))}/>
          </div>
        </div>

        <div style={{...card,padding:18}}>
          <label style={lbl}>Tipo de Diária *</label>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginTop:10}}>
            {TIPOS.map(op=>{
              const sel=form.tipoDiaria===op.tipo;
              return(
                <button key={op.tipo} className="press" type="button"
                  onClick={()=>setForm(f=>({...f,tipoDiaria:op.tipo,valorDiaria:f.dobra?op.valor*2:op.valor}))}
                  style={{background:sel?"rgba(14,165,233,.15)":"#0A1E35",border:"1.5px solid "+(sel?op.acento+"66":"rgba(14,165,233,.12)"),borderRadius:14,padding:"18px 12px",textAlign:"center",cursor:"pointer"}}>
                  <div style={{fontSize:28,marginBottom:8}}>{op.emoji}</div>
                  <div style={{fontSize:12,fontWeight:600,color:sel?txt:muted,marginBottom:6}}>{op.label}</div>
                  <div style={{fontSize:20,fontWeight:800,color:sel?op.acento:"#0D2440"}}>{fm(form.dobra?op.valor*2:op.valor)}</div>
                </button>
              );
            })}
          </div>
        </div>

        <div style={{...card,padding:18}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
            <div>
              <div style={{fontSize:14,fontWeight:700,color:txt}}>Dobra</div>
              <div style={{fontSize:11,color:muted,marginTop:2}}>Dia dobrado? O valor dobra automaticamente</div>
            </div>
            <button type="button" className="press"
              onClick={()=>setForm(f=>{
                const nd=!f.dobra;
                const ti=TIPOS.find(t=>t.tipo===f.tipoDiaria);
                return {...f,dobra:nd,valorDiaria:ti?(nd?ti.valor*2:ti.valor):f.valorDiaria};
              })}
              style={{width:52,height:28,borderRadius:14,border:"none",cursor:"pointer",position:"relative",background:form.dobra?"linear-gradient(135deg,#0EA5E9,#0284C7)":"#0D2440",flexShrink:0,boxShadow:form.dobra?"0 2px 10px rgba(14,165,233,.4)":"none"}}>
              <div style={{position:"absolute",top:3,left:form.dobra?26:3,width:22,height:22,borderRadius:"50%",background:form.dobra?"#fff":"#4B5568",transition:"left .2s"}}/>
            </button>
          </div>
          {form.dobra&&form.tipoDiaria&&(
            <div style={{marginTop:12,padding:"10px 14px",background:"rgba(14,165,233,.08)",border:"1px solid rgba(14,165,233,.2)",borderRadius:10,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <span style={{fontSize:12,color:muted}}>⚡ Dobra ativa</span>
              <span style={{fontSize:16,fontWeight:800,color:acent2}}>{fm(form.valorDiaria)}</span>
            </div>
          )}
        </div>

        <div style={{...card,padding:18}}>
          <label style={lbl}>Alimentação (opcional)</label>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12,marginTop:2}}>
            <div>
              <label style={{...lbl,color:"#38BDF8",marginBottom:5}}>🍽 Almoço (R$)</label>
              <input type="number" style={{...inp,color:"#38BDF8"}} placeholder="0,00" value={form.almoco} onChange={e=>setForm(f=>({...f,almoco:parseFloat(e.target.value)||""}))}/>
            </div>
            <div>
              <label style={{...lbl,color:acent2,marginBottom:5}}>🌙 Janta (R$)</label>
              <input type="number" style={{...inp,color:acent2}} placeholder="0,00" value={form.janta} onChange={e=>setForm(f=>({...f,janta:parseFloat(e.target.value)||""}))}/>
            </div>
          </div>
          {(form.almoco||form.janta)&&(
            <div style={{marginTop:12,padding:"10px 14px",background:"rgba(56,189,248,.06)",border:"1px solid rgba(56,189,248,.12)",borderRadius:10,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <span style={{fontSize:12,color:muted}}>{form.dobra?"⚡ Total c/ dobra":"Total c/ alimentação"}</span>
              <span style={{fontSize:16,fontWeight:800,color:"#22D3EE"}}>{fm((form.valorDiaria||0)+(form.almoco||0)+(form.janta||0))}</span>
            </div>
          )}
        </div>

        <div style={{...card,padding:18}}>
          <label style={lbl}>Observações</label>
          <textarea style={{...inp,minHeight:76,resize:"vertical",lineHeight:1.6}} placeholder="Notas adicionais…" value={form.obs} onChange={e=>setForm(f=>({...f,obs:e.target.value}))}/>
        </div>

        <div style={{display:"flex",gap:10}}>
          <BtnPrimary onClick={salvarDiaria} style={{flex:1,padding:15}}>{editId?"Salvar alterações":"Registrar diária"}</BtnPrimary>
          {editId&&<button className="press" style={{padding:"15px 18px",background:"rgba(248,113,113,.08)",border:"1px solid rgba(248,113,113,.18)",borderRadius:12,color:"#F87171",fontWeight:700,fontSize:18}} onClick={()=>setConfirmId(editId)}>🗑</button>}
        </div>
      </div>
      <Modal/>
    </div>
  );

  // ── ABA DIÁRIAS ──────────────────────────────────────────────────────────
  const renderDiarias=()=>(
    <div style={{paddingBottom:100}} className="fadein">
      <div style={{margin:"20px 20px 0",borderRadius:20,overflow:"hidden",position:"relative",background:"linear-gradient(145deg,#040D1A,#071E38)",border:"1px solid rgba(14,165,233,.2)"}}>
        <div style={{position:"absolute",top:-60,right:-60,width:200,height:200,background:"radial-gradient(circle,rgba(14,165,233,.08) 0%,transparent 65%)",pointerEvents:"none"}}/>
        <div style={{padding:"24px 22px 20px",position:"relative"}}>
          <div style={{fontSize:11,fontWeight:600,color:"rgba(56,189,248,.5)",textTransform:"uppercase",letterSpacing:"1.2px",marginBottom:8}}>Saldo Pendente</div>
          <div style={{fontSize:38,fontWeight:800,color:saldo>0?"#FACC15":"#22D3EE",letterSpacing:"-.04em",lineHeight:1}}>{fm(saldo)}</div>
          <div style={{display:"flex",gap:0,marginTop:20,paddingTop:16,borderTop:"1px solid rgba(14,165,233,.12)"}}>
            {[["Total",fm(totD),txt],["Recebido",fm(totA+totF),"#22D3EE"],["Diárias",""+diarias.length,acent2]].map(([l,v,c],i)=>(
              <div key={l} style={{flex:1,paddingLeft:i>0?16:0,borderLeft:i>0?"1px solid rgba(14,165,233,.12)":"none",marginLeft:i>0?16:0}}>
                <div style={{fontSize:10,color:muted,fontWeight:600,textTransform:"uppercase",letterSpacing:"1px"}}>{l}</div>
                <div style={{fontSize:14,fontWeight:700,color:c,marginTop:4}}>{v}</div>
              </div>
            ))}
          </div>
        </div>
      </div>

      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,padding:"12px 20px 4px"}}>
        {[{emoji:"🚶",label:"Rua",tipo:"rua",acento:"#38BDF8"},{emoji:"🏭",label:"Depósito",tipo:"deposito",acento:"#38BDF8"}].map(c=>{
          const n=dFilt.filter(d=>d.tipoDiaria===c.tipo).length;
          const v=dFilt.filter(d=>d.tipoDiaria===c.tipo).reduce((s,d)=>s+calcTotal(d),0);
          return(
            <div key={c.tipo} style={{...card,padding:"14px 16px",display:"flex",alignItems:"center",gap:12}}>
              <div style={{width:40,height:40,background:c.acento+"12",border:"1px solid "+c.acento+"25",borderRadius:12,display:"flex",alignItems:"center",justifyContent:"center",fontSize:18,flexShrink:0}}>{c.emoji}</div>
              <div>
                <div style={{fontSize:10,fontWeight:600,color:muted,textTransform:"uppercase",letterSpacing:"1px"}}>{c.label}</div>
                <div style={{fontSize:18,fontWeight:800,color:c.acento,marginTop:2}}>{n} <span style={{fontSize:11,fontWeight:500,color:muted}}>dias</span></div>
                <div style={{fontSize:11,color:muted,marginTop:1}}>{fm(v)}</div>
              </div>
            </div>
          );
        })}
      </div>

      <div style={{padding:"8px 20px",display:"flex",gap:8,flexWrap:"wrap"}}>
        <input style={{...inp,flex:1,minWidth:120,fontSize:13,padding:"10px 13px"}} placeholder="Buscar evento…" value={busca} onChange={e=>setBusca(e.target.value)}/>
        <select style={{...inp,flex:"0 0 auto",fontSize:12,padding:"10px 11px",color:filtroMes?txt:muted}} value={filtroMes} onChange={e=>setFiltroMes(e.target.value)}>
          <option value="">Mês</option>
          {meses.map(m=>{const[y,mo]=m.split("-");return <option key={m} value={m}>{MESES[parseInt(mo)-1]}/{y}</option>;})}
        </select>
        <select style={{...inp,flex:"0 0 auto",fontSize:12,padding:"10px 11px",color:filtroSt?txt:muted}} value={filtroSt} onChange={e=>setFiltroSt(e.target.value)}>
          <option value="">Status</option>
          <option value="pendente">Pendente</option>
          <option value="pago">Pago</option>
          <option value="cancelado">Cancelado</option>
        </select>
      </div>

      <div style={{padding:"6px 20px"}}>
        {dFilt.length===0?(
          <div style={{textAlign:"center",padding:"56px 20px"}}>
            <div style={{fontSize:48,marginBottom:16}}>📋</div>
            <div style={{fontSize:17,fontWeight:700,color:"#0D2440",marginBottom:8}}>Nenhuma diária</div>
            <BtnPrimary onClick={()=>abrirForm()} style={{padding:"13px 28px"}}>+ Nova Diária</BtnPrimary>
          </div>
        ):dFilt.map(d=>{
          const sc=SC[d.status]||SC.pendente;
          const ti=TIPOS.find(t=>t.tipo===d.tipoDiaria);
          const tot=calcTotal(d);
          return(
            <div key={d.id} style={{...card,marginBottom:10,overflow:"hidden"}}>
              <div className="press" style={{padding:"16px 16px 13px"}} onClick={()=>abrirForm(d)}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start"}}>
                  <div style={{flex:1,paddingRight:14,minWidth:0}}>
                    <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:8,flexWrap:"wrap"}}>
                      <span style={{fontSize:12,color:muted,fontWeight:500}}>{fd(d.data)}</span>
                      <button className="press"
                        onClick={e=>{e.stopPropagation();d.status==="pendente"?marcarPago(d.id):voltarPendente(d.id);}}
                        style={{display:"flex",alignItems:"center",gap:6,background:sc.bg,border:"1px solid "+sc.cor+"40",borderRadius:20,padding:"3px 10px 3px 6px",cursor:"pointer"}}>
                        <div style={{width:18,height:18,borderRadius:"50%",background:d.status==="pendente"?"rgba(250,204,21,.15)":"rgba(34,211,238,.2)",border:"1.5px solid "+sc.cor,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}>
                          {d.status==="pago"&&<span style={{fontSize:10,color:"#22D3EE",fontWeight:800,lineHeight:1}}>✓</span>}
                        </div>
                        <span style={{fontSize:11,fontWeight:700,color:sc.cor}}>{sc.label}</span>
                      </button>
                      {d.dobra&&<span style={{fontSize:10,fontWeight:700,borderRadius:6,padding:"2px 7px",background:"rgba(14,165,233,.15)",color:acent2}}>⚡ Dobra</span>}
                    </div>
                    <div style={{fontSize:16,fontWeight:700,color:txt,lineHeight:1.3}}>{d.evento}</div>
                    {ti&&<div style={{fontSize:12,color:muted,marginTop:5}}>{ti.emoji} {ti.label}</div>}
                    {(d.almoco||d.janta)&&(
                      <div style={{display:"flex",gap:10,marginTop:4}}>
                        {d.almoco&&<span style={{fontSize:11,color:"#38BDF8",fontWeight:600}}>🍽 {fm(d.almoco)}</span>}
                        {d.janta&&<span style={{fontSize:11,color:acent2,fontWeight:600}}>🌙 {fm(d.janta)}</span>}
                      </div>
                    )}
                    {d.obs&&<div style={{fontSize:12,color:"#1E2A3A",marginTop:4,fontStyle:"italic"}}>"{d.obs}"</div>}
                  </div>
                  <div style={{textAlign:"right",flexShrink:0}}>
                    <div style={{fontSize:19,fontWeight:800,color:"#22D3EE"}}>{fm(d.valorDiaria||0)}</div>
                    {(d.almoco||d.janta)&&<div style={{fontSize:13,fontWeight:700,color:txt,marginTop:3,borderTop:"1px solid rgba(14,165,233,.12)",paddingTop:3}}>= {fm(tot)}</div>}
                  </div>
                </div>
              </div>
              <div style={{borderTop:"1px solid rgba(14,165,233,.07)"}}>
                <button className="press" onClick={()=>setConfirmId(d.id)}
                  style={{width:"100%",background:"rgba(248,113,113,.04)",border:"none",color:"#F87171",padding:"11px 0",fontWeight:600,fontSize:12,display:"flex",alignItems:"center",justifyContent:"center",gap:6,fontFamily:"'Inter',sans-serif"}}>
                  🗑 Excluir
                </button>
              </div>
            </div>
          );
        })}
      </div>
      <Modal/>
    </div>
  );

  // ── ABA ADIANTAMENTO ─────────────────────────────────────────────────────
  const renderAdiant=()=>{
    const movs=[...movimentos].filter(m=>m.tipo==="adiantamento").sort((a,b)=>b.data.localeCompare(a.data));
    return(
      <div style={{paddingBottom:100}} className="fadein">
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,padding:"20px 20px 8px"}}>
          {[["Saldo Pendente",fm(saldo),"#FACC15"],["Adiantamentos",fm(totA),"#38BDF8"]].map(([l,v,c])=>(
            <div key={l} style={{...card,padding:"14px 14px"}}>
              <div style={{fontSize:9,color:muted,fontWeight:700,textTransform:"uppercase",letterSpacing:"1px",marginBottom:6}}>{l}</div>
              <div style={{fontSize:15,fontWeight:800,color:c}}>{v}</div>
            </div>
          ))}
        </div>
        <div style={{...card,margin:"0 20px",padding:18}}>
          <div style={{display:"flex",alignItems:"center",gap:14,marginBottom:18}}>
            <div style={{width:44,height:44,background:"linear-gradient(135deg,#0C4A6E,#0369A1)",borderRadius:13,display:"flex",alignItems:"center",justifyContent:"center",fontSize:20,flexShrink:0,boxShadow:"0 4px 16px rgba(56,189,248,.25)"}}>💵</div>
            <div>
              <div style={{fontSize:16,fontWeight:700,color:txt}}>Novo Adiantamento</div>
              <div style={{fontSize:12,color:muted,marginTop:2}}>Registrar recebimento parcial</div>
            </div>
          </div>
          <div style={{background:"rgba(56,189,248,.08)",border:"1px solid rgba(56,189,248,.18)",borderRadius:10,padding:"12px 14px",marginBottom:16,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <span style={{fontSize:12,color:muted,fontWeight:500}}>Saldo disponível</span>
            <span style={{fontSize:17,fontWeight:800,color:"#FACC15"}}>{fm(saldo)}</span>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12,marginBottom:12}}>
            <div><label style={lbl}>Data</label><input type="date" style={inp} value={mForm.data} onChange={e=>setMForm(f=>({...f,data:e.target.value}))}/></div>
            <div><label style={lbl}>Valor (R$)</label><input type="number" style={{...inp,fontWeight:800,fontSize:17,color:"#38BDF8"}} placeholder="0,00" value={mForm.valor} onChange={e=>{setMErro("");setMForm(f=>({...f,valor:e.target.value}));}}/></div>
          </div>
          <div style={{marginBottom:16}}>
            <label style={lbl}>Observação</label>
            <input style={inp} placeholder="Pix, dinheiro em mãos…" value={mForm.obs} onChange={e=>setMForm(f=>({...f,obs:e.target.value}))}/>
          </div>
          {mErro&&<div style={{color:"#F87171",fontSize:13,padding:"10px 14px",background:"rgba(248,113,113,.08)",border:"1px solid rgba(248,113,113,.18)",borderRadius:10,marginBottom:14,fontWeight:500}}>{mErro}</div>}
          <button className="press" style={{width:"100%",padding:14,background:"linear-gradient(135deg,#0C4A6E,#0369A1)",border:"none",borderRadius:12,color:"#fff",fontWeight:700,fontSize:14,boxShadow:"0 4px 18px rgba(56,189,248,.2)",fontFamily:"'Inter',sans-serif"}}
            onClick={()=>{setMForm(f=>({...f,tipo:"adiantamento"}));salvarMov();}}>
            Registrar Adiantamento
          </button>
        </div>
        <div style={{padding:"18px 20px"}}>
          <div style={{fontSize:11,fontWeight:700,color:muted,textTransform:"uppercase",letterSpacing:"1px",marginBottom:12}}>Histórico de Adiantamentos</div>
          {movs.length===0?(
            <div style={{textAlign:"center",padding:"32px 0",color:"#0D2440",fontSize:13,fontWeight:500}}>Nenhum adiantamento ainda</div>
          ):movs.map(m=>(
            <div key={m.id} style={{...card,marginBottom:10,overflow:"hidden"}}>
              <div style={{padding:"13px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                <div>
                  <div style={{fontSize:13,fontWeight:700,color:"#38BDF8",marginBottom:4}}>💵  Adiantamento</div>
                  <div style={{fontSize:12,color:muted}}>{fd(m.data)}{m.obs?<span style={{color:"#252A3A"}}> · {m.obs}</span>:""}</div>
                </div>
                <div style={{fontSize:17,fontWeight:800,color:"#22D3EE"}}>{fm(m.valor)}</div>
              </div>
              <div style={{borderTop:"1px solid rgba(14,165,233,.07)"}}>
                <button className="press" onClick={()=>excluirMov(m.id)} style={{width:"100%",background:"rgba(248,113,113,.04)",border:"none",color:"#F87171",padding:"9px",fontWeight:600,fontSize:12,fontFamily:"'Inter',sans-serif"}}>
                  🗑  Excluir
                </button>
              </div>
            </div>
          ))}
        </div>
      </div>
    );
  };

  // ── ABA FECHAMENTO ────────────────────────────────────────────────────────
  const renderFechamento=()=>{
    const dPagas=diarias.filter(d=>d.status==="pago");
    const dPend=diarias.filter(d=>d.status==="pendente");
    const totAlim=diarias.reduce((s,d)=>s+(d.almoco||0)+(d.janta||0),0);
    const totPend=dPend.reduce((s,d)=>s+calcTotal(d),0);
    const totPago=dPagas.reduce((s,d)=>s+calcTotal(d),0);
    return(
      <div style={{paddingBottom:100}} className="fadein">
        <div style={{margin:"20px 20px 0",borderRadius:20,overflow:"hidden",position:"relative",background:"linear-gradient(145deg,#040D1A,#071E38)",border:"1px solid rgba(34,211,238,.2)"}}>
          <div style={{position:"absolute",top:-50,right:-50,width:180,height:180,background:"radial-gradient(circle,rgba(34,211,238,.08) 0%,transparent 65%)",pointerEvents:"none"}}/>
          <div style={{padding:"22px 22px 18px",position:"relative"}}>
            <div style={{fontSize:11,fontWeight:600,color:"rgba(34,211,238,.5)",textTransform:"uppercase",letterSpacing:"1.2px",marginBottom:8}}>A Receber</div>
            <div style={{fontSize:36,fontWeight:800,color:saldo>0?"#22D3EE":"#FACC15",letterSpacing:"-.04em",lineHeight:1}}>{fm(saldo)}</div>
            <div style={{fontSize:12,color:muted,marginTop:6}}>Após descontar {fm(totA)} em adiantamentos</div>
            <button className="press" onClick={handlePDFF} disabled={gerandoF}
              style={{marginTop:16,width:"100%",background:"linear-gradient(135deg,#064E3B,#047857)",border:"1px solid rgba(34,211,238,.25)",borderRadius:10,color:"#22D3EE",padding:"13px",fontWeight:700,fontSize:14,display:"flex",alignItems:"center",justifyContent:"center",gap:8,opacity:gerandoF?.6:1,boxShadow:"0 4px 16px rgba(34,211,238,.15)",fontFamily:"'Inter',sans-serif"}}>
              {gerandoF?"⏳  Gerando PDF…":"📄  Exportar Fechamento em PDF"}
            </button>
          </div>
        </div>

        <div style={{padding:"14px 20px 0"}}>
          <div style={{fontSize:11,fontWeight:700,color:muted,textTransform:"uppercase",letterSpacing:"1px",marginBottom:10}}>Resumo Geral</div>
          <div style={{...card,overflow:"hidden"}}>
            {[
              {l:"Total em Diárias",v:fm(totD),c:txt,icon:"📋"},
              {l:"Alimentação inclusa",v:fm(totAlim),c:acent2,icon:"🍽"},
              {l:"Adiantamentos recebidos",v:fm(totA),c:"#38BDF8",icon:"💵"},
              {l:"Diárias já pagas ("+dPagas.length+")",v:fm(totPago),c:"#22D3EE",icon:"✅"},
              {l:"Diárias pendentes ("+dPend.length+")",v:fm(totPend),c:"#FACC15",icon:"⏳"},
            ].map(({l,v,c,icon},i,arr)=>(
              <div key={l} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"13px 16px",borderBottom:i<arr.length-1?"1px solid rgba(14,165,233,.07)":"none"}}>
                <div style={{display:"flex",alignItems:"center",gap:8}}>
                  <span style={{fontSize:15}}>{icon}</span>
                  <span style={{fontSize:13,color:muted,fontWeight:500}}>{l}</span>
                </div>
                <span style={{fontSize:13,fontWeight:700,color:c}}>{v}</span>
              </div>
            ))}
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"14px 16px",background:"rgba(34,211,238,.06)",borderTop:"1px solid rgba(34,211,238,.12)"}}>
              <div style={{display:"flex",alignItems:"center",gap:8}}>
                <span style={{fontSize:15}}>💰</span>
                <span style={{fontSize:14,fontWeight:700,color:txt}}>Saldo a Receber</span>
              </div>
              <span style={{fontSize:17,fontWeight:800,color:saldo>0?"#22D3EE":"#FACC15"}}>{fm(saldo)}</span>
            </div>
          </div>
        </div>

        {dPend.length>0&&(
          <div style={{padding:"16px 20px 0"}}>
            <div style={{fontSize:11,fontWeight:700,color:"#FACC15",textTransform:"uppercase",letterSpacing:"1px",marginBottom:10}}>⏳ Diárias Pendentes</div>
            {dPend.slice().sort((a,b)=>b.data.localeCompare(a.data)).map(d=>{
              const ti=TIPOS.find(t=>t.tipo===d.tipoDiaria);
              const tot=calcTotal(d);
              return(
                <div key={d.id} style={{...card,marginBottom:8,padding:"13px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                  <div>
                    <div style={{fontSize:13,fontWeight:700,color:txt,marginBottom:3}}>{d.evento}{d.dobra?<span style={{color:acent2,fontSize:11}}> ⚡</span>:""}</div>
                    <div style={{fontSize:11,color:muted}}>{fd(d.data)}{ti?" · "+ti.emoji+" "+ti.label:""}</div>
                    {(d.almoco||d.janta)&&<div style={{fontSize:11,color:acent2,marginTop:2}}>{d.almoco?"🍽 "+fm(d.almoco)+" ":""}{d.janta?"🌙 "+fm(d.janta):""}</div>}
                  </div>
                  <div style={{textAlign:"right"}}>
                    <div style={{fontSize:15,fontWeight:800,color:"#FACC15"}}>{fm(tot)}</div>
                    {(d.almoco||d.janta)&&<div style={{fontSize:10,color:muted,marginTop:1}}>diária: {fm(d.valorDiaria||0)}</div>}
                  </div>
                </div>
              );
            })}
          </div>
        )}

        {totA>0&&(
          <div style={{padding:"16px 20px 0"}}>
            <div style={{fontSize:11,fontWeight:700,color:"#38BDF8",textTransform:"uppercase",letterSpacing:"1px",marginBottom:10}}>💵 Adiantamentos Recebidos</div>
            {[...movimentos].filter(m=>m.tipo==="adiantamento").sort((a,b)=>b.data.localeCompare(a.data)).map(m=>(
              <div key={m.id} style={{...card,marginBottom:8,padding:"13px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                <div>
                  <div style={{fontSize:13,fontWeight:600,color:"#38BDF8",marginBottom:3}}>💵 Adiantamento</div>
                  <div style={{fontSize:11,color:muted}}>{fd(m.data)}{m.obs?" · "+m.obs:""}</div>
                </div>
                <div style={{fontSize:15,fontWeight:800,color:"#22D3EE"}}>{fm(m.valor)}</div>
              </div>
            ))}
          </div>
        )}

        <div style={{margin:"16px 20px 0",borderRadius:14,padding:"16px 18px",background:saldo>0?"rgba(34,211,238,.06)":"rgba(250,204,21,.06)",border:"1px solid "+(saldo>0?"rgba(34,211,238,.15)":"rgba(250,204,21,.15)")}}>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <div>
              <div style={{fontSize:11,fontWeight:700,color:muted,textTransform:"uppercase",letterSpacing:"1px"}}>Valor Final a Receber</div>
              <div style={{fontSize:12,color:muted,marginTop:3}}>{diarias.length} diárias · {fm(totA)} adiantado</div>
            </div>
            <div style={{fontSize:26,fontWeight:800,color:saldo>0?"#22D3EE":"#FACC15"}}>{fm(saldo)}</div>
          </div>
        </div>
      </div>
    );
  };

  return(
    <div style={{minHeight:"100vh",background:"#040D1A",color:txt,fontFamily:"'Inter',sans-serif",maxWidth:520,margin:"0 auto"}}>
      <div style={HDR}>
        <div>
          <div style={{fontSize:20,fontWeight:800,color:txt,letterSpacing:"-.04em"}}>
            {aba==="diarias"?"Diárias":aba==="adiantamento"?"Adiantamento":"Fechamento"}
          </div>
          <div style={{fontSize:11,color:muted,marginTop:2,fontWeight:500}}>
            {aba==="diarias"?"Controle de dias trabalhados":aba==="adiantamento"?"Recebimento parcial":"Resumo completo"}
          </div>
        </div>
        <div style={{display:"flex",gap:8,alignItems:"center"}}>
          <button className="press" style={{background:"transparent",border:"1px solid rgba(14,165,233,.12)",borderRadius:10,padding:"9px 12px",fontSize:17,color:muted,fontFamily:"'Inter',sans-serif"}} onClick={()=>setTela("pdf")}>📄</button>
          {aba==="diarias"&&<BtnPrimary onClick={()=>abrirForm()} style={{padding:"9px 18px",fontSize:13}}>+ Nova</BtnPrimary>}
        </div>
      </div>

      {aba==="diarias"      && renderDiarias()}
      {aba==="adiantamento" && renderAdiant()}
      {aba==="fechamento"   && renderFechamento()}

      <div style={{position:"fixed",bottom:0,left:"50%",transform:"translateX(-50%)",width:"100%",maxWidth:520,background:"rgba(4,13,26,.97)",borderTop:"1px solid rgba(14,165,233,.12)",display:"flex",zIndex:20,backdropFilter:"blur(20px)"}}>
        {[{id:"diarias",icon:"📋",label:"Diárias"},{id:"adiantamento",icon:"💵",label:"Adiantamento"},{id:"fechamento",icon:"🔒",label:"Fechamento"}].map(t=>{
          const on=aba===t.id;
          return(
            <button key={t.id} className="press" onClick={()=>setAba(t.id)}
              style={{flex:1,background:"none",border:"none",cursor:"pointer",padding:"11px 4px 10px",display:"flex",flexDirection:"column",alignItems:"center",gap:4,position:"relative",fontFamily:"'Inter',sans-serif"}}>
              {on&&<div style={{position:"absolute",top:0,left:"50%",transform:"translateX(-50%)",width:20,height:2,background:"#38BDF8",borderRadius:"0 0 3px 3px"}}/>}
              <div style={{fontSize:18,filter:on?"none":"grayscale(100%) opacity(.28)"}}>{t.icon}</div>
              <div style={{fontSize:10,fontWeight:on?700:500,color:on?acent2:muted}}>{t.label}</div>
            </button>
          );
        })}
      </div>

      {aba==="diarias"&&(
        <button className="press" onClick={()=>abrirForm()}
          style={{position:"fixed",bottom:80,right:"calc(50vw - 240px)",zIndex:30,
            width:58,height:58,borderRadius:"50%",border:"none",cursor:"pointer",
            background:"linear-gradient(135deg,#0EA5E9,#0284C7)",
            boxShadow:"0 6px 28px rgba(14,165,233,.55), 0 2px 8px rgba(0,0,0,.4)",
            display:"flex",alignItems:"center",justifyContent:"center",fontSize:26,
            color:"#fff",fontWeight:700}}>
          +
        </button>
      )}

      {toast&&(
        <div style={{position:"fixed",bottom:86,left:"50%",transform:"translateX(-50%)",background:"#071628",border:"1px solid "+toast.cor+"30",color:toast.cor,padding:"10px 20px",borderRadius:50,fontSize:13,fontWeight:600,boxShadow:"0 8px 32px rgba(0,0,0,.6)",zIndex:100,whiteSpace:"nowrap",backdropFilter:"blur(12px)",fontFamily:"'Inter',sans-serif"}}>
          {toast.msg}
        </div>
      )}
    </div>
  );
}

