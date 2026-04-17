# Documentacao Tecnica Completa: CS Dashboard ROI

## Visao Geral do Projeto

Sistema de onboarding e dashboard para a **Mentoria ROI** (Grupo ROI - mentoria de vendas na Amazon Brasil).

**Dois arquivos HTML independentes que compartilham dados via localStorage:**
- `onboarding-mentoria-roi.html` (formulario de onboarding, 30 perguntas + LeadScore)
- `dashboard-onboarding-roi.html` (dashboard de metricas, funil, trending, analytics)

**Stack atual:** HTML/CSS/JS puro, zero dependencias, localStorage para persistencia. Pronto para migrar para Supabase.

---

## PARTE 1: FORMULARIO DE ONBOARDING

### 1.1 Arquitetura Geral

O formulario e um SPA (Single Page Application) com 34 telas navegaveis via transicao CSS. Cada tela e uma `<div class="screen">` posicionada absolutamente. A navegacao e sequencial com pulos condicionais.

**Constantes principais:**
```
TOTAL_Q = 30
SESSION_STORAGE_KEY = 'roi_sessions'
```

**CSS Variables (Design System):**
```
--yellow: #FF9900     (cor primaria/destaque)
--green: #16a34a      (botoes, sucesso)
--green-light: #4ade80
--green-dark: #14532d
--black: #080808      (fundo)
--card: #141414       (cards/blocos)
--card-hover: #1e1e1e
--border: #2a2a2a
--white: #ffffff
--gray: #888888
--gray-light: #bbbbbb
```

### 1.2 Fluxo Completo de Telas (SCREEN_ORDER)

```
screen-welcome          -> Tela inicial
screen-1                -> Q1: Dados pessoais (nome, whatsapp, email)
screen-2                -> Q2: Genero
screen-3                -> Q3: Faixa etaria
screen-4                -> Q4: Profissao
screen-5                -> Q5: Computador proprio
screen-6                -> Q6: Periodo do dia
screen-7                -> Q7: Loja ativa Amazon
screen-8                -> Q8: Outro marketplace
screen-9                -> Q9: Objetivo com Amazon
screen-10               -> Q10: Mineracao de produtos (CONDICIONAL)
screen-11               -> Q11: Primeira compra (CONDICIONAL)
screen-12               -> Q12: Oferta (CONDICIONAL)
screen-13               -> Q13: Primeira venda
screen-14               -> Q14: Investimento disponivel (SCORE)
screen-15               -> Q15: Reinvestimento 4 meses (SCORE)
screen-16               -> Q16: Horas/dia (SCORE)
screen-17               -> Q17: Conhecimento Amazon (SCORE)
screen-19               -> Q19: Habilidades PC (SCORE) [Q18 REMOVIDA]
screen-20               -> Q20: Meta faturamento 6 meses
screen-22               -> Q22: Intro CNPJ [Q21 REMOVIDA]
screen-23               -> Q23: Responsavel pela loja
screen-24               -> Q24: Compromisso (checkbox)
screen-25               -> Q25: Qtd socios
screen-26               -> Q26: Dados pessoais + docs (tela mega dinamica)
screen-cnpj-exclusivity -> CNPJ exclusivo p/ marketplaces?
screen-27               -> Q27: Razao Social
screen-28               -> Q28: Nome Fantasia
screen-29               -> Q29: Capital Social
screen-30               -> Q30: Aceite dos Termos
screen-loading          -> Tela de carregamento (5 etapas)
screen-result           -> Tela de sucesso
```

### 1.3 Logica de Navegacao Condicional

```javascript
function nextQ(qNum) {
  var next = qNum + 1;

  // Q18 removida (redundante com Q7)
  if (next === 18) next = 19;

  // Q21 removida
  if (next === 21) next = 22;

  // Q10/Q11/Q12 puladas se: sem loja Amazon E sem outro marketplace
  if (answers.q7 === 2 && answers.q8 !== 0 && (next === 10 || next === 11 || next === 12)) {
    next = 13;
  }

  // Apos Q29, vai p/ tela de exclusividade CNPJ (nao Q30 direto)
  if (qNum === 29) {
    // rota especial: savePessoas() -> screen-cnpj-exclusivity -> Q27
  }
}
```

**Matriz de decisao Q7 x Q8 para Q10/Q11/Q12:**

| Q7 (loja Amazon) | Q8 (outro MP) | Q10/Q11/Q12 |
|---|---|---|
| Nao tenho (2) | Sim (0) | MOSTRA com opcao extra "Sim, em outro marketplace" |
| Nao tenho (2) | Nao (1) | PULA direto pra Q13 |
| Sim (0 ou 1) | Sim (0) | MOSTRA com opcao extra "Sim, em outro marketplace" |
| Sim (0 ou 1) | Nao (1) | MOSTRA normalmente (3 opcoes) |

