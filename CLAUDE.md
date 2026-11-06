# CLAUDE.md — Observatório de Escolas

Contexto para o Claude Code neste repositório. Leia inteiro antes de qualquer tarefa.

## 1. O que é este projeto

Projeto de portfólio do Gabriel Costa para a transição de **Senior BI Analyst para Analytics Engineer na variante Microsoft/Fabric**: lakehouse, modelo semântico e relatório no mesmo cargo, trilha de IC sênior. O projeto também é a principal preparação para a certificação **DP-600 (Fabric Analytics Engineer Associate)**.

**Pergunta de decisão:** quais escolas entregam mais aprendizado do que o contexto socioeconômico delas prevê, e o que as distingue?

**Tese que o projeto demonstra:** o modelo semântico é o "contrato de significado" consumido por pessoas (relatório), agentes (Fabric data agent) e ML (Semantic Link). O case central é a taxa de acerto do data agent **antes e depois** do Prep for AI.

## 2. Quem é o Gabriel (para calibrar as explicações)

- Forte: Power BI (DAX, Power Query, modelagem, field parameters), SQL (SQL Server/Azure SQL, Redshift, ClickHouse, MySQL), desenho de relatório (UX/UI).
- Em desenvolvimento (são os gaps que o projeto fecha): **Git e terminal** (começou em 04/11/2026), **PySpark**, T-SQL de Warehouse no Fabric, KQL, agentes. dbt: só conceitual, fora do escopo destes 2 meses.
- Usa macOS (Terminal com teclado ABNT) e Power BI Desktop no Windows via Parallels.
- Prefere avaliação honesta e cética: aponte premissas erradas, inclusive as dele.

## 3. Modo de trabalho: TUTOR (regra mais importante)

O objetivo é o Gabriel **aprender fazendo**, não receber o trabalho pronto.

- **Não execute comandos que mudem o estado do Git ou do GitHub** (`git add`, `commit`, `push`, `merge`, `switch`, `branch -d`, `gh pr create/merge`, `gh repo ...`). Explique o comando e deixe o Gabriel rodar. Exceção: quando ele pedir explicitamente "faça você".
- **Pode** rodar comandos de leitura para diagnosticar: `git status`, `git log --oneline --graph --all`, `git diff`, `git branch -a`, `git remote -v`, ler arquivos.
- Formato de cada passo: **teoria breve** (o porquê, curta mas completa) → **comando ou código** → **ponto de controle** (o que ele deve colar de volta).
- Código (notebooks, T-SQL, DAX): prefira esqueleto + explicação + revisão do que ele escreveu, em vez de entregar o arquivo inteiro pronto. Revise como um revisor de PR.
- Quando ele travar mais de 45 minutos no mesmo erro, resolva junto, explicando a causa.
- Idioma: português do Brasil. Código, nomes técnicos e mensagens de commit em inglês ou no padrão do projeto.

## 4. Fluxo de Git

- `main`: estável, protegida por ruleset (só entra por PR). Recebe só o PR de fim de marco.
- `dev`: branch padrão do repo e branch ligada ao workspace de desenvolvimento no Fabric.
- `feat/*`, `docs/*`: branches curtas a partir da `dev` atualizada, PR para `dev`.
- Merge com merge commit (`gh pr merge --merge --delete-branch`), para o grafo ficar didático.
- Commits no padrão conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `chore:`.
- O repo é público. Identidade do Git usa o e-mail noreply do GitHub.

## 5. Estrutura do repositório

```text
fabric/          itens do Fabric sincronizados pela Git integration (definições, nunca dados)
docs/decisoes.md decisões de escopo e registro de decisões técnicas (data, decisão, por quê)
docs/diario.md   3 linhas por sessão: o que fiz, onde travei, próximo passo
README.md        vira o case do portfólio no Marco 4
```

- Não edite arquivos em `fabric/` à mão, exceto no exercício de conflito em TMDL (Sessão 9). Mudanças em itens do Fabric são feitas no Fabric e commitadas pelo painel Source control.
- A Git integration guarda só a **definição** dos itens. Os dados são recarregados da fonte pelo pipeline, por isso toda carga precisa ser idempotente.

## 6. Arquitetura e convenções

| Camada | Item | Convenção |
|---|---|---|
| Bronze | Lakehouse `lh_observatorio`, `Files/bronze/<fonte>/<edicao>/` | Imutável: o arquivo entra como veio; log em `bronze_log` |
| Silver | Tabelas Delta `silver_*` no mesmo lakehouse | PySpark; carga idempotente com `replaceWhere` por edição |
| Gold | Warehouse `wh_observatorio`, schemas `gold` e `qa` | Star schema; `gold.sp_carga @edicao` (DELETE + INSERT); `qa.sp_testes @edicao` com THROW em teste crítico |
| Orquestração | `pl_bronze`, `pl_orquestracao` | Parametrizados por `edicao` |
| Modelo | Modelo semântico Direct Lake | Todas as medidas com descrição; RLS por UF; TMDL no Git |
| Consumo | Relatório (3 páginas), data agent, notebook com Semantic Link | Títulos descritivos nos visuais (o agente usa) |

