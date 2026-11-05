# criar-marco-tasks-clickup

Skill do Claude Code que cria as atividades de produção de um marco (milestone) no ClickUp da Exos. Cada task
nasce na lista de processo certa do Marketing (Copy, Design, Tráfego…), com os 13 campos personalizados copiados
de uma task-irmã do mesmo projeto. A task fica ligada ao marco e listada na descrição dele. Quando pedido, a skill
também cria um Doc dentro da task e as dependências entre as tasks.

```
marco + tasks pedidas  →  descoberta  →  criação  →  tags, vínculo, descrição do marco, dependências  →  Doc na task  →  conferência
                          (marco, listas, task-irmã)
```

## Instalação

```bash
git clone https://github.com/Exos-Marketing/criar-marco-tasks-clickup.git \
  ~/.claude/skills/criar-marco-tasks-clickup
```

Requisitos:

- O MCP do ClickUp conectado no Claude Code: é por ele que a skill cria, linka e confere as tasks.
- O MCP do Playwright, só para o Doc dentro da task (a API do ClickUp não aceita task como pai de Doc).
- Acesso ao workspace da Exos no ClickUp (`9013012202`).

**Nenhuma credencial mora neste repo.**

## Uso

No Claude Code, peça por exemplo "cria as tasks de copy e design do marco Planejamento de Conteúdo
[ET][PEP][MCE][26]". A skill pergunta o que faltar (marco, tasks e listas, sigla, tags, dependências, Doc) e
segue em 4 fases:

1. **Descoberta**: acha o marco, as listas de destino e uma task-irmã do projeto para copiar os campos.
2. **Criação**: nome `[Atividade] [SIGLA]`, status "para fazer", prioridade normal e `**Briefing:**` no topo.
3. **Pós-criação**: tags, vínculo com o marco, seção "Atividades" na descrição do marco e dependências.
4. **Doc na task**: criado pela interface (Anexar arquivo → Novo documento) e renomeado pela API.

## Regras

- Task de copy não leva briefing na descrição: só o link do Planejamento Estratégico e, depois, o(s) Doc(s) de entrega.
- A descrição do marco é preservada; os links novos entram na seção "Atividades".

## O que tem aqui

| arquivo | o que faz |
|---|---|
| `SKILL.md` | o fluxo em 4 fases, os IDs dos 13 campos e das listas de processo e os parâmetros certos das ferramentas do MCP |

---

Interno · Exos Marketing Digital e Internacionalização Ltda.
