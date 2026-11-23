# E.E. Dom Miguel — Boletim Escolar Online

Portal de notas onde **alunos e responsáveis** acompanham o boletim bimestral:
médias por disciplina, evolução por bimestre, situação (aprovado/recuperação/reprovado)
e impressão do boletim em A4.

## Estrutura do projeto

| Camada | Onde | O quê |
|---|---|---|
| Front-end | `index.html` (arquivo único) | Login + boletim, sem dependências externas — abre direto no navegador |
| Banco | Supabase `cwuhmsbupdnlaaxdxwjw` | Tabelas `turmas`, `perfis`, `disciplinas`, `notas` + view `boletim` |
| Auth | Supabase Auth (e-mail) | Cada aluno/responsável entra com matrícula + senha |

## Contas de demonstração

Todas as contas usam a senha **`dom2026`**. O e-mail interno é
`<matrícula>@alunos.eedommiguel.com.br` (montado automaticamente pelo app).

| Matrícula | Aluno | Turma | Perfil de desempenho |
|---|---|---|---|
| 2026001 | Ana Beatriz Silva | 9º Ano A | Destaque (média ~8,5) |
| 2026002 | Carlos Eduardo Souza | 9º Ano A | Limítrofe (média ~6,1) |
| 2026003 | Mariana Oliveira Santos | 9º Ano A | Bom (média ~7,2) |
| 2026004 | João Pedro Ferreira | 9º Ano B | Médio (média ~6,7) |
| 2026005 | Laura Fernandes Costa | 9º Ano B | Excelente (média ~8,9) |
| 2026006 | Pedro Henrique Almeida | 9º Ano B | Em risco (média ~4,8) |

> O botão **“Ver demonstração”** na tela de login abre o boletim com dados
> fictícios embutidos, sem precisar de internet.

## Regras de avaliação implementadas

- Notas de **0,0 a 10,0**, um lançamento por disciplina por bimestre.
- Média do bimestre = média das 8 disciplinas; média anual = média das médias.
- Situação: **≥ 6,0 aprovado** · **5,0–5,9 recuperação** · **< 5,0 reprovado**.
  O corte fica em `MIN_APPROVAL` / `MIN_RECOVERY` no topo do `<script>` do `index.html`.

## Segurança

- **RLS ligado** em todas as tabelas: cada aluno autenticado só lê o **próprio**
  perfil e as **próprias** notas (`auth.uid()`).
- `turmas` e `disciplinas` são legíveis por qualquer usuário autenticado.
- Sem política de INSERT/UPDATE/DELETE para alunos — lançamento de notas só via
  painel/serviço da secretaria.
- Cadastro público **desabilitado** no Auth (`disable_signup: true`); as contas
  são criadas pela escola. E-mail autoconfirmado (`mailer_autoconfirm: true`)
  pois o domínio `alunos.eedommiguel.com.br` é interno.
- A chave usada no front-end é a **publishable** (`sb_publishable_...`), que é
  pública por definição e segura com RLS. A chave *service_role* **não** aparece
  em nenhum arquivo do projeto.

## Como cadastrar o primeiro aluno real

1. No painel do Supabase → **Authentication → Users → Add user**:
   e-mail `<matrícula>@alunos.eedommiguel.com.br`, senha temporária, marque
   “Auto Confirm User”.
2. Copie o `id` (UUID) do usuário criado e rode no SQL Editor:

   ```sql
   insert into public.perfis (id, matricula, nome, turma_id, responsavel)
   select 'UUID-DO-USUARIO', '2027007', 'Nome do Aluno',
          (select id from public.turmas where nome = '9º Ano A'),
          'Nome do Responsável';
   ```

3. Lance as notas em `public.notas` (ou use a view `public.boletim` para conferir):

   ```sql
   insert into public.notas (aluno_id, disciplina_id, bimestre, nota)
   values ('UUID-DO-USUARIO', (select id from public.disciplinas where nome = 'Matemática'), 1, 7.5);
   ```

## View `boletim`

`select * from public.boletim;` retorna uma linha por aluno × disciplina com
`bimestre_1..4` e `media`. Usa `security_invoker = true`, então respeita o RLS
do chamador.

## Próximos passos sugeridos

1. **Painel do professor** — tela separada para lançar/corrigir notas (papel
   `professor` em uma tabela `professores`).
2. **Frequência** — tabela `frequencia` + coluna no boletim (a escola exige ≥ 75%).
3. **Recuperação** — registrar notas de recuperação final e regra de desempate.
4. **Domínio próprio** — publicar em `boletim.eedommiguel.com.br` e ajustar
   `Site URL`/`URI allow list` no Auth.
5. **Troca de senha no 1º acesso** — forçar o responsável a trocar a senha
   temporária.

## Arquivos

- `index.html` — aplicação completa (login, boletim, impressão)
- `README.md` — este documento