### 1.4 Todas as Perguntas, Opcoes e Indices

#### BLOCO PERFIL

**Q1: Dados pessoais** (screen-1)
- Campos: nome (texto), whatsapp (mascara tel), email (validacao regex)
- Validacao: nome nao vazio, whatsapp >= 10 digitos, email formato valido
- Dados salvos: `answers.name`, `answers.whatsapp`, `answers.email`

**Q2: Genero** (screen-2)
- idx 0: Masculino
- idx 1: Feminino
- idx 2: Prefiro nao informar

**Q3: Faixa etaria** (screen-3)
- idx 0: Menos de 18 anos
- idx 1: Entre 18 e 24 anos
- idx 2: Entre 25 e 34 anos
- idx 3: Entre 35 e 44 anos
- idx 4: Acima de 45 anos

**Q4: Profissao** (screen-4)
- idx 0: CLT
- idx 1: Autonomo
- idx 2: Empresario
- idx 3: Desempregado
- idx 4: Estudante
- idx 5: Servidor Publico
- idx 6: Outro

**Q5: Computador proprio** (screen-5)
- idx 0: Sim
- idx 1: Nao (mostra alerta recomendando investir)
- idx 2: Uso compartilhado

**Q6: Periodo de operacao** (screen-6)
- idx 0: Manha
- idx 1: Tarde
- idx 2: Noite
- idx 3: Variado
- idx 4: Nao definido

**Q7: Loja ativa Amazon** (screen-7) [CONDICIONA Q10/Q11/Q12]
- idx 0: Sim, ja vendo
- idx 1: Sim, mas ainda nao vendo
- idx 2: Nao tenho

**Q8: Outro marketplace** (screen-8) [CONDICIONA Q10/Q11/Q12]
- idx 0: Sim (abre campo extra "Qual?" salvo em answers.q8_qual)
- idx 1: Nao

**Q9: Objetivo com Amazon** (screen-9)
- idx 0: Renda extra
- idx 1: Aumentar renda
- idx 2: Viver da Amazon
- idx 3: Escalar negocio

#### BLOCO EXECUCAO ATUAL (CONDICIONAL)

**Q10: Mineracao de produtos na Amazon** (screen-10) [CONDICIONAL]
- idx 0: Nao comecei
- idx 1: Em andamento
- idx 2: Produtos validados
- idx 3: Sim, mas em outro marketplace (visivel apenas se Q8 === 0)

**Q11: Primeira compra na Amazon** (screen-11) [CONDICIONAL]
- idx 0: Nao comecei
- idx 1: Em negociacao
- idx 2: Ja comprei
- idx 3: Sim, mas em outro marketplace (visivel apenas se Q8 === 0)

**Q12: Oferta na Amazon** (screen-12) [CONDICIONAL]
- idx 0: Nao tenho
- idx 1: Em analise/envio
- idx 2: Ativa
- idx 3: Sim, mas em outro marketplace (visivel apenas se Q8 === 0)

**Q13: Primeira venda na Amazon** (screen-13)
- idx 0: Nao
- idx 1: Sim, poucas vendas
- idx 2: Sim, vendas consistentes

#### BLOCO LEADSCORE (Q14-Q19 pontuam)

**Q14: Investimento disponivel HOJE** (screen-14) - SCORE + OVERRIDES
- idx 0: Menos de R$2.000 -> 5 pts -> **TIER E AUTOMATICO**
- idx 1: R$2.000 a R$5.000 -> 15 pts -> **TIER E AUTOMATICO**
- idx 2: R$5.000 a R$10.000 -> 30 pts
- idx 3: R$10.000 a R$20.000 -> 40 pts
- idx 4: Acima de R$20.000 -> 50 pts -> **TIER A AUTOMATICO (score = 100)**

**Q15: Reinvestimento 4 meses** (screen-15) - SCORE
- idx 0: Nao pretendo reinvestir -> 0 pts
- idx 1: Ate R$1.000/mes -> 5 pts
- idx 2: R$1.000 a R$3.000/mes -> 10 pts
- idx 3: R$3.000 a R$5.000/mes -> 15 pts
- idx 4: Acima de R$5.000/mes -> 20 pts

**Q16: Horas/dia** (screen-16) - SCORE
- idx 0: Menos de 1h -> 5 pts
- idx 1: 1 a 2h -> 10 pts
- idx 2: 2 a 4h -> 20 pts
- idx 3: Mais de 4h -> 30 pts

**Q17: Conhecimento Amazon** (screen-17) - SCORE
- idx 0: Nenhum -> 0 pts
- idx 1: Basico -> 4 pts
- idx 2: Intermediario -> 8 pts
- idx 3: Avancado -> 12 pts

