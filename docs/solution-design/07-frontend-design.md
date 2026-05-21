# 07 — Design do Frontend

> **Camada:** Experience | **Depende de:** 06-api-design

## Referência Visual

O SIFU reproduz o visual do **SIAFI Web**, que segue o **Design System do Governo Federal (DSGov)** do SERPRO. A interface é corporativa e transacional: formulários extensos, tabelas com paginação, menus hierárquicos, sem ornamentos desnecessários.

Fontes de referência visuais pesquisadas:
- Manual SIAFI (manualsiafi.tesouro.gov.br)
- Tutorial UFSM de acesso ao SIAFI Web (2024)
- Manual de emissão de NE — UFPel (2021)
- Design System Gov.br v4 (next-ds.estaleiro.serpro.gov.br)

---

## Sistema de Cores (DSGov / Gov.br)

| Token | Hex | Uso |
|---|---|---|
| `cor-primaria` | `#1351B4` | Cabeçalho, botões primários, links ativos no menu |
| `cor-primaria-escura` | `#0C326F` | Hover de botões primários, cabeçalho hover |
| `cor-secundaria` | `#2670E8` | Destaques, badges de status ativo |
| `cor-fundo` | `#FFFFFF` | Fundo de todas as telas |
| `cor-fundo-cinza` | `#F8F8F8` | Fundo de linhas alternadas em tabelas |
| `cor-texto` | `#333333` | Texto principal |
| `cor-texto-secundario` | `#636363` | Labels, placeholders |
| `cor-borda` | `#CCCCCC` | Bordas de campos e tabelas |
| `cor-erro` | `#E52207` | Campos obrigatórios (*), mensagens de erro |
| `cor-sucesso` | `#168821` | Status aprovado, processado |
| `cor-alerta` | `#FFCD07` | Status pendente, atenção |
| `cor-cancelado` | `#888888` | Status cancelado, anulado |

## Tipografia

Fonte principal: **Rawline** (fonte oficial do Governo Federal).
Fallback: `Verdana, Geneva, Tahoma, sans-serif`.

CDN da fonte Rawline: disponível via Design System Gov.br.

| Uso | Tamanho | Peso |
|---|---|---|
| Títulos de seção | 18px | SemiBold (600) |
| Labels de formulário | 14px | Regular (400) |
| Dados em tabela | 14px | Regular (400) |
| Cabeçalho de tabela | 13px | SemiBold (600) |
| Botões | 14px | SemiBold (600) |
| Mensagens de erro | 13px | Regular (400) |

---

## Layout Geral

```
┌──────────────────────────────────────────────────────────────────────┐
│ CABEÇALHO (altura: 60px, fundo: #1351B4, texto: branco)              │
│ [SIFU logo] Sistema Integrado Financeiro Unificado  [UG][Ano][Usuário▼]│
├────────────┬─────────────────────────────────────────────────────────┤
│            │ BREADCRUMB: Início > Execução > Nota de Empenho         │
│  SIDEBAR   ├─────────────────────────────────────────────────────────┤
│  (240px,   │                                                          │
│  retrátil) │              ÁREA DE CONTEÚDO                           │
│            │                                                          │
│ ▼ Orçamento│  [título da página]                                      │
│   Classif. │                                                          │
│   Dotações │  [filtros / formulário / tabela]                         │
│   Notas de │                                                          │
│   Crédito  │                                                          │
│            │                                                          │
│ ▼ Execução │                                                          │
│   Empenhos │                                                          │
│   Liquidações                                                         │
│   Pagamentos                                                          │
│            │                                                          │
│ ▼ Consultas│                                                          │
│   Execução │                                                          │
│   Dashboard│                                                          │
│            │                                                          │
│ ▼ Cadastros│                                                          │
│   UGs      │                                                          │
│   Fornecedores                                                        │
│            │                                                          │
│ ▼ Administ.│                                                          │
│   Usuários │                                                          │
│   Tokens   │                                                          │
│   Auditoria│                                                          │
└────────────┴─────────────────────────────────────────────────────────┘
```

O sidebar pode ser recolhido para apenas ícones (largura: 56px), expandindo ao passar o mouse.

---

## Telas Principais

### Tela de Login

