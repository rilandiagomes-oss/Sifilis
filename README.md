<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Tratamento e notificação – Sífilis</title>
  <style>
    body { font-family: Arial; background: #f4f6f8; padding: 20px; }
    .box { background: #fff; padding: 20px; border-radius: 8px; max-width: 700px; margin: auto; box-shadow: 0 2px 6px rgba(0,0,0,0.1);}
    label { display: block; margin-top: 12px; font-weight: bold; }
    select { padding: 6px; width: 100%; margin-top: 5px; border-radius: 4px; border: 1px solid #ccc;}
    button { margin-top: 15px; padding: 10px; width: 100%; cursor: pointer; border: none; border-radius: 5px; font-size: 16px; }
    .info-btn { background-color: #1976d2; color: white; }
    .avaliar-btn { background-color: #2e7d32; color: white; }
    .pasta-btn { background-color: #6d4c41; color: white; }
    hr { margin: 20px 0; border: none; border-top: 1px solid #ccc; }
    .alerta { margin-top: 15px; font-weight: bold; padding: 10px; border-radius: 5px; white-space: pre-line; }
    .alerta.positivo { background-color: #d4edda; color: #155724; }
    .tratamento, .notificacao { margin-top: 15px; background-color: #e3f2fd; padding: 10px; border-radius: 5px; white-space: pre-line; }
    .subtitulo { font-weight: bold; margin-top: 10px; }
    .radio-group label { display: block; margin-top: 5px; font-weight: normal; }
  </style>
</head>
<body>

<div class="box">
  <h2>Apoio à decisão – Sífilis</h2>

  <button class="info-btn" onclick="mostrarDefinicoes()">📘 Definições</button>
  <button class="info-btn" onclick="mostrarTeste()">🧪 Teste Rápido</button>
  <button class="pasta-btn" onclick="abrirPasta()">📂 Notificações / Normas</button>

  <hr>

  <label>Situação do caso? <span style="color:red">*</span></label>
  <select id="gestante" onchange="limparResultados()">
    <option value="" selected>Selecione</option>
    <option value="nao">Pop Geral</option>
    <option value="sim">Gestante</option>
  </select>

  <label>Classificação da sífilis <span style="color:red">*</span></label>
  <select id="tipo" onchange="limparResultados()">
    <option value="" selected>Selecione</option>
    <option value="recente">Sífilis recente</option>
    <option value="tardia">Sífilis tardia</option>
  </select>

  <div class="radio-group" id="criteriosPopGeral" style="display:none;">
    <span class="subtitulo">Situação clínica <span style="color:red">*</span>:</span>
    <label><input type="radio" name="pop_situacao" value="sit1"> Assintomático, teste rápido reagente</label>
    <label><input type="radio" name="pop_situacao" value="sit2"> Assintomático, VDRL reagente</label>
    <label><input type="radio" name="pop_situacao" value="sit3"> Sintomático com teste reagente</label>
  </div>

  <button class="avaliar-btn" onclick="avaliar()">Avaliar caso</button>

  <div id="resultado" class="alerta"></div>
  <div id="tratamento" class="tratamento"></div>
  <div id="notificacao" class="notificacao"></div>
</div>

<script>
function formatarData(d){
  return `${String(d.getDate()).padStart(2,'0')}/${String(d.getMonth()+1).padStart(2,'0')}/${d.getFullYear()}`;
}

function limparResultados(){
  document.getElementById("resultado").innerText="";
  document.getElementById("tratamento").innerText="";
  document.getElementById("notificacao").innerText="";

  const gestante=document.getElementById("gestante").value;
  document.getElementById("criteriosPopGeral").style.display = gestante ? "block" : "none";

  document.getElementsByName("pop_situacao").forEach(r=>r.checked=false);
}

function abrirPasta(){
  window.open("https://drive.google.com/drive/folders/10TiK57aXQk62LYshuoSNQFnEuIuZK7kg?usp=sharing","_blank");
}

function avaliar(){
  const gestante=document.getElementById("gestante").value;
  const tipo=document.getElementById("tipo").value;

  if(!gestante || !tipo){
    alert("⚠️ Preencha todos os campos!");
    return;
  }

  let marcado=false;
  document.getElementsByName("pop_situacao").forEach(r=>{ if(r.checked) marcado=true; });

  if(!marcado){
    alert("⚠️ Selecione uma situação.");
    return;
  }

  const d1 = new Date();

  let esquema="";
  let obs="";

  if(tipo==="recente"){
    esquema=`💊 Tratamento:\n1ª dose: ${formatarData(d1)}`;
  } else{
    const d2=new Date(d1); d2.setDate(d2.getDate()+7);
    const d3=new Date(d2); d3.setDate(d3.getDate()+7);

    esquema=`💊 Tratamento:\n`+
             `1ª dose: ${formatarData(d1)}\n`+
             `2ª dose: ${formatarData(d2)}\n`+
             `3ª dose: ${formatarData(d3)}`;

    if(gestante==="sim"){
      obs="⚠️ Intervalo ideal: 7 dias. Se >9 dias, retratar.";
    }
  }

  document.getElementById("resultado").innerText="✔️ Conduta definida";
  document.getElementById("resultado").className="alerta positivo";

  document.getElementById("tratamento").innerText = esquema + (obs ? "\n\n" + obs : "");

  let notificacao="";

  if(gestante==="sim"){
    notificacao="📌 Notificação obrigatória: SIM\nTipo: Sífilis em gestante";
  } else{
    const sel=[...document.getElementsByName("pop_situacao")].find(r=>r.checked)?.value;

    if(sel==="sit2"){
      notificacao="📌 Notificação obrigatória: SIM\nCritério: VDRL reagente\nTipo: Sífilis adquirida";
    } 
    else if(sel==="sit3"){
      notificacao="📌 Notificação obrigatória: SIM\nCritério: Caso sintomático\nTipo: Sífilis adquirida";
    } 
    else{
      notificacao="📌 Notificação obrigatória: NÃO\nConduta: Realizar busca de histórico clínico e epidemiológico, com evidência para sífilis. Instituir tratamento conforme protocolo vigente. Aguardar resultado de VDRL para fins de notificação.\nTipo: Sífilis adquirida";
    }
  }

  document.getElementById("notificacao").innerText = notificacao;
}

// 🔵 DEFINIÇÕES COMPLETAS RESTAURADAS
function mostrarDefinicoes(){
  alert(
"SÍFILIS PRIMÁRIA:\nFerida geralmente única no local de entrada da bactéria.\n\n"+
"SÍFILIS SECUNDÁRIA:\nManchas no corpo, febre, ínguas.\n\n"+
"SÍFILIS LATENTE:\nAssintomática.\nLatente recente ≤1 ano.\nLatente tardia >1 ano.\n\n"+
"SÍFILIS TERCIÁRIA:\nLesões cutâneas, ósseas, cardiovasculares e neurológicas."
  );
}

// 🔵 TESTE RÁPIDO RESTAURADO
function mostrarTeste(){
  alert(
"TESTE RÁPIDO PARA SÍFILIS:\n\n"+
"Se reagente, confirmar com exame laboratorial.\n"+
"No mesmo dia do início do tratamento, coletar sangue para monitoramento.\n"+
"Pessoas tratadas podem manter teste reagente mesmo após cura."
  );
}
</script>

</body>
</html>