**Q19: Habilidades com computador** (screen-19) - SCORE
- idx 0: Nenhuma / muita dificuldade -> 0 pts
- idx 1: Basico -> 3 pts
- idx 2: Intermediario -> 5 pts
- idx 3: Avancado -> 8 pts

**Q20: Meta faturamento 6 meses** (screen-20)
- idx 0: Ate R$5.000
- idx 1: R$5.000 a R$20.000
- idx 2: R$20.000 a R$50.000
- idx 3: Acima de R$50.000

#### BLOCO CNPJ

**Q22: Intro CNPJ** (screen-22)
- Tela informativa com checklist de documentos necessarios
- So botao "Continuar"

**Q23: Responsavel pela loja** (screen-23)
- idx 0: Eu mesmo
- idx 1: Socio
- idx 2: Funcionario
- idx 3: Terceirizado

**Q24: Compromisso** (screen-24)
- Checkbox unico: "Eu me comprometo a aguardar o Grupo ROI abrir minha loja"
- Salva: answers.q24 = true/false

**Q25: Socios** (screen-25)
- idx 0: Apenas eu (sem socios) -> answers.q25 = 0, answers.q25_total = 1
- Opcao B: Sim, terei socios -> abre input numerico (1-10)
  - answers.q25 = N (qtd socios alem do responsavel)
  - answers.q25_total = N + 1

**Q26: Dados pessoais e documentos** (screen-26) - TELA DINAMICA
- Gera N blocos de pessoa (1 para responsavel, N para socios)
- Detalhes completos na secao 1.5

**CNPJ Exclusividade** (screen-cnpj-exclusivity) - ENTRE Q26 e Q27
- Pergunta: "O seu CNPJ sera exclusivo para vendas em marketplaces?"
- Opcao A: Sim, sera somente p/ marketplaces -> avanca p/ Q27
  - answers.cnpj_exclusivity = 'exclusivo_marketplaces'
  - answers.cnpj_wants_contact = null
- Opcao B: Nao, utilizarei para outras finalidades -> mostra caixa de contato
  - answers.cnpj_exclusivity = 'outras_finalidades'
  - Caixa mostra: "CNPJ para Amazon e gratuito. Outras finalidades = adicional pago"
  - Botao verde: "Sim, entre em contato comigo!" -> answers.cnpj_wants_contact = true
  - Link discreto: "Nao, utilizarei apenas para marketplaces" -> answers.cnpj_wants_contact = false

**Q27: Razao Social** (screen-27)
- Campo texto livre
- Observacao: "A Razao Social pode ser alterada adicionando ECOMMERCE, COMERCIO, VAREJO..."
- Alerta: "Proibido termos Amazon, Prime, Video, Kindle"
- Salva: answers.q27

**Q28: Nome Fantasia** (screen-28)
- Campo texto livre
- Observacao: "O nome fantasia pode ser alterado adicionando ECOMMERCE, COMERCIO, VAREJO..."
- Salva: answers.q28

**Q29: Capital Social** (screen-29)
- idx 0: R$ 5.000
- idx 1: R$ 7.000
- idx 2: R$ 10.000
- idx 3: + de R$ 10.000 -> abre campo custom (minimo R$ 10.001, mascara moeda)
  - answers.q29 = 3
  - answers.q29_custom = valor numerico

**Q30: Aceite dos Termos** (screen-30)
- 6 secoes informativas (beneficio CNPJ, envio Amazon, certificado digital, assessoria contabil, prazos, aceite)
- Dados pre-preenchidos do responsavel (nome, email, whatsapp, endereco)
- 1 checkbox obrigatorio: "Aceito os Termos de Adesao"
- Link discreto: "Acessar termo de adesao"
- Botao "Enviar" -> startLoading()

### 1.5 Tela Dinamica Q26 (buildPessoasForm)

Gera formulario para N pessoas (responsavel + socios). Campos por pessoa:

```
CAMPOS PARA TODOS:
- Nome completo (min 2 palavras)
- CPF (mascara, 11 digitos)
- RG (texto)
- Upload documento CNH/RG (file + camera, max 10MB, min 50KB img)
- Estado de nascimento (dropdown via API IBGE)
- Cidade de nascimento (dropdown cascata via API IBGE)
- CEP (mascara, 8 digitos, autofill via ViaCEP)
- Rua/Avenida (autofill, readonly quando CEP encontrado)
- Bairro (autofill, readonly quando CEP encontrado)
- Numero (texto, obrigatorio)
- Cidade e UF atuais (readonly, autofill)
- Complemento (opcional)
- Upload comprovante de endereco (file + camera)
- Data de nascimento (date picker)
- Estado civil (Solteiro/Casado/Divorciado/Outro)
  - Se Casado: Regime de casamento + Upload certidao de casamento
- WhatsApp (mascara tel)
- Servidor Publico? (Sim/Nao)
- Beneficio INSS? (Sim/Nao)
- Certificado Digital? (Sim/Nao)
- CNH ativa? (Sim/Nao)
- Autodeclaracao de raca (Branca/Preta/Parda/Amarela/Indigena)

CAMPOS APENAS PARA SOCIOS (i > 0):
- Profissao (texto)
- E-mail pessoal (email valido)

CAMPOS QUANDO TEM SOCIOS (numPessoas > 1):
- Percentual de participacao (1-100%, soma deve = 100%)
```

