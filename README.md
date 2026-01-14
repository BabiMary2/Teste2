<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Controle de Estoque</title>

<style>
body {
  font-family: Arial, sans-serif;
  background: #f4f4f4;
  padding: 20px;
}

h2 {
  background: #222;
  color: #fff;
  padding: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  margin-bottom: 20px;
}

th, td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: center;
}

input, select {
  padding: 6px;
  margin-bottom: 6px;
}

button {
  padding: 6px 12px;
  cursor: pointer;
  margin-bottom: 10px;
}

.disponivel { background: #c8f7c5; }
.esgotado { background: #f7c5c5; font-weight: bold; }

.periodo {
  background: #fff;
  padding: 10px;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>
</head>

<body>

<h2>Cadastrar Produto</h2>

<input id="nomeProduto" placeholder="Nome do produto">
<input id="qtdProduto" type="number" placeholder="Quantidade inicial">
<input id="precoProduto" type="number" step="0.01" placeholder="Preço unitário (R$)">
<button onclick="adicionarProduto()">Adicionar</button>

<h2>Estoque</h2>

<input
  id="pesquisaEstoque"
  placeholder="Pesquisar produto no estoque..."
  onkeyup="atualizarTabela()"
>

<button onclick="ordenarEstoque()">Ordenar estoque A–Z</button>

<table id="tabela">
<tr>
  <th>Produto</th>
  <th>Preço</th>
  <th>Qtd</th>
  <th>Status</th>
  <th>Retirar</th>
  <th>Quem comprou</th>
  <th>Ação</th>
</tr>
</table>

<h2>Histórico de Vendas</h2>

<div class="periodo">
  <strong>Período:</strong>

  <select id="mes" onchange="atualizarHistorico()">
    <option value="">Todos</option>
    <option value="1">Janeiro</option>
    <option value="2">Fevereiro</option>
    <option value="3">Março</option>
    <option value="4">Abril</option>
    <option value="5">Maio</option>
    <option value="6">Junho</option>
    <option value="7">Julho</option>
    <option value="8">Agosto</option>
    <option value="9">Setembro</option>
    <option value="10">Outubro</option>
    <option value="11">Novembro</option>
    <option value="12">Dezembro</option>
  </select>

  <input
    type="number"
    id="ano"
    placeholder="Ano"
    value="2026"
    onchange="atualizarHistorico()"
  >

  <input
    id="pesquisaComprador"
    placeholder="Pesquisar comprador..."
    onkeyup="atualizarHistorico()"
  >
</div>

<table id="historico">
<tr>
  <th>Produto</th>
  <th>Qtd</th>
  <th>Total</th>
  <th>Comprador</th>
  <th>Data</th>
  <th>Ação</th>
</tr>
</table>

<script>
let estoque = JSON.parse(localStorage.getItem("estoque")) || [];
let vendas = JSON.parse(localStorage.getItem("vendas")) || [];

estoque = estoque.map(p => ({
  nome: p.nome,
  qtd: p.qtd,
  preco: Number(p.preco) || 0
}));

function salvar() {
  localStorage.setItem("estoque", JSON.stringify(estoque));
  localStorage.setItem("vendas", JSON.stringify(vendas));
}

function ordenarEstoque() {
  estoque.sort((a, b) => a.nome.localeCompare(b.nome));
  salvar();
  atualizarTabela();
}

function atualizarTabela() {
  const tabela = document.getElementById("tabela");
  const filtro = document.getElementById("pesquisaEstoque").value.toLowerCase();

  while (tabela.rows.length > 1) tabela.deleteRow(1);

  estoque.forEach((p, i) => {
    if (!p.nome.toLowerCase().includes(filtro)) return;

    const r = tabela.insertRow();
    r.className = p.qtd > 0 ? "disponivel" : "esgotado";

    r.innerHTML = `
      <td>${p.nome}</td>
      <td>R$ ${p.preco.toFixed(2)}</td>
      <td>${p.qtd}</td>
      <td>${p.qtd > 0 ? "Disponível" : "ESGOTADO"}</td>
      <td>
        <input type="number" min="1" value="1" id="retirar${i}">
      </td>
      <td>
        <input placeholder="Nome do comprador" id="comprador${i}">
        <button onclick="retirar(${i})">Vender</button>
      </td>
      <td>
        <button onclick="excluirProduto(${i})">Excluir</button>
      </td>
    `;
  });
}

function atualizarHistorico() {
  const tabela = document.getElementById("historico");
  const compradorFiltro = document.getElementById("pesquisaComprador").value.toLowerCase();
  const mes = document.getElementById("mes").value;
  const ano = document.getElementById("ano").value;

  while (tabela.rows.length > 1) tabela.deleteRow(1);

  vendas.forEach((v, i) => {
    if (compradorFiltro && !v.comprador.toLowerCase().includes(compradorFiltro)) return;
    if (mes && v.mes != mes) return;
    if (ano && v.ano != ano) return;

    const r = tabela.insertRow();
    r.innerHTML = `
      <td>${v.produto}</td>
      <td>${v.qtd}</td>
      <td>R$ ${v.total.toFixed(2)}</td>
      <td>${v.comprador}</td>
      <td>${v.data}</td>
      <td>
        <button onclick="excluirVenda(${i})">Excluir</button>
      </td>
    `;
  });
}

function adicionarProduto() {
  const nome = nomeProduto.value.trim();
  const qtd = Number(qtdProduto.value);
  const preco = Number(precoProduto.value);

  if (!nome || qtd <= 0 || preco <= 0) {
    alert("Preencha corretamente");
    return;
  }

  estoque.push({ nome, qtd, preco });

  nomeProduto.value = "";
  qtdProduto.value = "";
  precoProduto.value = "";

  salvar();
  atualizarTabela();
}

function retirar(i) {
  const qtd = Number(document.getElementById("retirar" + i).value);
  const comprador = document.getElementById("comprador" + i).value.trim();

  if (qtd <= 0 || !comprador) {
    alert("Informe quantidade e comprador");
    return;
  }

  if (estoque[i].qtd < qtd) {
    alert("Quantidade insuficiente");
    return;
  }

  estoque[i].qtd -= qtd;

  const data = new Date();

  vendas.push({
    produto: estoque[i].nome,
    qtd,
    total: estoque[i].preco * qtd,
    comprador,
    data: data.toLocaleString("pt-BR"),
    mes: data.getMonth() + 1,
    ano: data.getFullYear()
  });

  salvar();
  atualizarTabela();
  atualizarHistorico();
}

function excluirVenda(i) {
  if (!confirm("Excluir esta venda?")) return;
  vendas.splice(i, 1);
  salvar();
  atualizarHistorico();
}

function excluirProduto(i) {
  if (!confirm("Excluir produto?")) return;
  estoque.splice(i, 1);
  salvar();
  atualizarTabela();
}

atualizarTabela();
atualizarHistorico();
</script>

</body>
</html>
