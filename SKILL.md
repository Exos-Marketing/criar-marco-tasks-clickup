---
name: criar-marco-tasks-clickup
description: Use quando o usuário pedir para criar tasks dentro de um marco (milestone) do ClickUp Exos, associar atividades de produção (copy, design, tráfego etc.) a um marco existente, configurar campos personalizados do projeto, vincular ao milestone, atualizar a descrição do marco com seção Atividades, criar documento dentro da task, ou configurar dependências entre tasks. Gatilhos: "cria as tasks do marco X", "adiciona atividade ao marco", "tasks de copy e design do projeto Y", "cria tasks relacionadas ao planejamento de conteúdo", "vincula tasks ao marco", "cria dependência entre tasks".
---

# Criar Tasks Vinculadas a Marco no ClickUp (Exos)

Cria atividades de produção nas **listas de processo do Marketing**, com todos os **campos personalizados** do projeto, **vincula ao marco** (milestone), atualiza a **descrição do marco** com links rápidos, cria **Doc interno** na task e configura **dependências** entre tasks.

## Hierarquia (regra sagrada)
`Atividade → linkada ao → Marco → vinculado ao → Projeto`
- **Atividade** = task nas listas de processo do Marketing (`Copy/Design/Tráfego/Infra…`).
- **Marco** = task Milestone em lista de fase do projeto (custom_item_id == 1).
- O link entre atividade e marco é feito via `clickup_add_task_link`.

## Entradas mínimas (pergunte o que faltar)
1. **Marco destino** — nome ou ID do milestone ao qual as tasks serão vinculadas.
2. **Tasks a criar** — nome de cada atividade + lista de destino (Copy? Design? Tráfego?).
3. **Sigla do projeto** — ex.: `[ET][PEP][MCE][26]`. Usada como sufixo de todos os nomes.
4. **Tags** — ex.: `perpétuo`, `ebook`, `lançamento` (opcional; pergunte se não informado).
5. **Dependências** — se alguma task bloqueia outra (ex.: "Copy deve ser concluída antes do Design").
6. **Doc na task** — se deve criar um documento ClickUp dentro de alguma task.

## Fluxo (4 fases)

### Fase 1 — Descoberta (SEMPRE antes de criar qualquer coisa)
1. **Localizar o marco** — `clickup_search` pelo nome. Confirmar que é milestone (`custom_item_id == 1`). Anotar `task_id` e a lista em que está.
2. **Localizar as listas de destino** — no space Marketing (`90130315367`), folder `Produção de Conteúdo 📄` (`90130830065`):
   - Copywriting → lista `Processo de Copywriting` (`901301657744`)
   - Design → lista `Processo de Design & Criação` (`901301657750`)
   - Tráfego, Infra, Automação → consultar via `clickup_get_folder` se não souber o ID.
3. **Ler uma task-irmã** — `clickup_filter_tasks` na lista destino filtrando pela sigla do projeto. Pegar o `task_id` de uma task existente e chamar `clickup_get_task` para obter **todos os campos personalizados** com seus valores reais (option_id, user_id). Esse passo garante que os campos do novo card sejam idênticos aos do projeto.

> Se não existir task-irmã, ler os campos da lista via `clickup_get_custom_fields` e perguntar ao usuário os valores-chave (Cliente, Produto, Idioma).

### Fase 2 — Criação das tasks (`clickup_create_task`)
Para cada task:
- **`list_id`** — ID da lista de processo correta.
- **`name`** — `[Nome da atividade] [SIGLA][TIPO][PRODUTO][MÊS][ANO]`
- **`status`** — `para fazer`
- **`priority`** — `normal`
- **`markdown_description`** — começar com `**Briefing:**` no topo, **menos nas tasks de copy**, que não levam briefing (ver "Task de copy" abaixo).
- **`custom_fields`** — array com os 13 campos padrão copiados da task-irmã:

| Campo | Field ID | Tipo |
|---|---|---|
| Cliente | `4334ea1b-2d90-4974-9ac2-287142e9be5e` | dropdown |
| Produto | `b9d27455-df3d-439b-aa68-1af6c0d08fe1` | dropdown |
| Empresa | `60a53862-4eaa-49d7-910a-f20fb4352145` | dropdown |
| Idioma | `f45be5a5-a908-453f-bc37-c8300c40e71d` | dropdown |
| Estrategista | `d0c8fa87-69ee-4130-a8ad-f6058eec11bf` | users (array) |
| Copywriter | `c8a7d8d1-fced-4bbc-8147-d097fe9d61f5` | users (array) |
| Designer | `97d791d0-f777-4081-8b09-45ac6d339e8d` | users (array) |
| Editor de Vídeos | `6519b36d-da24-4312-88e7-e23052fd0c85` | users (array) |
| Webdesigner | `c49608f9-49e9-4934-ac8a-e7f0d72a177c` | users (array) |
| Infra - Tecnologia | `5a0710b5-1c1d-4d46-9ac0-df52fd7bc72f` | users (array) |
| Gestor Automações | `17c20951-740d-40fc-8502-218c4c0a1a99` | users (array) |
| Gestor Projetos | `23d522ad-218a-400d-90d7-0b3d97d0148d` | users (array) |
| Gestor Tráfego | `0b61034c-4d5f-4d4b-b21a-e0afa8e489c1` | users (array) |