**Observacao especial (quando tem socios):**
- No card do responsavel: "O Socio Administrador sera, tambem, o Responsavel Legal perante a Receita Federal"
- No comprovante: "O comprovante de endereco deve ser referente ao endereco preenchido acima no formulario"

### 1.6 Formula do LeadScore

```javascript
// INVESTIMENTO (peso ~50%)
var p1 = scorePoints[14][answers.q14];  // max 50
var p2 = scorePoints[15][answers.q15];  // max 20
var investimento = (p1 * 0.7) + (p2 * 0.3);  // max 41

// TEMPO (peso ~30%)
var tempo = scorePoints[16][answers.q16];  // max 30

// CAPACIDADE (peso ~20%)
var c1 = scorePoints[17][answers.q17];  // max 12
var c3 = scorePoints[19][answers.q19];  // max 8
var capacidade = c1 + c3;  // max 20

// TOTAL
var score = investimento + tempo + capacidade;  // max 91
```

**Tabela de pontos:**
```
Q14 Investimento:    [5, 15, 30, 40, 50]   max 50
Q15 Reinvestimento:  [0, 5, 10, 15, 20]    max 20
Q16 Horas/dia:       [5, 10, 20, 30]       max 30
Q17 Conhecimento:    [0, 4, 8, 12]         max 12
Q19 Habilidades PC:  [0, 3, 5, 8]          max 8
```

**Classificacao por Tier:**
```
Score >= 85 -> Tier A
Score >= 70 -> Tier B
Score >= 50 -> Tier C
Score >= 30 -> Tier D
Score <  30 -> Tier E
```

**Overrides automaticos (ignoram o score calculado):**
- Q14 = 0 ou 1 (investimento < R$5.000) -> **Tier E forcado** (score mantido)
- Q14 = 4 (investimento > R$20.000) -> **Tier A forcado** (score = 100)

### 1.7 Estruturas de Dados no localStorage

#### `roi_submissions` (array de objetos)
```json
[
  {
    "timestamp": "2026-04-17T15:30:00.000Z",
    "answers": {
      "name": "Joao Silva",
      "whatsapp": "(11) 99999-8888",
      "email": "joao@teste.com",
      "q2": 0,
      "q3": 2,
      "q4": 1,
      "q5": 0,
      "q6": 2,
      "q7": 0,
      "q8": 0,
      "q8_qual": "Shopee",
      "q9": 1,
      "q10": 1,
      "q11": 0,
      "q12": 0,
      "q13": 0,
      "q14": 2,
      "q15": 2,
      "q16": 2,
      "q17": 2,
      "q19": 2,
      "q20": 1,
      "q22": "12.345.678/0001-89",
      "q23": 0,
      "q24": true,
      "q25": 0,
      "q25_total": 1,
      "q27": "Joao Silva ME",
      "q28": "JS Store",
      "q29": 0,
      "q29_custom": null,
      "cnpj_exclusivity": "exclusivo_marketplaces",
      "cnpj_wants_contact": null,
      "pessoas": [
        {
          "nome": "Joao Silva",
          "cpf": "123.456.789-00",
          "rg": "12.345.678-9",
          "uf_nascimento": "SP",
          "cidade_nascimento_nome": "Sao Paulo",
          "cidade_nascimento": "Sao Paulo - SP",
          "endereco_cep": "01310-100",
          "endereco_rua": "Avenida Paulista",
          "endereco_numero": "1000",
          "endereco_bairro": "Bela Vista",
          "endereco_cidade": "Sao Paulo",
          "endereco_uf": "SP",
          "endereco_complemento": "Sala 1",
          "endereco_completo": "Avenida Paulista, 1000 (Sala 1), Bela Vista, Sao Paulo - SP (01310-100)",
          "cep": "01310-100",
          "data_nascimento": "1990-01-15",
          "estado_civil": "Solteiro",
          "regime_casamento": "",
          "whatsapp": "(11) 99999-8888",
          "servidor_publico": "Nao",
          "beneficio_inss": "Nao",
          "certificado_digital": "Nao",
          "cnh_ativa": "Sim",
          "autodeclaracao_raca": "Parda"
        }
      ],
      "pessoas_files": {
        "p0_doc": {"name": "rg.jpg", "type": "image/jpeg", "size": 150000, "data": "data:image/jpeg;base64,..."},
        "p0_endereco": {"name": "comprovante.pdf", "type": "application/pdf", "size": 200000, "data": "data:application/pdf;base64,..."}
      }
    },
    "score": {
      "score": 57,
      "tier": "C",
      "override": false,
      "overrideReason": null
    }
  }
]
```