```
┌──────────────────────────────────────────────────────────┐
│                    (fundo branco)                        │
│                                                          │
│              ┌──────────────────────────┐               │
│              │  [Logo SIFU]             │               │
│              │  Sistema Integrado       │               │
│              │  Financeiro Unificado    │               │
│              │                          │               │
│              │  Login *                 │               │
│              │  ┌──────────────────┐    │               │
│              │  │                  │    │               │
│              │  └──────────────────┘    │               │
│              │                          │               │
│              │  Senha *                 │               │
│              │  ┌──────────────────┐    │               │
│              │  │                  │    │               │
│              │  └──────────────────┘    │               │
│              │                          │               │
│              │  [   Entrar   ] ←azul    │               │
│              │                          │               │
│              │  Esqueci minha senha     │               │
│              └──────────────────────────┘               │
│                                                          │
│   * Campo obrigatório                    (rodapé gov)   │
└──────────────────────────────────────────────────────────┘
```

### Tela de Lista (padrão para NE, NC, OB, etc.)

```
┌─ Notas de Empenho ────────────────────────────────────────┐
│                                                           │
│  ┌─ Filtros ──────────────────────────────────────────┐  │
│  │ UG: [______▼]  Exercício: [2025▼]  Status: [___▼]  │  │
│  │ Fornecedor: [____________]  Período: [__] a [__]    │  │
│  │                              [Pesquisar] [Limpar]   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                           │
│  [+ Nova Nota de Empenho]               143 registros    │
│                                                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Número      │ Fornecedor    │ Valor      │ Status   │  │
│  ├─────────────┼───────────────┼────────────┼──────────┤  │
│  │ 2025MIN0001 │ Empresa ABC   │ R$ 80.000  │ ●A LIQ.  │  │
│  │ 2025MIN0002 │ Empresa XYZ   │ R$ 45.000  │ ●LIQUID. │  │
│  │ 2025MIN0003 │ Fornec. Beta  │ R$ 12.000  │ ●PAGA    │  │
│  │ 2025MIN0004 │ Empresa DEF   │ R$ 30.000  │ ●ANULADA │  │
│  └────────────────────────────────────────────────────┘  │
│                                                           │
│  ← Anterior   Página 1 de 8   Próxima →    [20 ▼/pág]   │
└───────────────────────────────────────────────────────────┘
```

### Formulário de Nota de Empenho

```
┌─ Nova Nota de Empenho ────────────────────────────────────┐
│                                                           │
│  ┌─ Dados do Empenho ─────────────────────────────────┐  │
│  │ UG Emitente *         Exercício *                   │  │
│  │ [Ministério da Educ.▼] [2025▼]                     │  │
│  │                                                     │  │
│  │ Tipo de Empenho *     Data de Emissão *             │  │
│  │ [ORDINÁRIO       ▼]   [20/05/2025    ]              │  │
│  │                                                     │  │
│  │ Fornecedor *                                        │  │
│  │ [CNPJ ou nome... 🔍]                                │  │
│  │                                                     │  │
│  │ Processo                                            │  │
│  │ [23000.001234/2025-01                ]              │  │
│  │                                                     │  │
│  │ Valor *                                             │  │
│  │ [R$              80.000,00]                         │  │
│  │                                                     │  │
│  │ Descrição do Objeto *                               │  │
│  │ [                                             ]     │  │
│  │ [                                             ]     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─ Célula Orçamentária ──────────────────────────────┐  │
│  │ Dotação *                                           │  │
│  │ [Selecione a dotação... 🔍]                         │  │
│  │                                                     │  │
│  │ Ação Orçamentária    Plano Interno                  │  │
│  │ [2065 - Apoio ao..] [PI-001 - ...]  (read-only)    │  │
│  │                                                     │  │
│  │ Natureza Despesa     Fonte Recurso    PTRES          │  │
│  │ [3.3.90.30       ]  [0100          ] [12345]        │  │
│  │                                                     │  │
│  │ Saldo Disponível: R$ 430.000,00                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  * Campo obrigatório                                      │
│  [Cancelar]                          [Emitir Empenho]    │
└───────────────────────────────────────────────────────────┘
```

### Dashboard

