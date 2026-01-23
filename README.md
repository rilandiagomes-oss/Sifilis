<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Apoio à Indicação – Sífilis</title>
  <style>
    body { font-family: Arial; background: #f4f6f8; padding: 20px; }
    .box { background: #fff; padding: 20px; border-radius: 8px; max-width: 700px; margin: auto; box-shadow: 0 2px 6px rgba(0,0,0,0.1);}
    label { display: block; margin-top: 12px; font-weight: bold; }
    select, input[type="date"] { padding: 6px; width: 100%; margin-top: 5px; border-radius: 4px; border: 1px solid #ccc;}
    button { margin-top: 15px; padding: 10px; width: 100%; cursor: pointer; border: none; border-radius: 5px; font-size: 16px; }
    .info-btn { background-color: #1976d2; color: white; }
    .avaliar-btn { background-color: #2e7d32; color: white; }
    .pasta-btn { background-color: #6d4c41; color: white; }
    hr { margin: 20px 0; border: none; border-top: 1px solid #ccc; }
    .alerta { margin-top: 15px; font-weight: bold; padding: 10px; border-radius: 5px; white-space: pre-line; }
    .alerta.positivo { background-color: #d4edda; color: #155724; }
    .alerta.negativo { background-color: #f8d7da; color: #721c24; }
    .tratamento, .notificacao { margin-top: 15px; background-color: #e3f2fd; padding: 10px; border-radius: 5px; white-space: pre-line; }
    .subtitulo { font-weight: bold; margin-top: 10px; }
    .radio-group label { display: block; margin-top: 5px; font-weight: normal; }
  </style>
</head>
<body>

<div class="box">
  <h2>Apoio à decisão – Sífilis</h2>

  <button class="info-btn" onclick="mostrarDefinicoes()">📘 Definições dos casos de Sífilis</button>
  <button class="info-btn" onclick="mostrarTeste()">🧪 Sobre o Teste Rápido para Sífilis</button>
  <button class="pasta-btn" onclick="abrirPasta()">📂 Acessar Notificações / Normas Técnicas</button>

  <hr>

  <label>Gestante? <span style="color:red">*</span></label>
  <select id="gestante" onchange="limparResultados()">
    <option value="" selected>Selecione</option>
    <option value="nao">Não</option>
    <option value="sim">Sim</option>
  </select>

  <label>Classificação da sífilis <span style="color:red">*</span></label>
  <select id="tipo" onchange="limparResultados()">
    <option value="" selected>Selecione</option>
    <option value="recente">Recente (primária, secundária, latente recente ≤1 ano)</option>
    <option value="tardia">Tardia (latente tardia >1 ano, latente ignorada, terciária)</option>
  </select>

  <label>Data da 1ª dose <span style="color:red">*</span></label>
  <input type="date" id="dose1">

  <!-- População geral - radio buttons -->
  <div class="radio-group" id="criteriosPopGeral" style="display:none;">
    <span class="subtitulo">População geral – Sífilis adquirida <span style="color:red">*</span>:</span>
    <label><input type="radio" name="pop_situacao" value="sit1"> Situação 1: Assintomático, apenas teste treponêmico reagente (teste rápido). </label>
    <label><input type="radio" name="pop_situacao" value="sit2"> Situação 2: Assintomático ou sintomático, teste não treponêmico reagente + treponêmico reagente. </label>
    <label><input type="radio" name="pop_situacao" value="sit3"> Situação 3: Sintomático, pelo menos um teste reagente. </label>
  </div>

  <button class="avaliar-btn" onclick="avaliar()">Avaliar caso</button>

  <!-- Resultados -->
  <div id="resultado" class="alerta"></div>
  <div id="tratamento" class="tratamento"></div>
  <div id="notificacao" class="notificacao"></div>
</div>

<script>
// Funções utilitárias
function diasEntre(d1,d2){ return Math.round((d2-d1)/(1000*60*60*24)); }
function formatarData(d){ return `${String(d.getDate()).padStart(2,'0')}/${String(d.getMonth()+1).padStart(2,'0')}/${d.getFullYear()}`; }

// Limpa resultados e mostra critérios
function limparResultados(){
  document.getElementById("resultado").innerText="";
  document.getElementById("resultado").className="alerta";
  document.getElementById("tratamento").innerText="";
  document.getElementById("notificacao").innerText="";
  const gestante=document.getElementById("gestante").value;
  document.getElementById("criteriosPopGeral").style.display=gestante==="nao"?"block":"none";

  // Desmarca radios da população geral ao mudar seleção
  const radios=document.getElementsByName("pop_situacao");
  radios.forEach(r=>r.checked=false);
}

// Abrir pasta do Drive
function abrirPasta(){
  window.open("https://drive.google.com/drive/folders/10TiK57aXQk62LYshuoSNQFnEuIuZK7kg?usp=sharing","_blank");
}

// Avaliar tratamento e notificação
function avaliar(){
  const gestante=document.getElementById("gestante").value;
  const tipo=document.getElementById("tipo").value;
  const d1Input=document.getElementById("dose1").value;

  // Validação obrigatória
  if(!gestante || !tipo || !d1Input){
    alert("⚠️ Todos os campos obrigatórios devem ser preenchidos!");
    return;
  }

  // Para população geral, obrigar selecionar 1 radio
  if(gestante==="nao"){
    const radios=document.getElementsByName("pop_situacao");
    let selecionado=null;
    for(const r of radios){ if(r.checked){ selecionado=r.value; break; } }
    if(!selecionado){
      alert("⚠️ Selecione uma situação para a população geral.");
      return;
    }
  }

  const d1=new Date(d1Input);
  const hoje=new Date();
  if(d1>hoje){ alert("Data da 1ª dose futura inválida."); return; }

  // Tratamento
  let esquema="";
  let msg="Tratamento adequado ✔️";
  let obs="";

  if(tipo==="recente"){
    esquema=`💊 Tratamento proposto:\n1ª dose: Benzilpenicilina benzatina 2,4 milhões UI IM – ${formatarData(d1)}`;
  } else{
    const d2=new Date(d1); d2.setDate(d2.getDate()+7);
    const d3=new Date(d2); d3.setDate(d3.getDate()+7);
    esquema=`💊 Tratamento proposto:\n1ª dose: Benzilpenicilina benzatina 2,4 milhões UI IM – ${formatarData(d1)}\n`+
             `2ª dose: Benzilpenicilina benzatina 2,4 milhões UI IM – ${formatarData(d2)}\n`+
             `3ª dose: Benzilpenicilina benzatina 2,4 milhões UI IM – ${formatarData(d3)}\n`+
             `Dose total: 7,2 milhões UI IM`;
    if(gestante==="sim"){
      obs="⚠️ Intervalo recomendado para gestante: 7 dias entre doses. Em caso de atraso, caso ultrapasse mais de 9 dias, a gestante deve ser retratada.";
    }
  }

  document.getElementById("resultado").innerText=msg+(obs? "\n"+obs:"");
  document.getElementById("resultado").className="alerta positivo";
  document.getElementById("tratamento").innerText=esquema;

  // Notificação
  let notificacao="";
  if(gestante==="sim"){
    notificacao="📌 Notificação obrigatória: SIM\nTipo: Sífilis em gestante";
  } else{
    const radios=document.getElementsByName("pop_situacao");
    let selecionado=null;
    for(const r of radios){ if(r.checked){ selecionado=r.value; break; } }
    if(selecionado==="sit2" || selecionado==="sit3") notificacao="📌 Notificação obrigatória: SIM\nTipo: Sífilis adquirida (população geral)";
    else if(selecionado==="sit1") notificacao="📌 Notificação obrigatória: NÃO (aguarda VDRL)\nTipo: Sífilis adquirida (população geral)";
  }

  document.getElementById("notificacao").innerText=notificacao;
}

// Botões educativos
function mostrarDefinicoes(){ alert(
"SÍFILIS PRIMÁRIA:\nFerida geralmente única no local de entrada da bactéria.\nSurge entre 10 e 90 dias após o contágio.\n\n"+
"SÍFILIS SECUNDÁRIA:\nManchas no corpo, febre, ínguas, etc.\n\n"+
"SÍFILIS LATENTE:\nFase assintomática.\nLatente recente: ≤1 ano.\nLatente tardia: >1 ano.\n\n"+
"SÍFILIS TERCIÁRIA:\nLesões cutâneas, ósseas, cardiovasculares e neurológicas."
);}

function mostrarTeste(){ alert(
"TESTE RÁPIDO PARA SÍFILIS:\nSe reagente, confirmar com exame laboratorial.\nNo mesmo dia do início do tratamento, coletar sangue para monitoramento.\nPessoas tratadas podem manter teste reagente mesmo após cura."
);}
</script>

</body>
</html>