#### `roi_sessions` (array de objetos - tracking do funil)
```json
[
  {
    "id": "sess_1713369600000_abc12345",
    "startedAt": "2026-04-17T12:00:00.000Z",
    "lastActivityAt": "2026-04-17T12:15:30.000Z",
    "lastScreen": "screen-26",
    "lastScreenOrder": 25,
    "screensVisited": ["screen-welcome", "screen-1", "screen-2", "..."],
    "completed": false,
    "completedAt": null,
    "userAgent": "Mozilla/5.0...",
    "name": "Joao Silva",
    "email": "joao@teste.com",
    "whatsapp": "(11) 99999-8888"
  }
]
```

#### Caches de API (localStorage)
```
ibge_states        -> [{"sigla":"AC","nome":"Acre"}, ...]  (27 estados)
ibge_cities_SP     -> ["Adamantina", "Adolfo", ...]         (por UF)
ibge_cities_RJ     -> [...]
```

### 1.8 APIs Externas Utilizadas

| API | URL | Quando | Dados | Cache |
|---|---|---|---|---|
| IBGE Estados | `https://servicodados.ibge.gov.br/api/v1/localidades/estados?orderBy=nome` | Ao abrir Q26 | [{sigla, nome}] | `ibge_states` |
| IBGE Cidades | `https://servicodados.ibge.gov.br/api/v1/localidades/estados/{UF}/municipios?orderBy=nome` | Ao selecionar estado | [nomes] | `ibge_cities_{UF}` |
| ViaCEP | `https://viacep.com.br/ws/{CEP}/json/` | Ao digitar 8 digitos de CEP | {logradouro, bairro, localidade, uf} | Nao (1 request por CEP) |

**Fallbacks:** Se IBGE falhar, cai pra input texto livre. Se ViaCEP falhar, libera campos rua/bairro para edicao manual.

### 1.9 Upload de Arquivos

- Tipos aceitos: image/jpeg, image/png, image/webp, image/heic, application/pdf
- Tamanho max: 10MB
- Tamanho min (imagens): 50KB
- Armazenamento: Base64 data URL em `answers.pessoas_files`
- Camera: Mobile usa `capture="environment"`, desktop usa `getUserMedia` com modal
- Chaves dos arquivos: `p{i}_doc`, `p{i}_endereco`, `p{i}_certidao`

---

## PARTE 2: DASHBOARD

### 2.1 Secoes do Dashboard (ordem de cima pra baixo)

1. **Header** - titulo + botoes Exportar CSV / Limpar dados
2. **Summary Cards** - Total Leads + contagem por Tier (A-E) com %
3. **Search Bar** - busca por nome, email, WhatsApp, CPF
4. **Filtros** - Data inicio/fim, Score min/max, Tier, Limpar filtros
5. **Tabela de Submissions** - clicavel, expand com detalhes + quick actions
6. **Trending** - grafico de barras diario/semanal/mensal + KPIs
7. **Oportunidades CNAE** - leads que pediram contato sobre CNAEs extras
8. **Funil de Conversao** - drop-off screen-by-screen
9. **Analytics por Pergunta** - distribuicao de respostas por MC question

### 2.2 Busca Rapida

```javascript
// Busca em:
a.name, a.email, a.whatsapp,
p0.nome, p0.cpf, p0.whatsapp, p0.email,
// + todos os socios
```
- Case-insensitive
- Normaliza: remove espacos, parenteses, pontos, hifens (pra buscar telefone/CPF sem formatacao)
- Highlight amarelo nos matches na tabela

### 2.3 Trending (Diario/Semanal/Mensal)

**Tabs:** Diario | Semanal | Mensal

**KPIs por periodo:**
- Atual (Hoje / Esta semana / Este mes) + delta % vs anterior
- Anterior (Ontem / Semana passada / Mes passado)
- Media (ultimos 7 dias / 4 semanas / 6 meses)
- Pico no periodo + quando

**Grafico:**
- Diario: 14 barras (ultimos 14 dias)
- Semanal: 12 barras (ultimas 12 semanas)
- Mensal: 12 barras (ultimos 12 meses)

### 2.4 Oportunidades CNAE (Upsell)

Filtra leads onde `answers.cnpj_wants_contact === true`.
Mostra: nome, tempo relativo (timeAgo), WhatsApp link, email, Tier badge, botao copiar email.
Ordenado do mais recente pro mais antigo.
Badge com contagem total.

### 2.5 Funil de Conversao