```
┌─ Dashboard — Exercício 2025 ──────────────────────────────┐
│                                                           │
│  UG: [Ministério da Educação         ▼]  [Atualizar]     │
│                                                           │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐  │
│  │ CRÉDITO DISP.│ │ EMPENHADO    │ │ LIQUIDADO        │  │
│  │ R$ 1.400.000 │ │ R$ 800.000   │ │ R$ 600.000       │  │
│  │  ▲ 63,6%     │ │  ▲ 36,4%     │ │  ▲ 27,3%         │  │
│  │  do total    │ │  do total    │ │  do total        │  │
│  └──────────────┘ └──────────────┘ └──────────────────┘  │
│                   ┌──────────────┐                        │
│                   │ PAGO         │                        │
│                   │ R$ 500.000   │                        │
│                   │  ▲ 22,7%     │                        │
│                   │  do total    │                        │
│                   └──────────────┘                        │
│                                                           │
│  ┌─ Execução por Ação Orçamentária ───────────────────┐  │
│  │ Ação           │ Dotação    │ Empenh. │ % Exec.    │  │
│  │ 2065 - Apoio.. │ 1.200.000  │ 500.000 │ 41,7%      │  │
│  │ 2066 - Manutenç│ 1.000.000  │ 300.000 │ 30,0%      │  │
│  └────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

---

## Componentes Reutilizáveis

| Componente | Descrição |
|---|---|
| `CabecalhoSistema` | Barra superior fixa com logo, UG, exercício, usuário |
| `MenuLateral` | Sidebar retrátil com grupos e itens de navegação |
| `Breadcrumb` | Caminho de navegação acima do conteúdo |
| `FiltrosPesquisa` | Painel de filtros com campos específicos por módulo |
| `TabelaDados` | Tabela paginada com ordenação, usando TanStack Table |
| `FormularioCampo` | Label + input + mensagem de erro + asterisco obrigatório |
| `SeletorComLupa` | Campo de texto com botão de busca para entidades |
| `BadgeStatus` | Indicador colorido de status (A_LIQUIDAR, APROVADA, etc.) |
| `CartaoMetrica` | Card de dashboard com valor e percentual |
| `ModalConfirmacao` | Confirmação para ações irreversíveis (aprovar, cancelar) |
| `MensagemErro` | Exibição de erros de API no formato RFC 7807 |

---

## Estrutura de Arquivos

```
src/
├── assets/
│   └── rawline/         ← fontes do gov.br
├── componentes/
│   ├── layout/
│   │   ├── CabecalhoSistema.jsx
│   │   ├── MenuLateral.jsx
│   │   └── Breadcrumb.jsx
│   ├── formulario/
│   │   ├── FormularioCampo.jsx
│   │   └── SeletorComLupa.jsx
│   ├── tabela/
│   │   └── TabelaDados.jsx
│   └── comum/
│       ├── BadgeStatus.jsx
│       ├── CartaoMetrica.jsx
│       ├── ModalConfirmacao.jsx
│       └── MensagemErro.jsx
├── paginas/
│   ├── autenticacao/
│   │   └── Login.jsx
│   ├── orcamento/
│   │   ├── dotacao/
│   │   ├── notacredito/
│   │   └── classificacao/
│   ├── execucao/
│   │   ├── notaempenho/
│   │   ├── liquidacao/
│   │   └── ordembancaria/
│   ├── consultas/
│   │   ├── ExecucaoOrcamentaria.jsx
│   │   └── Dashboard.jsx
│   ├── cadastros/
│   │   ├── unidadegestora/
│   │   └── fornecedor/
│   └── administracao/
│       ├── usuario/
│       ├── token/
│       └── auditoria/
├── servicos/
│   └── api.js           ← instância Axios configurada com base URL e interceptors
├── contextos/
│   └── AutenticacaoContexto.jsx
└── roteador.jsx
```

---

## Comportamentos de UX (inspirados no SIAFI Web)

- **Campos read-only** preenchidos automaticamente aparecem em fundo cinza claro, não editáveis.
- **Campos obrigatórios** têm asterisco vermelho `*` após o label.
- **Lupas de pesquisa** (`🔍`) em campos de entidades (fornecedor, dotação) abrem modal de busca.
- **Saldo disponível** é exibido em tempo real ao selecionar a dotação no formulário de NE.
- **Ações destrutivas** (anular, cancelar, estornar) exigem confirmação em modal.
- **Feedback de operação**: toast de sucesso (verde) ou erro (vermelho) após submit.
- **Bloqueio de formulário**: botão de submissão é desabilitado durante a requisição.
- **Paginação**: padrão de 20 itens; seletor de quantidade por página (10, 20, 50, 100).
