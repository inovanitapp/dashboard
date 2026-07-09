
# Esse é o roteiro para tornar o Inova Leagues funcional no Supabase

O protótipo hoje é 100% visual (mock em `src/lib/mock-data.ts` e `src/lib/competitions-data.ts`). Autenticação, perfis e papéis (admin/coordenador/aluno) já existem no banco. Falta modelar e conectar: equipes, membros, competições, marcos, tarefas (kanban), mensagens de chat e eventos de agenda.

A ordem abaixo é incremental — cada fase entrega algo funcional antes da próxima. Ao final de cada fase você me envia o prompt indicado (copiar/colar), eu implemento e valido.

## Ordem sugerida das fases

```text
1. Fundação de dados (schema completo + RLS)
2. Equipes e membros
3. Competições, inscrições e marcos
4. Kanban (colunas + tarefas + atribuições)
5. Chat por equipe (Realtime)
6. Agenda (eventos manuais + marcos + mensagens fixadas)
7. Painel admin (gestão de usuários e papéis)
8. Limpeza dos mocks + seed opcional
```

## Prompts prontos para cada fase

**Fase 1 — Schema e RLS**
> "Crie as migrações Supabase para as tabelas: teams, team_members (com role: coordenador/aluno), competitions, competition_registrations (liga team↔competition), milestones (por competição, com toggle done por equipe → milestone_progress), kanban_columns, kanban_tasks, chat_messages, agenda_events. Inclua GRANTs, RLS com auth.uid() e uso de private.has_role para admin, timestamps com trigger update_updated_at, e habilite Realtime nas tabelas de chat, kanban e agenda."

**Fase 2 — Equipes**
> "Substitua os mocks de equipes por dados reais do Supabase. Coordenadores criam/editam suas equipes e convidam membros por matrícula; alunos veem apenas as equipes em que participam; admin vê tudo. Use server functions com requireSupabaseAuth para escrita e useSuspenseQuery para leitura."

**Fase 3 — Competições**
> "Conecte /app/competitions e /app/teams/$teamId/competition ao Supabase. Admin cria competições e marcos; coordenador inscreve a equipe; membros marcam marcos como concluídos (milestone_progress). Atualize a barra de progresso a partir do banco."

**Fase 4 — Kanban**
> "Torne o Kanban funcional por equipe: colunas persistentes, drag-and-drop com atualização otimista, tarefas com assignee (membro da equipe), prioridade, tags e data. Sincronize via Realtime."

**Fase 5 — Chat**
> "Implemente o chat da equipe com chat_messages via Supabase Realtime. Suporte /task para criar tarefa no Kanban e /event para criar evento na agenda a partir da mensagem."

**Fase 6 — Agenda**
> "Consolide na agenda: eventos manuais (agenda_events), marcos de competição da equipe e mensagens fixadas do chat. Filtro por origem e criação/edição pelo coordenador."

**Fase 7 — Admin**
> "Crie /app/admin (protegido por has_role('admin')) para listar usuários, alterar papéis via user_roles e resetar senha via Auth Admin API em server function."

**Fase 8 — Limpeza**
> "Remova src/lib/mock-data.ts e competitions-data.ts, apague referências e (opcional) crie um seed SQL com 1 competição e 1 equipe de demonstração vinculadas ao Carlos e à Ana Carla."

## Notas técnicas

- Cada fase abre migração própria (aprovada antes do código) para manter reversibilidade.
- Escrita sempre em `createServerFn` + `requireSupabaseAuth`; leitura por `useSuspenseQuery` com loader `ensureQueryData`.
- Realtime em `chat_messages`, `kanban_tasks`, `agenda_events` (ALTER PUBLICATION supabase_realtime).
- RLS: membros leem/escrevem via `team_members`; admin via `private.has_role`; coordenador via papel na `team_members`.
- "Leaked password protection" continua sendo ajuste manual no dashboard Auth.

Diga "vamos pela fase 1" (ou cole o prompt correspondente) para eu começar.