Le `roi_sessions` do localStorage.
Para cada tela do SCREEN_ORDER, conta quantas sessoes chegaram naquela tela (lastScreenOrder >= idx).
Calcula drop-off entre telas consecutivas.

**KPIs:** Sessoes iniciadas, Concluidas, Abandonadas, Taxa conversao %

### 2.6 Quick Actions (Expanded Row)

Ao expandir uma linha da tabela:
- Copiar Nome, Email, WhatsApp, CPF
- Abrir WhatsApp (wa.me/55{digits})
- Enviar email (mailto)
- Abrir todos os anexos (abre cada arquivo em nova aba)
- Copiar resumo (texto formatado com todos os dados relevantes do lead)

### 2.7 Labels de Opcoes (Dashboard)

```javascript
var optionLabels = {
  q2:  ['Masculino', 'Feminino', 'Prefiro nao informar'],
  q3:  ['Menos de 18 anos', 'Entre 18 e 24 anos', 'Entre 25 e 34 anos', 'Entre 35 e 44 anos', 'Acima de 45 anos'],
  q4:  ['CLT', 'Autonomo', 'Empresario', 'Desempregado', 'Estudante', 'Servidor Publico', 'Outro'],
  q5:  ['Sim', 'Nao', 'Uso compartilhado'],
  q6:  ['Manha', 'Tarde', 'Noite', 'Variado', 'Nao definido'],
  q7:  ['Sim, ja vendo', 'Sim, mas nao vendo', 'Nao tenho'],
  q8:  ['Sim', 'Nao'],
  q9:  ['Renda extra', 'Aumentar renda', 'Viver da Amazon', 'Escalar negocio'],
  q10: ['Nao comecei', 'Em andamento', 'Produtos validados', 'Sim, em outro marketplace'],
  q11: ['Nao comecei', 'Em negociacao', 'Ja comprei', 'Sim, em outro marketplace'],
  q12: ['Nao tenho', 'Em analise/envio', 'Ativa', 'Sim, em outro marketplace'],
  q13: ['Nao', 'Sim, poucas vendas', 'Sim, vendas consistentes'],
  q14: ['Menos de R$2.000', 'R$2.000 a R$5.000', 'R$5.000 a R$10.000', 'R$10.000 a R$20.000', 'Acima de R$20.000'],
  q15: ['Nao pretendo reinvestir', 'Ate R$1.000/mes', 'R$1.000 a R$3.000/mes', 'R$3.000 a R$5.000/mes', 'Acima de R$5.000/mes'],
  q16: ['Menos de 1h', '1 a 2h', '2 a 4h', 'Mais de 4h'],
  q17: ['Nenhum', 'Basico', 'Intermediario', 'Avancado'],
  q19: ['Nenhuma/dificuldade', 'Basico', 'Intermediario', 'Avancado'],
  q20: ['Ate R$5.000', 'R$5.000 a R$20.000', 'R$20.000 a R$50.000', 'Acima de R$50.000'],
  q21: ['Eu mesmo', 'Socio', 'Funcionario', 'Ainda nao definido'],
  q23: ['Eu mesmo', 'Socio', 'Funcionario', 'Terceirizado'],
  q25: ['Apenas eu', '2 socios', '3 socios', '4 ou mais'],
  q29: ['R$ 5.000', 'R$ 7.000', 'R$ 10.000', '+ de R$ 10.000']
};
```

### 2.8 Pontuacao no Dashboard (scorePointsMap)

```javascript
var scorePointsMap = {
  q14: [5, 15, 30, 40, 50],   // /50 pts
  q15: [0, 5, 10, 15, 20],    // /20 pts
  q16: [5, 10, 20, 30],       // /30 pts
  q17: [0, 4, 8, 12],         // /12 pts
  q19: [0, 3, 5, 8]           // /8 pts
};
```

### 2.9 FUNNEL_SCREENS (Dashboard)