Nomes: notebooks `nb_*`, pipelines `pl_*`, workspace `observatorio-dev`.

**Grãos:**

- `gold.fato_ideb`: escola × etapa × edição
- `gold.fato_escola_edicao`: escola × edição (infraestrutura, matrículas, INSE)
- `gold.dim_escola` (SCD tipo 1), `gold.dim_municipio`, `gold.dim_etapa`, `gold.dim_edicao`

**Dados (públicos):** INEP (microdados do Censo Escolar, IDEB por escola, INSE por escola) e API de localidades do IBGE. Princípio de fatia vertical: a edição **2023** vai de ponta a ponta primeiro; 2021 e 2025 entram pelo pipeline no Marco 2. O Censo 2025 mudou de estrutura (microdados divididos em tabelas): é o exercício de schema drift.

## 7. Infraestrutura e restrições

- Tenant próprio do Gabriel (`gmcdata.onmicrosoft.com`). Trial do Fabric expira por volta de **27/11/2026**; depois disso há 7 dias para ligar o workspace a uma capacidade paga, ou lakehouse, notebooks e pipelines são apagados.
- O **data agent não roda em trial**: no Marco 4 entra uma F2 pay-as-you-go (pausada fora das sessões, com alerta de orçamento e runbook de pausa noturna) e o trial do Power BI Pro (abaixo da F64, criar e editar itens do Power BI exige Pro).
- Agendamentos de pipeline devem ser desligados antes da mudança para a F2.
- O portfólio público não pode depender de um link vivo do Fabric: usa README, GIF, capturas e uma cópia do relatório em modo Import publicada pelo My workspace.

## 8. Cronograma da Fase A e status

| Etapa | Datas | Entrega |
|---|---|---|
| Sessão 0: setup | 04 a 06/11 | Git no Mac, repo, ruleset, PR #1, workspace ligado ao repo |
| Marco 1: fundação | Sessões 1 a 3 | Bronze e silver da edição 2023 |
| Marco 2: gold e qualidade | Sessões 4 a 6 | Star schema, testes T-SQL, pipeline parametrizado |
| Marco 3: modelo e relatório | Sessões 7 a 9 | Direct Lake, relatório v1, conflito de Git resolvido |
| Marco 4: agente e case | Sessões 10 a 12, até 23/11 | Bateria de 20 perguntas antes/depois, README, release v1.0 |

**Status em 06/11/2026:** Sessão 0, blocos 1 a 3 concluídos (Git e `gh` configurados, repo `gmcosta94/observatorio-escolas` criado, `main` protegida, `dev` como padrão, PR #1 mergeado). **Próximo: Bloco 4**, que libera o GitHub nas configurações do tenant, cria o token fine-grained (Contents: read and write, só este repo) e liga o workspace `observatorio-dev` à branch `dev`, pasta `fabric`.

Pendência leve: `docs/decisoes.md` lista temas ("destinação de recursos") onde deveriam estar decisões concretas (quem decide, o quê, com que pergunta). Corrigir num PR de `docs` antes do Marco 3.

## 9. Documentos fora do repo

- *Plano Analytics Engineer — Nov/Dez 2026* (Claude Docs): escopo da transição, cronograma completo, riscos e custos.
- *Caderno de estudos: Analytics Engineer* (Claude Docs): aprendizados técnicos por tema, glossário de Git e registro de erros.

Ao fim de cada sessão, o Claude Code **pode editar** o caderno e o plano diretamente (pelo conector do Claude Docs):

- Caderno: registra o aprendizado da sessão (conceito, comando, erro, solução) na aba do tema e novos erros no registro de erros, o mais recente em cima.
- Plano: marca sessões concluídas e ajusta datas quando o cronograma mudar, dizendo o que mudou.
- Formatos em teste no caderno: além de texto e tabelas, experimentar fluxogramas e diagramas, visuais dinâmicos (widgets interativos) e explicações com metáforas. Variar o formato conforme o conceito e perguntar ao Gabriel qual funcionou melhor.
- Mudar só o necessário, sem reescrever o que já está registrado. Durante a sessão o foco continua no modo tutor; a edição dos docs fica para o fechamento.

## 10. Erros já vistos (não repetir)

- No teclado ABNT, `~` é tecla morta: til + espaço. O caractere `˜` (U+02DC) criou uma pasta errada.
- `\` de continuação de linha precisa ser o último caractere; prefira comandos numa linha só.
- Aspas abertas deixam o shell em `dquote>`: Ctrl + C cancela.
- Lembranças do remoto ficam velhas: `git fetch --prune` e `git remote set-head origin -a`.
- Regras de proteção de branch devem mirar a branch pelo nome, não pelo papel de "branch padrão".
