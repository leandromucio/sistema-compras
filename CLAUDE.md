# Sistema de Compras — Instruções para Claude Code

## O que é este projeto
Sistema web de gestão de compras de panelas importadas via freteiros.
Arquivo único: `index.html` — HTML + CSS + JavaScript puro, sem frameworks.
Banco de dados: Firebase Firestore (tempo real, multi-usuário).
Hospedagem: Netlify em https://genuine-lolly-5286ff.netlify.app

## Estrutura do arquivo index.html
- **Linhas 1-115**: HTML head, CSS completo, scripts Firebase
- **Linhas 116-230**: Firebase config, funções de storage (ss, saveLocal, getLocal), STATE, listeners
- **Linhas 231-400**: Nova Compra — cabeçalho do pedido e tabela de itens
- **Linhas 401-800**: Funções da tabela de itens (autocomplete SKU, cálculos, salvarPedido)
- **Linhas 800-1100**: Aba Pedidos (buildHistorico, buildPedidosTable, filtros, checkboxes)
- **Linhas 1100-1300**: Banco de Dados (comissões, freteiros, busca, edição em massa)
- **Linhas 1300-1500**: Importar, Usuários, printPedido
- **Linhas 1500+**: Firebase INIT com retry e sincronização

## Regras de negócio importantes

### Comissão por freteiro (padrão fixo)
```
Pecinha: 18%
Marquinho / Marquinho Angel: 20%
Lu Freteira: 16%
Renato: 15%
Vanessa: 15%
Rosane: 12%
```
- Ao selecionar freteiro, preenche comissão automaticamente
- Ao salvar pedido, faz UPSERT no banco de dados (SKU + freteiro → comissão%)
- Banco de dados tem prioridade sobre padrão do freteiro

### Cálculo de custo
```
Valor Unit (R$)  = U$ × Cotação
Frete UNT        = Valor Unit × (Comissão% / 100)
Custo Unit       = Valor Unit + Frete UNT
Custo Total      = Custo Unit × Quantidade
Custo s/ Frete   = U$ × Cotação × Quantidade
A pagar freteiro:
  - Nota Paga   → só frete (Frete UNT × Quantidade)
  - A Pagar     → Custo Total completo
```

### Firebase — cuidados críticos
- `save(key, val)` atualiza STATE + localStorage + Firestore
- Após salvar, ignora snapshots por 3 segundos (evita eco)
- No INIT: se localStorage tem mais registros que Firestore, usa local e sincroniza
- Escrita com retry (3 tentativas com delay crescente)
- Coleção Firestore: `sistema`, docs: `compras`, `comissoes`, `freteiros`, `users`

### Estrutura de um registro de compra
```js
{
  id: Date.now(),
  data: 'YYYY-MM-DD',
  pedido: '123',
  nota_status: 'a_pagar' | 'nota_paga',
  produto: 'AHGB10PA',       // SKU
  freteiro: 'Marquinho',
  preco_usd: 27,
  quantidade: 2,
  cotacao: 5.45,
  comissao_pct: 20,          // em % (não decimal)
  custo_unit: 195.42,
  custo_total: 390.84,
  valor_pagar_freteiro: 59.13,
  status_lancado: false,
  usuario: 'admin',
  origem: 'Manual'
}
```

### Estrutura do banco de comissões
```js
{ produto: 'AHGB10PA', freteiro: 'Marquinho', comissao: 20 }
// comissao em % (não decimal)
```

## Usuários padrão
- admin / admin123 (role: admin — vê tudo)
- usuario / 123456 (role: user — vê só próprias compras)

## Versão atual
v2025.03.30-r18 — exibida na tela de login

## Ao fazer alterações
1. Sempre editar o arquivo `index.html` na pasta
2. Manter a versão atualizada (rNN no final)
3. Nunca quebrar a estrutura do Firebase init
4. Testar cálculos de custo antes de salvar
5. Após editar: usuário faz Commit + Push no GitHub Desktop