```javascript
var FUNNEL_SCREENS = [
  { id: 'screen-welcome',          label: 'Tela inicial',                sub: 'Welcome / intro' },
  { id: 'screen-1',                label: 'Q1: Dados pessoais',          sub: 'Nome, WhatsApp, e-mail' },
  { id: 'screen-2',                label: 'Q2: Genero',                  sub: '' },
  { id: 'screen-3',                label: 'Q3: Faixa etaria',            sub: '' },
  { id: 'screen-4',                label: 'Q4: Profissao',               sub: '' },
  { id: 'screen-5',                label: 'Q5: Computador proprio',      sub: '' },
  { id: 'screen-6',                label: 'Q6: Periodo do dia',          sub: '' },
  { id: 'screen-7',                label: 'Q7: Loja Amazon',             sub: '' },
  { id: 'screen-8',                label: 'Q8: Outro marketplace',       sub: '' },
  { id: 'screen-9',                label: 'Q9: Objetivo',                sub: '' },
  { id: 'screen-10',               label: 'Q10: Mineracao',              sub: '' },
  { id: 'screen-11',               label: 'Q11: Primeira compra',        sub: '' },
  { id: 'screen-12',               label: 'Q12: Oferta',                 sub: '' },
  { id: 'screen-13',               label: 'Q13: Primeira venda',         sub: '' },
  { id: 'screen-14',               label: 'Q14: Investimento',           sub: 'Pontua no score' },
  { id: 'screen-15',               label: 'Q15: Reinvestimento',         sub: 'Pontua no score' },
  { id: 'screen-16',               label: 'Q16: Horas/dia',              sub: 'Pontua no score' },
  { id: 'screen-17',               label: 'Q17: Conhecimento',           sub: 'Pontua no score' },
  { id: 'screen-19',               label: 'Q19: Habilidades PC',         sub: 'Pontua no score' },
  { id: 'screen-20',               label: 'Q20: Meta 6 meses',           sub: '' },
  { id: 'screen-22',               label: 'Q22: Intro CNPJ',             sub: 'Checklist documentos' },
  { id: 'screen-23',               label: 'Q23: Responsavel',            sub: '' },
  { id: 'screen-24',               label: 'Q24: Compromisso',            sub: 'Checkbox de aceite' },
  { id: 'screen-25',               label: 'Q25: Qtd socios',             sub: '' },
  { id: 'screen-26',               label: 'Q26: Dados pessoais + docs',  sub: 'Tela longa (maior drop)' },
  { id: 'screen-cnpj-exclusivity', label: 'CNPJ: Exclusividade',         sub: 'Marketplaces ou outras finalidades' },
  { id: 'screen-27',               label: 'Q27: Razao Social',           sub: '' },
  { id: 'screen-28',               label: 'Q28: Nome Fantasia',          sub: '' },
  { id: 'screen-29',               label: 'Q29: Capital Social',         sub: '' },
  { id: 'screen-30',               label: 'Q30: Termos + contrato',      sub: '' },
  { id: 'screen-loading',          label: 'Loading: submissao',          sub: 'Processando envio' },
  { id: 'screen-result',           label: 'Concluido',                   sub: 'Formulario finalizado' }
];
```

---

## PARTE 3: ARQUITETURA SUPABASE (PROPOSTA)

### 3.1 Esquema de Tabelas Recomendado

```sql
-- Tabela principal de leads/submissions
CREATE TABLE submissions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT now(),
  
  -- Dados basicos (Q1)
  name TEXT NOT NULL,
  email TEXT,
  whatsapp TEXT,
  
  -- Respostas do formulario (Q2-Q30)
  answers JSONB NOT NULL,
  
  -- Score calculado
  score INTEGER,
  tier CHAR(1) CHECK (tier IN ('A','B','C','D','E')),
  score_override BOOLEAN DEFAULT false,
  score_override_reason TEXT,
  
  -- CNPJ
  cnpj_exclusivity TEXT,
  cnpj_wants_contact BOOLEAN,
  
  -- Metadados
  user_agent TEXT,
  ip_address INET
);

-- Tabela de pessoas/socios (1:N com submissions)
CREATE TABLE pessoas (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  submission_id UUID REFERENCES submissions(id) ON DELETE CASCADE,
  pessoa_index INTEGER NOT NULL, -- 0 = responsavel, 1+ = socios
  
  nome TEXT NOT NULL,
  cpf TEXT,
  rg TEXT,
  uf_nascimento CHAR(2),
  cidade_nascimento TEXT,
  endereco_cep TEXT,
  endereco_rua TEXT,
  endereco_numero TEXT,
  endereco_bairro TEXT,
  endereco_cidade TEXT,
  endereco_uf CHAR(2),
  endereco_complemento TEXT,
  endereco_completo TEXT,
  data_nascimento DATE,
  estado_civil TEXT,
  regime_casamento TEXT,
  whatsapp TEXT,
  servidor_publico BOOLEAN,
  beneficio_inss BOOLEAN,
  certificado_digital BOOLEAN,
  cnh_ativa BOOLEAN,
  autodeclaracao_raca TEXT,
  profissao TEXT,
  email TEXT,
  percentual INTEGER,
  
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Tabela de arquivos (1:N com pessoas)
CREATE TABLE arquivos (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  pessoa_id UUID REFERENCES pessoas(id) ON DELETE CASCADE,
  tipo TEXT NOT NULL, -- 'doc', 'endereco', 'certidao'
  nome_arquivo TEXT,
  mime_type TEXT,
  tamanho_bytes INTEGER,
  storage_path TEXT, -- path no Supabase Storage
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Tabela de sessoes/funil
CREATE TABLE sessions (
  id TEXT PRIMARY KEY, -- sess_timestamp_random
  started_at TIMESTAMPTZ DEFAULT now(),
  last_activity_at TIMESTAMPTZ,
  last_screen TEXT,
  last_screen_order INTEGER,
  screens_visited TEXT[],
  completed BOOLEAN DEFAULT false,
  completed_at TIMESTAMPTZ,
  user_agent TEXT,
  name TEXT,
  email TEXT,
  whatsapp TEXT
);

-- Indices para performance
CREATE INDEX idx_submissions_tier ON submissions(tier);
CREATE INDEX idx_submissions_created ON submissions(created_at DESC);
CREATE INDEX idx_submissions_score ON submissions(score);
CREATE INDEX idx_submissions_cnpj_contact ON submissions(cnpj_wants_contact) WHERE cnpj_wants_contact = true;
CREATE INDEX idx_pessoas_submission ON pessoas(submission_id);
CREATE INDEX idx_sessions_completed ON sessions(completed);
CREATE INDEX idx_sessions_last_screen ON sessions(last_screen_order);
```

