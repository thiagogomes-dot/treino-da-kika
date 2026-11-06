# CLAUDE.md — Treino da Kika

App web de treino da Erika (Kika): 4 treinos por semana (inferior/superior), protegendo joelhos (condromalácia) e
lombar (hérnia L5-S1). Publicado em https://thiagogomes-dot.github.io/treino-da-kika/ (repo
`thiagogomes-dot/treino-da-kika`). Clone local: `C:\Users\Thiago Gomes\projetos\treino-da-kika`.

É um "irmão" do Treino TH do Thiago (`C:\Users\Thiago Gomes\projetos\Treino-Hibrido`, ver o CLAUDE.md de lá):
mesma base de código e as mesmas funções, portadas em 2026-11-05.

## Regras

- Português do Brasil, respostas curtas. O Thiago aprova antes de mudar; custo zero; nunca ElevenLabs.
- **A identidade visual da Kika é dela e fica como está** (pedido do Thiago, 2026-11-05): tema claro verde-menta
  (com modo escuro), fonte Outfit, cores `--inf` #2a9d84 / `--sup` #ec8670, avatar dela. Funções novas usam as
  variáveis dela (`--acc`, `--bg`, `--glass-a`…), não as cores do Treino TH.
- Celular dela: Android.

## O que foi portado do Treino TH (2026-11-05)

- `sw.js` (`treino-kika-vN`): cache arquivo a arquivo; aviso do descanso e contagem na tela de bloqueio
  (+15 s / Pular); lembretes por push com textos dela e mensagem própria nos dias de descanso (qua, sáb, dom).
  O SW só limpa caches `treino-kika-*` (os dois apps estão no mesmo domínio github.io).
- Timer: alerta vermelho (#e5484d) nos últimos 10 s, bipe 3-2-1, alarme alto.
- Ajustes: instalar, testar aviso/som, horários dos lembretes, Spotify.
- Lembretes: mesmo Cloudflare Worker do Treino TH (`treino-th-push.thisangom-91a.workers.dev`, mesma chave VAPID).
- Spotify: mesmo app do Spotify do Thiago (Client ID `97a18fb0dd514659bee6f6bfecaad08b`). Precisa da Redirect URI
  `https://thiagogomes-dot.github.io/treino-da-kika/` e da conta da Erika em **User Management** no painel.
- Tela de abertura: avatar, "Treino da Kika", "4x por semana, joelho e lombar protegidos.", semana e treino do dia.
- Removido o código do Artifact (`window.claude`); `localStorage` `treino-erika-v1` mantido (dados dela preservados).

## Ciclo menstrual (2026-11-05)

- Fica em **Progresso → Ciclo** (seletor `.seg2` Treino | Ciclo; `S.prog`). O Thiago preferiu juntar com Progresso
  a ter uma aba própria, e **reprovou o calendário único** (treino preenchido + anel do ciclo): "muito caótico".
  Os calendários de treino e de ciclo ficam **separados**.
- Dados em `S.cycle` = `{periods:[{start,end}], logs:{data:{e,m,flow,sym}}, len, plen}`, salvos no mesmo
  `localStorage` e no backup. Fases por `cyInfo(data)`: menstrual, folicular, fértil (ovulação −5 a +1), ovulação
  (duração − 14), lútea, TPM (últimos 6 dias), atraso. Médias pelos ciclos registrados (padrão 28/5).
- Tela: anel do ciclo, próxima menstruação e período fértil, botões começou/terminou, dica de treino da fase
  (pensada no joelho e na lombar dela), check-in (energia, humor, fluxo, sintomas), calendário de fases,
  "Ciclo × treino" (treinos, energia e dor por fase), histórico e durações. Cartão da fase na tela de treino.
- Referências de funcionamento: Garmin (ciclo + treino), FitrWoman, Wild.AI, Clue, Flo. Aviso: não é contraceptivo.
- Próximo: aba **Dieta** (esperando o material da nutricionista e os remédios com horários; lembretes de remédio
  pelo mesmo Worker).

## Avaliação do treino (2026-11-05)

- Cartão **⭐ Avaliar treino** (`rvCard`): aparece depois da primeira série marcada e no cartão de treino concluído.
  Estrelas 1–5 + comentários/dúvidas (texto ou ditado 🎤 pelo SpeechRecognition do Chrome, pt-BR, grátis).
  O Thiago trocou o comentário por exercício por essa avaliação geral do treino.
- Envio: `POST /note` no Worker (`push-server/` do Treino TH), com `uid` aleatório do aparelho (`treino-uid`).
  A IA grátis do Cloudflare (Workers AI, `@cf/meta/llama-3.3-70b-instruct-fp8-fast`) responde na hora.
- **Nada vai para o Thiago** (pedido dele). Respostas só para ela, no app e por push (subs com `uid` dela).
- Revisão diária do Claude: tarefa agendada `revisao-treino-kika` (22h, no PC do Thiago), usa
  `node tools/revisao.mjs pendentes|todas|responder|aplicada` (token admin em `~/.config/treino-th-admin.txt`).
  Pode propor mudança no plano; ela aprova no app ("Aplicar essa mudança"); a revisão seguinte edita PLAN, sobe o
  cache, faz commit/push e marca como aplicada.

## Teste local

`.claude/launch.json` de `K:\CLAUDE` tem `treino-kika` (porta 8791).

## Trocar de treino (novo ciclo)

O plano fica na constante `PLAN` de `index.html` (um objeto por dia, com a lista de exercícios). Para trocar:
1. Receber o treino novo (foto, PDF ou texto do professor, ou montar junto), mostrar a proposta e esperar a aprovação.
2. Reescrever só o `PLAN` (e o texto do Guia, se mudar a lógica). **Manter o mesmo `id` para o exercício que
   continua** (assim o histórico de carga e o gráfico de evolução seguem), e usar `id` novo só para exercício novo.
3. Subir a versão do `CACHE` no `sw.js`, testar no navegador local, publicar.
4. No app: Ajustes → "Começar novo ciclo hoje" (zera o contador de semanas; o histórico fica).

## Recordes, sugestão de carga e resumo da semana (2026-11-05)

- **Recordes** (`findPRs`): ao concluir o treino, maior carga do exercício (ou mais reps com a carga recorde) vs
  histórico; o primeiro registro não conta. Ficam em `sess.prs`, aparecem no cartão de concluído e no histórico (🏆).
- **Sugestão de carga** (`sugLoad`/`sugHtml`, botão "Usar"): 2 treinos completos seguidos com a mesma carga →
  sobe um passo (`INC`: 2,5 kg no Treino TH, 2 kg na Kika; 5 lb se o exercício estiver em libras). Dor ≥ 5 nos
  últimos 7 dias → manter. Série incompleta → manter.
- **Resumo da semana**: cartão "Esta semana" em Progresso (`weekCard`); o app manda o texto (`POST /summary`) ao
  abrir e ao concluir treino, e o Worker envia o push no domingo às 20h (hora local), só para celulares com lembretes ativos.

## Trocar o treino de hoje (2026-11-05)

- `todayPlan()` = treino de hoje: o do dia da semana, ou o escolhido em "🔁 Trocar treino" (`S.program.swap =
  {date, day}`, vale só no dia). Lista mostra primeiro os treinos não feitos nesta semana. Em outro dia da barra:
  "Fazer este treino hoje". `swapBar(day)` fica no topo do treino.
- Na barra da semana, um treino conta como feito se foi feito em qualquer dia da semana (`weekDone`).
