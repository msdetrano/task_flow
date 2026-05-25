# TaskFlow AI — Gestão Inteligente de Tarefas

> **Disciplina:** Desenvolvimento de Aplicações Web Low-Code  
> **Instituição:** UniFECAF & Rocketseat  
> **Plataforma:** OutSystems

> **Link do Projeto:** https://personal-axdkuugi-dev.outsystems.app/TaskFlowAI

---

## Sumário

- [Sobre o Projeto](#sobre-o-projeto)
- [Diferenciais](#diferenciais)
- [Arquitetura Canvas](#arquitetura-canvas)
- [Modelo de Dados](#modelo-de-dados)
- [Telas da Aplicação](#telas-da-aplicação)
- [Inteligência Artificial](#inteligência-artificial)
- [Automações e Timers](#automações-e-timers)
- [Sistema de Autorização](#sistema-de-autorização)
- [Acessibilidade e Sustentabilidade](#acessibilidade-e-sustentabilidade)
- [Como Testar](#como-testar)
- [Tecnologias](#tecnologias)

---

## Sobre o Projeto

**TaskFlow AI** é uma aplicação web low-code desenvolvida em **OutSystems** para gestão inteligente de tarefas corporativas. Vai além de um simples gerenciador de tarefas: integra **Inteligência Artificial via OpenAI** para sugestão automática de prioridade com base no conteúdo de cada demanda.

O sistema resolve um problema real de equipes de TI, suporte e operações — a dificuldade de priorizar dezenas de tarefas de forma objetiva, sem viés humano, com rastreabilidade completa de cada decisão.


<img width="1173" height="901" alt="image" src="https://github.com/user-attachments/assets/c29a6419-e25e-4bd7-b19c-a6e75f2b62e7" />


---

## Diferenciais

| # | Diferencial | Descrição |
|---|---|---|
| 🤖 | **IA Integrada** | Sugestão automática de prioridade (Baixa/Média/Alta/Crítica) via OpenAI GPT-4o-mini |
| 📊 | **Dashboard Analítico** | KPIs em tempo real: pendentes, em andamento, concluídas, atrasadas |
| 🗂️ | **Kanban Visual** | Visualização drag-and-drop por status (componente Forge) |
| 📋 | **Auditoria Completa** | Histórico detalhado de toda alteração — quem, quando e o quê |
| 🎨 | **Categorização Dinâmica** | Categorias com cores personalizadas em hexadecimal |
| 🔔 | **Notificações Automáticas** | Alertas de prazo 24h antes do vencimento via e-mail |
| 📅 | **Relatório Semanal** | Resumo de produtividade gerado automaticamente toda segunda-feira |
| 🇧🇷 | **Interface em Português** | Totalmente localizada para o mercado brasileiro |

---

## Arquitetura Canvas

A aplicação segue a **Arquitetura Canvas do OutSystems** em 3 camadas com separação total de responsabilidades:

```
┌────────────────────────────────────────────────────────────┐
│                    INTERFACE LAYER                          │
│  TaskFlow_UI                                               │
│  Dashboard · Tasks · TaskDetailView · TaskDetailScreen     │
│  CategoryServers · TaskHistoryDetailScreen                 │
└──────────────────────────┬─────────────────────────────────┘
                            │  consome apenas Public Actions
┌──────────────────────────▼─────────────────────────────────┐
│                      CORE LAYER                             │
│  TaskFlow_Core                                             │
│  Entities · Static Entities · Server Actions · Timers      │
│  Task · TaskHistory · CategoryServer                       │
└──────────────────────────┬─────────────────────────────────┘
                            │  consome Connectors & Services
┌──────────────────────────▼─────────────────────────────────┐
│                   FOUNDATION LAYER                          │
│  TaskFlow_Foundation                                       │
│  OpenAI REST Connector · Email Service · Site Properties   │
└────────────────────────────────────────────────────────────┘
```

### Por que essa separação?

- **Interface** nunca acessa entidades diretamente → sem acoplamento com banco
- **Core** centraliza toda lógica de negócio → reutilizável e testável
- **Foundation** isola integrações externas → fácil substituição sem impactar o resto

---

## Modelo de Dados

### Entidades Principais

#### `Task` — Entidade Central

| Atributo | Tipo | Descrição |
|---|---|---|
| `Id` | LongInteger | Identificador único (PK, AutoNumber) |
| `Title` | Text | Título da tarefa (obrigatório) |
| `Description` | Text | Descrição detalhada |
| `StatusId` | FK → TaskStatus | Status atual da tarefa |
| `PriorityId` | FK → Priority | Prioridade definida pelo usuário |
| `AIPriorityId` | FK → Priority | **Prioridade sugerida pela IA** |
| `AIAccepted` | Boolean | Se o usuário aceitou a sugestão da IA |
| `CategoryServerId` | FK → CategoryServer | Categoria dinâmica com cor |
| `AssignedToUserId` | FK → User | Responsável pela tarefa |
| `CreatedByUserId` | FK → User | Usuário que criou a tarefa |
| `DueDate` | Date | Data de vencimento (obrigatória) |
| `IsOverdue` | Boolean | Calculado automaticamente pelo Timer diário |
| `IsArchived` | Boolean | Soft delete (Default: False) |

#### `TaskHistory` — Auditoria

| Atributo | Tipo | Descrição |
|---|---|---|
| `TaskId` | FK → Task | Tarefa relacionada |
| `ChangedByUserId` | FK → User | Quem fez a alteração |
| `ChangedOn` | DateTime | Data/hora da mudança |
| `OldStatusId` | FK → TaskStatus | Status anterior |
| `NewStatusId` | FK → TaskStatus | Novo status |
| `Notes` | Text | Observações sobre a mudança |

#### `CategoryServer` — Categorias Dinâmicas

| Atributo | Tipo | Descrição |
|---|---|---|
| `Name` | Text(100) | Nome da categoria |
| `ColorHex` | Text | Código hexadecimal da cor |
| `IsActive` | Boolean | Ativa ou inativa |

### Entidades Estáticas

```
TaskStatus:  Pending | InProgress | Done | Archived
Priority:    Low | Medium | High | Critical
```

### Relacionamentos

```
User ──< Task (AssignedToUserId)     1 usuário → muitas tarefas atribuídas
User ──< Task (CreatedByUserId)      1 usuário → muitas tarefas criadas
Task ──< TaskHistory                 1 tarefa  → muitos registros de histórico
CategoryServer ──< Task              1 categoria → muitas tarefas
```

---

## Telas da Aplicação

### Dashboard
Visão executiva com KPIs em 3 colunas (Pendentes, Em Andamento, Concluídas), gráfico de barras por status, gráfico de pizza por prioridade e lista das 5 tarefas mais urgentes ordenadas por prazo crescente.

### Lista de Tarefas (`Tasks`)
Tabela completa com filtros por status, prioridade e categoria. Busca por título, ordenação por qualquer coluna e paginação automática. Badge colorido de status e prioridade em cada linha.

### Visualização de Tarefa (`TaskDetailView`)
Layout em 3 colunas: detalhes da tarefa, timeline de datas e informações de atribuição. Seção dedicada aos atributos de IA (prioridade sugerida + status de aceitação). Aba de histórico com todas as mudanças registradas.

### Formulário de Tarefa (`TaskDetailScreen`)
Criação e edição em formulário organizado em seções: Detalhes, Atribuição, Datas, Prioridade e Status. Integra o fluxo de sugestão de IA e exibe a prioridade sugerida ao lado da prioridade manual.

### Gestão de Categorias (`CategoryServers`)
Edição inline com visualização de cor em tempo real. Filtro por status ativo, busca por nome e adição de novas categorias com código hexadecimal personalizado.

### Histórico (`TaskHistoryDetailScreen`)
Visualização detalhada de cada mudança com status anterior, novo status, data/hora e observações.

---

## Inteligência Artificial

### Como Funciona

A IA é acionada ao criar uma tarefa. A Server Action `SuggestPriority_AI` no Core chama o conector REST da Foundation:

```
[System] Você é um assistente de priorização de tarefas.
         Responda APENAS com: Baixa, Media, Alta ou Critica.

[User]   Título: {Title}
         Descrição: {Description}
         Categoria: {CategoryName}
         Prazo: {DueDate}  |  Hoje: {Today}
```

### Mapeamento da Resposta

| Resposta da IA | Prioridade no Sistema |
|---|---|
| `"Critica"` / `"Crítica"` | 🔴 Critical |
| `"Alta"` | 🟠 High |
| `"Media"` / `"Média"` | 🟡 Medium |
| `"Baixa"` / falha / timeout | 🟢 Low (fallback seguro) |

O campo `AIPriorityId` armazena a sugestão. O campo `PriorityId` armazena a decisão final do usuário. `AIAccepted` registra se a sugestão foi aceita — dados úteis para refinamento futuro do modelo.

---

## Automações e Timers

| Timer | Agendamento | Ação |
|---|---|---|
| `NotifyDeadlineTimer` | Diário às 08:00 | E-mail para tarefas com prazo em 24h |
| `WeeklyReportTimer` | Segundas às 07:00 | Relatório de produtividade para gestores |
| `CheckOverdueTimer` | Diário às 06:00 | Marca `IsOverdue=True` em tarefas atrasadas |
| `ArchiveOldTimer` | Semanal | Arquiva tarefas concluídas há mais de 30 dias |

---

## Sistema de Autorização

| Role | Permissões |
|---|---|
| **Admin** | Acesso total: usuários, categorias, tarefas, histórico, configurações |
| **Manager** | Gerenciamento completo de tarefas, categorias e histórico |
| **User** | Visualiza e atualiza suas próprias tarefas atribuídas |

---

## Acessibilidade e Sustentabilidade

**Acessibilidade (WCAG AA):**
- Labels explícitos em todos os inputs (atributo `For`, não só placeholder)
- Contraste mínimo 4.5:1 entre texto e fundo
- Navegação completa por teclado (Tab, Enter, Esc)
- ARIA labels em ícones sem texto
- Mensagens de erro descritivas

**Sustentabilidade Digital:**
- Paginação em todas as listagens (máximo 20 registros)
- Cache de dados estáticos com refresh de 30 minutos
- Soft delete em vez de exclusão física
- Timers agendados fora do horário de pico
- Aggregates otimizados — seleção apenas dos atributos necessários

---

## Como Testar

### Acesso

```
URL:    [inserir URL após publicação no OutSystems]
Admin:  admin@taskflow.com   /  TaskFlow@2025
User:   teste@taskflow.com   /  TaskFlow@2025
```

### Fluxo Principal

**1. Criar tarefa com sugestão de IA**
1. Dashboard → "Nova Tarefa"
2. Título: `"Servidor de produção fora do ar"`
3. Descrição: `"Cliente reportou sistema inacessível desde 14h"`
4. Salvar → a IA sugere automaticamente prioridade **Crítica**

**2. Acompanhar no Kanban**
1. Menu → Kanban
2. Arrastar tarefa de "Pendente" para "Em Andamento"
3. Verificar atualização de status em tempo real

**3. Consultar Dashboard**
1. Verificar KPIs atualizados
2. Observar gráficos de distribuição por status e prioridade

**4. Visualizar histórico**
1. Abrir tarefa → aba Histórico
2. Ver registros de mudanças com usuário e timestamp

---

## Tecnologias

| Tecnologia | Uso |
|---|---|
| **OutSystems** | Plataforma low-code principal |
| **Arquitetura Canvas** | Separação em Interface, Core e Foundation |
| **OpenAI GPT-4o-mini** | Sugestão automática de prioridade |
| **Kanban Board (Forge)** | Visualização drag-and-drop |
| **OutSystems Charts** | Gráficos do dashboard |
| **OutSystems Timers** | Automações de notificação e relatório |
| **OutSystems Email** | Alertas de prazo e relatórios semanais |
| **OutSystems UI** | Framework de interface responsiva |

---

## Estrutura do Repositório

```
taskflow-ai/
├── README.md                    ← Este arquivo
├── docs/
│   ├── parte_teorica.pdf        ← Documento teórico completo
│   └── apresentacao.pptx        ← Slides da apresentação
├── screenshots/
│   ├── dashboard.png
│   ├── task_list.png
│   ├── task_detail.png
│   ├── kanban.png
│   └── categories.png
└── outsystems/
    └── TaskFlowAI.oap           ← Arquivo exportado do OutSystems
```

---

*Projeto acadêmico — Desenvolvimento de Aplicações Web Low-Code · UniFECAF / Rocketseat · 2026*