### 3.2 Supabase Storage (Arquivos)

```
Bucket: onboarding-docs
Estrutura: /{submission_id}/{pessoa_index}/{tipo}.{ext}

Exemplo:
/abc123/0/doc.jpg          (RG do responsavel)
/abc123/0/endereco.pdf     (comprovante do responsavel)
/abc123/1/doc.jpg           (RG do socio 1)
/abc123/1/certidao.jpg     (certidao casamento socio 1)
```

### 3.3 Pontos de Integracao (o que mudar no front)

**No formulario (onboarding):**
1. `startLoading()` -> em vez de `localStorage.setItem('roi_submissions')`, fazer `POST` para Supabase
2. Upload de arquivos -> em vez de base64 em localStorage, upload para Supabase Storage
3. `initSession()` / `trackScreenView()` -> UPSERT na tabela sessions

**No dashboard:**
1. `getFilteredSubmissions()` -> em vez de `localStorage.getItem`, fazer SELECT no Supabase
2. `getFilteredSessions()` -> SELECT na tabela sessions
3. Filtros -> viram WHERE clauses na query
4. Auto-refresh -> Supabase Realtime (subscribe to changes)

### 3.4 Variaveis de Ambiente Necessarias

```
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_ANON_KEY=eyJ...
SUPABASE_STORAGE_BUCKET=onboarding-docs
```

### 3.5 RLS (Row Level Security) Sugerida

```sql
-- Submissions: qualquer um pode inserir (o form e publico), apenas autenticados podem ler
ALTER TABLE submissions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Anyone can insert" ON submissions FOR INSERT WITH CHECK (true);
CREATE POLICY "Authenticated can read" ON submissions FOR SELECT USING (auth.role() = 'authenticated');

-- Sessoes: mesmo padrao
ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Anyone can upsert" ON sessions FOR ALL WITH CHECK (true);
CREATE POLICY "Authenticated can read" ON sessions FOR SELECT USING (auth.role() = 'authenticated');
```

---

## PARTE 4: DEPLOY

### 4.1 Estrutura de Arquivos no Repo

```
cs-dashboard-project/
  public/
    index.html              <- formulario de onboarding
    dashboard.html          <- dashboard CS
    assets/
      roi-avatar.png        <- avatar do Grupo ROI
  supabase/
    migrations/
      001_create_tables.sql <- schema do banco
  .gitignore
  README.md
```

### 4.2 Para rodar localmente

```bash
cd public
npx http-server -p 8090 -c-1
```
- Formulario: http://localhost:8090/
- Dashboard: http://localhost:8090/dashboard.html

### 4.3 Para deploy (Vercel/Netlify)

Apontar o build directory para `/public`. Nao tem build step (HTML puro).

### 4.4 Status do Push para GitHub

O commit esta pronto localmente em `/tmp/cs-dashboard-project/`.
Para fazer o push, e necessario um **Personal Access Token (PAT)** do GitHub da conta `roiventures26`:
1. GitHub > Settings > Developer Settings > Personal Access Tokens > Tokens (classic)
2. Generate new token com permissao `repo`
3. Rodar: `git push https://<TOKEN>@github.com/roiventures26/cs-dashboard-project.git main`

---

## PARTE 5: PROXIMOS PASSOS

- [ ] Push pro GitHub (precisa PAT)
- [ ] Criar projeto Supabase e rodar migrations
- [ ] Integrar formulario com Supabase (submissions + storage)
- [ ] Integrar dashboard com Supabase (leitura)
- [ ] Deploy em Vercel/Netlify
- [ ] Webhook para HubSpot (criar deal/contact ao submeter)
- [ ] Notificacoes (lead Tier A/B -> alerta pro CS via WhatsApp/email)
- [ ] Auth no dashboard (login para CS acessar)