> `Progresso` (`2f593e8c-…`) é automático — não setar.

Formato dos campos no payload:
```json
[
  {"id": "4334ea1b-...", "value": "<option_id_do_dropdown>"},
  {"id": "d0c8fa87-...", "value": [87901690]}
]
```

#### Task de copy: sem briefing na descrição
Regra de 04/11/26. Task de copy = nome começa com "Copy" (tipo Copy).
- **Não** escrever `**Briefing:**` na descrição, nem o mesmo conteúdo com outro rótulo (o que escrever,
  resumo do Planejamento Estratégico, regras, alertas).
- A descrição leva **só os links**: `Planejamento estratégico: {URL do marco Planejamento Estratégico do
  projeto}` e, depois da Fase 4, `Documento(s) de entrega: {nome do Doc} — {URL}` (uma linha por Doc).
- As demais tasks e os marcos seguem com `**Briefing:**`.

### Fase 3 — Pós-criação
Executar **em paralelo** assim que tiver os IDs das tasks criadas:

**a) Adicionar tags** (`clickup_add_tag_to_task`) — ex.: `perpétuo`, `ebook`.

**b) Vincular ao marco** (`clickup_add_task_link`) — para cada task criada:
```
task_id   = <id_do_marco>
links_to  = <id_da_task_criada>
```

**c) Atualizar descrição do marco** (`clickup_update_task`) — acrescentar seção de links:
```markdown
**Briefing:**

**Atividades:**
- [Nome da Task 1 [SIGLA][26]](https://app.clickup.com/t/<id_task_1>)
- [Nome da Task 2 [SIGLA][26]](https://app.clickup.com/t/<id_task_2>)
```
> Se a descrição do marco já tiver conteúdo, **preservar** o que existe e **acrescentar** os novos links à seção Atividades.

**d) Dependências** (`clickup_add_task_dependency`) — se o usuário indicar que uma task bloqueia outra:
```
task_id    = <task_que_aguarda>
depends_on = <task_que_deve_ser_concluída_primeiro>
type       = "waiting_on"
```

### Fase 4 — Doc interno na task (via Playwright)
A **API do ClickUp não suporta task como parent de Doc**. Usar o browser:

1. `browser_navigate` → `https://app.clickup.com/t/<task_id>`
2. Clicar em **"Anexar arquivo"** (visível na parte inferior da task).
3. No menu que abrir, clicar em **"Novo documento"**.
4. O Doc é criado automaticamente vinculado à task.
5. Renomear via API: `clickup_update_document_page`:
   - `document_id` = ID do doc (da URL: `docs/<doc_id>/`)
   - `page_id` = ID da página (da URL: `docs/<doc_id>/<page_id>`)
   - `name` = `[Nome da task] [SIGLA][26]`

## Parâmetros corretos das ferramentas MCP (gotchas)

| Ferramenta | Parâmetro correto | Erro comum |
|---|---|---|
| `clickup_create_task` | `list_id` (snake_case) | `listId` falha |
| `clickup_create_task` | `priority: "normal"` (string) | `3` (int) falha |
| `clickup_add_task_link` | `task_id` + `links_to` | `taskId` falha |
| `clickup_add_task_dependency` | campo `type` obrigatório: `"waiting_on"` ou `"blocking"` | omitir `type` falha |
| `clickup_update_document_page` | `document_id` + `page_id` | `doc_id` falha |

## Verificação final
```
clickup_get_task(<marco_id>)
  → linked_tasks deve conter todos os IDs criados
  → markdown_description deve ter seção "Atividades" com links

clickup_get_task(<task_id>)
  → list correto
  → status "para fazer"
  → custom_fields com os 13 campos preenchidos
  → tags presentes
  → dependencies_count > 0 (se houve dependência)
  → task de copy: descrição sem briefing, só com os links (Planejamento estratégico e Doc)
```

## Referência rápida — Listas de processo do Marketing (Exos)

| Lista | ID | Tipo de atividade |
|---|---|---|
| Processo de Copywriting | `901301657744` | copy, SMS, email, scripts |
| Processo de Design & Criação | `901301657750` | criativos, layouts, ebooks |
| Agendamentos, Publicações & Disparos | `901301657745` | agendamentos de conteúdo |
| Planejamentos & Cronogramas | `901301657748` | planejamentos gerais |
| Máquina de Ideias | `901301657743` | brainstorm e ideias |

Space Marketing: `90130315367` · Folder Produção de Conteúdo 📄: `90130830065`
