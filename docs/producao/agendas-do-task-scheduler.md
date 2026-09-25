# Agendas do Task Scheduler

A partir da versão `0.3.0a5`, quando um bot é iniciado pelo **Agendador de Tarefas do Windows**, o jaylog identifica as tasks ativas que executam aquele bot e envia suas agendas uma vez por processo para `POST /logs/host-schedules`.

O backend substitui atomicamente as agendas sincronizadas anteriores daquele serviço. Assim, editar uma task no Windows e executar o bot novamente atualiza a visão de agendas sem acumular linhas antigas.

## Quando a coleta acontece

A coleta só roda quando todas estas condições são verdadeiras:

- o sistema é Windows;
- a cadeia de processos indica que o bot veio do Task Scheduler;
- o envio HTTP tem `JAYLOG_LOG_HTTP_ENDPOINT` e `JAYLOG_LOG_HTTP_API_KEY` configurados;
- há uma task ativa que pode ser associada ao entrypoint atual.

Ela é executada dentro de uma thread daemon. O `configure()` não espera pelo comando `schtasks`, e falhas de leitura, XML, permissão ou rede não atrasam nem interrompem o bot.

## Como uma task é associada ao bot

O jaylog compara o **arquivo** que iniciou o processo, e não apenas a pasta. Isso evita atribuir ao bot a agenda de outro script no mesmo diretório.

Dois formatos de action `Exec` são suportados:

| Formato | Exemplo | Como é associado |
| --- | --- | --- |
| Script direto | `python.exe main.py`, com **Iniciar em** `C:\bots\faturamento` | O primeiro argumento `.py` ou `.pyw` é resolvido contra **Iniciar em** e comparado ao entrypoint. Caminhos absolutos e executáveis congelados também são aceitos. |
| `.bat` ou `.cmd` | `C:\bots\faturamento\run.bat` | A pasta do batch — ou **Iniciar em** — precisa ser a pasta do entrypoint. Quando o arquivo pode ser lido, ele também precisa mencionar o nome do script, como `main.py`. |

Se um `.bat`/`.cmd` não puder ser lido (por exemplo, por permissão), a coincidência da pasta é suficiente. Variáveis de ambiente, aspas, diferenças de maiúsculas/minúsculas e a barra final de **Iniciar em** são normalizadas antes da comparação.

## Triggers representados

O backend recebe uma combinação de **Repetição**, **Granularidade** e **Horário**. O jaylog só envia triggers que consegue representar exatamente.

| Configuração no Agendador | Agenda enviada |
| --- | --- |
| Diário às 08:00 | `DIARIO` — `08:00` |
| Semanal, segunda/quarta/sexta às 07:30 | Uma linha `SEMANA` para cada dia, às `07:30` |
| Mensal, dias 5 e 20, todos os meses, às 09:00 | Uma linha `DIA` para cada dia do mês, às `09:00` |
| Diário às 08:00, repetir a cada 30 min por 10 h | Horários diários de `08:00` até `17:30` |
| Semanal, segunda às 22:00, repetir de hora em hora por 4 h | Segunda `22:00`/`23:00` e terça `00:00`/`01:00` |
| Mensal, dia 10 às 06:00, repetir a cada 2 h por 12 h | Dia 10, de `06:00` até `16:00` |
| Uma vez às 08:00, repetir de hora em hora sem duração | Agenda diária, com os 24 horários |
| Ao iniciar o computador ou ao fazer logon | `CONTINUO` |

Para uma repetição por intervalo, cada disparo é expandido em horários. Se houver mais de **24 horários em um dia de calendário**, o trigger é representado como `CONTINUO`. Se qualquer task correspondente for contínua, ela prevalece sobre as demais agendas do serviço.

Horários de `StartBoundary` com `Z` ou offset são convertidos para o fuso local da máquina. Uma `EndBoundary` já vencida e triggers explicitamente desabilitados não são enviados.

## Triggers e configurações ignorados

O jaylog nunca aproxima uma agenda. O trigger inteiro é ignorado quando ele não tem uma representação exata, incluindo:

- diário com intervalo de dias maior que 1, ou semanal com intervalo de semanas maior que 1;
- mensal com apenas parte dos meses, último dia do mês ou "primeira segunda"/outro dia da semana ordinal;
- trigger de uma vez sem repetição, ou com repetição de duração finita;
- repetição mensal que cruza meia-noite;
- repetição indefinida cujo intervalo não forma horários fixos diários;
- atraso aleatório (`RandomDelay`), idle, evento, registro ou mudança de sessão;
- elementos desconhecidos no trigger.

Tasks em `\Microsoft\...` e tasks desabilitadas também são descartadas. Uma task pode ter várias actions e vários triggers: basta uma action executar o bot; um trigger inválido não descarta os demais triggers válidos da mesma task.

## Configuração

Não há variável obrigatória nova para o caso comum. Com o endpoint de logs configurado, o endpoint de agendas é derivado automaticamente:

```text
https://api.example/logs/add
→ https://api.example/logs/host-schedules
```

| Variável | Padrão | Descrição |
| --- | --- | --- |
| `JAYLOG_HOST_SCHEDULE_ENABLED` | `true` | Liga o envio de agendas. Só tem efeito no Windows, quando iniciado pelo Task Scheduler. |
| `JAYLOG_HOST_SCHEDULE_HTTP_ENDPOINT` | derivado | Override da URL de `POST /logs/host-schedules`. |
| `JAYLOG_HOST_SCHEDULE_TIMEOUT` | `10` | Timeout, em segundos, de `schtasks /query /xml`. |

Exemplo para desativar apenas essa sincronização:

```env
JAYLOG_HOST_SCHEDULE_ENABLED=false
```

## Falhas e limites

| Situação | Comportamento |
| --- | --- |
| `schtasks` ausente, timeout, erro de permissão ou XML inválido | Nenhuma lista vazia é enviada — as agendas já existentes no backend são preservadas — e há um aviso único com o prefixo `[jaylog] schedule:`. |
| Nenhuma task corresponde ao bot, ou nenhum trigger é representável | Nada é enviado. |
| Mais de 500 linhas após a consolidação | Nada é enviado e o jaylog avisa; o teto protege o contrato HTTP. |
| Rede fora, `429` ou `5xx` | O reporter reutiliza as tentativas com backoff do registro de ambiente. |
| Backend sem a rota (`404`/`405`) | O envio de agendas é desativado para o processo, com um aviso. |
| Credencial ou payload recusado (`401`, `403`, `422`) | O envio é desativado para o processo, com um aviso. |

O envio é único por processo: agendas não são reenviadas quando o backend pede o reenvio do [Registro de Ambiente](registro-de-ambiente.md), porque esse pedido se refere apenas à identidade da execução.

## Próximo passo

Veja também [Métricas de Recursos](metricas-de-recursos.md) e [Secrets em Produção](secrets.md) para completar a configuração de produção.
