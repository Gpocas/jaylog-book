# Métricas de Recursos (CPU, Memória e Disco)

A partir da 0.3.0a4, enquanto o processo roda, o jaylog envia a cada minuto uma amostra de uso de **CPU**, **memória** e **disco** — da **máquina** e do **próprio processo** — para `POST /logs/host-metrics`. O backend usa essas amostras para desenhar gráficos por instância (serviço + máquina + usuário), com a linha de teto de cada recurso.

Os **limites da máquina** (quantidade de CPUs, RAM total e tamanho do disco) não mudam durante a execução, então não vão em cada amostra: eles seguem uma vez só, junto do [Registro de Ambiente](registro-de-ambiente.md). Com os dois, o painel consegue mostrar, por exemplo:

```
Configuração: 2 CPUs · 8 GB RAM · 10 GB de disco (C:\)

CPU:   80%
RAM:   37,5% — 3 GB de 8 GB
DISCO: 70%   — 7 GB de 10 GB
```

## Como ligar

Não é preciso configurar nada novo. Basta o envio HTTP estar configurado (`JAYLOG_LOG_HTTP_ENDPOINT` e `JAYLOG_LOG_HTTP_API_KEY`) e o registro de ambiente ligado (padrão):

```python
from jaylog import JaylogSettings, configure

configure(JaylogSettings())  # métricas começam a sair no próximo minuto cheio
```

O endpoint é derivado do endpoint de log, do mesmo jeito que o de host: `https://api.example/logs/add` vira `https://api.example/logs/host-metrics`.

| Variável | Padrão | Descrição |
| --- | --- | --- |
| `JAYLOG_HOST_METRICS_ENABLED` | `true` | Liga a coleta. Só vale se `JAYLOG_HOST_REPORT_ENABLED` também estiver ligado |
| `JAYLOG_HOST_METRICS_INTERVAL` | `60` | Segundos entre amostras (mínimo `10`) |
| `JAYLOG_HOST_METRICS_HTTP_ENDPOINT` | derivado | Override do endpoint; por padrão `/logs/add` vira `/logs/host-metrics` |

!!! note

    As métricas dependem do registro de ambiente porque é ele que diz ao backend em qual máquina e com qual usuário aquela execução (`run_id`) está rodando. Sem ele, as amostras chegariam, mas nunca apareceriam no gráfico.

## O que é medido

| Campo | Unidade | O que informa |
| --- | --- | --- |
| `sampled_at` | ISO 8601 (UTC) | Momento da amostra |
| `host_cpu_pct` | % (0–100) | CPU da máquina inteira, média desde a amostra anterior |
| `host_mem_used_bytes` | bytes | Memória em uso na máquina (`total − disponível`, o "Em uso" do Gerenciador de Tarefas) |
| `host_mem_pct` | % (0–100) | Memória em uso sobre a RAM total |
| `host_disk_used_bytes` | bytes | Espaço usado no disco onde o bot está |
| `host_disk_pct` | % (0–100) | Espaço usado sobre o tamanho do disco |
| `proc_cpu_pct` | % (0–100) | CPU do processo **e dos filhos**, média do intervalo, sobre a capacidade total da máquina |
| `proc_mem_bytes` | bytes | Memória (RSS) do processo **e dos filhos** |
| `proc_mem_pct` | % (0–100) | Memória do processo sobre a RAM total |
| `proc_io_read_bytes` | bytes | Bytes lidos pelo processo e filhos no intervalo |
| `proc_io_write_bytes` | bytes | Bytes escritos pelo processo e filhos no intervalo |

E os limites, enviados no registro de ambiente:

| Campo | O que informa |
| --- | --- |
| `cpu_count` | CPUs lógicas da máquina |
| `memory_total_bytes` | RAM física total |
| `disk_total_bytes` | Tamanho do disco medido |
| `disk_path` | Raiz do disco medido, como `C:\` ou `/` |

Qualquer campo que não puder ser medido vai como `null` — nunca como `0`.

### Detalhes que mudam a leitura do gráfico

- **"Processo" inclui os filhos.** Bots de RPA abrem Chrome, Excel e outros programas; o consumo real costuma estar neles, não no Python. O jaylog soma toda a árvore de processos iniciada pelo bot.
- **CPU do processo é relativa à máquina.** Em uma máquina com 2 CPUs, um bot usando um núcleo inteiro aparece como `50%`. Assim `proc_cpu_pct` e `host_cpu_pct` ficam na mesma escala e podem ser comparados no mesmo gráfico.
- **Disco do processo é E/S, não ocupação.** Um processo não "ocupa" uma porcentagem do disco; o que dá para medir é quanto ele lê e escreve. No Windows, esse contador soma **toda** a E/S do processo — arquivos, rede e dispositivos — por isso o nome é `io`, e não `disk`.
- **O disco medido é o do script, não o do `cwd`.** Em uma tarefa do Agendador sem "Iniciar em", o diretório de trabalho é `C:\Windows\system32`; o jaylog mede o disco onde está o script (ou o `.exe`), para não mostrar o `C:` quando o bot roda em `D:`.
- **Amostras no minuto cheio.** A coleta é alinhada ao relógio (12:01:00, 12:02:00, …), o que dá exatamente um ponto por minuto no gráfico. A primeira leitura depois do `configure()` serve só de base para os cálculos de CPU e E/S e não é enviada — publicá-la apareceria como "0%".
- **Última amostra no encerramento.** O `shutdown()` coleta e envia uma amostra final, para a execução não terminar com um buraco no último minuto.

## Um coletor por processo

Diferente do registro de ambiente, que sai uma vez por `app_name`, existe **um único coletor por processo**. CPU e memória são do processo; com [múltiplos loggers](../guia-de-uso/multiplos-loggers.md), um coletor por logger mandaria a mesma amostra várias vezes. O coletor se vincula ao **primeiro** item de `configure()` que tenha métricas habilitadas:

```python
configure([settings_orders, settings_billing])  # um coletor, vinculado a "ORDERS"
```

## Falhas não param o bot

A coleta segue a mesma regra do resto do jaylog: nada aqui pode derrubar ou travar a aplicação.

| Situação | Comportamento |
| --- | --- |
| Uma leitura falha (permissão, processo filho que acabou de fechar) | Só aquele campo vai como `null`; o resto da amostra segue |
| Backend fora do ar, `429` ou `5xx` | As amostras ficam guardadas e vão juntas no próximo minuto. O buffer guarda até **60 amostras** (1 h); passando disso, as mais antigas são descartadas |
| Backend antigo, sem a rota (`404`/`405`) | A coleta é desligada no processo, com um único aviso no stderr |
| Credencial ou payload recusado (`401`, `403`, `422`) | A coleta é desligada no processo, com aviso no stderr |
| Backend não conhece esta execução (`x-jaylog-host-required: 1`) | O jaylog reenvia o registro de ambiente |
| `psutil` não carrega nesta plataforma | A coleta não sobe e avisa uma vez no stderr; logs e registro de ambiente seguem normais |

!!! tip

    Para desligar só as métricas e manter o registro de ambiente, use `JAYLOG_HOST_METRICS_ENABLED=false`.

## Próximo passo

Em produção, credenciais como `JAYLOG_LOG_HTTP_API_KEY` normalmente não ficam no código nem no `.env` versionado — veja [Secrets em Produção](secrets.md).
