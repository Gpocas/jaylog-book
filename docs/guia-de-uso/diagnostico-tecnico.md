# Diagnóstico Técnico

O jaylog possui um canal de diagnóstico interno para observar sua própria
operação. Ele registra decisões de configuração, inicialização de handlers,
coletas auxiliares, chamadas a subprocessos, transmissões HTTP, retries e
encerramento.

Esse canal é diferente dos registros produzidos pela aplicação. Definir
`JAYLOG_LOG_LEVEL=DEBUG` permite registros `logger.debug(...)`; definir
`JAYLOG_DEBUG=true` habilita os diagnósticos internos do jaylog.

## Habilitação

Em `.env.logging` ou no ambiente do processo:

```env
JAYLOG_DEBUG=true
```

Ou diretamente em código:

```python
from jaylog import JaylogSettings, configure

configure(JaylogSettings(app_name="meu-bot", debug=True))
```

O padrão é `false`. Quando desabilitado, o canal não produz saída nem altera o
comportamento dos handlers normais.

## Destinos

`JAYLOG_DEBUG_HANDLERS` recebe uma lista separada por vírgulas. Os valores
aceitos são `console`, `file` e `http`; o padrão é somente `console`.

```env
JAYLOG_DEBUG=true
JAYLOG_DEBUG_HANDLERS=console,file
```

Em código, use a mesma sintaxe:

```python
settings = JaylogSettings(
    app_name="meu-bot",
    debug=True,
    debug_handlers="console,file",
)
```

| Destino | Comportamento | Requisito |
| --- | --- | --- |
| `console` | Emite imediatamente em `stderr`, inclusive antes da construção do logger. | Nenhum; é independente de `JAYLOG_LOG_CONSOLE_ENABLED`. |
| `file` | Encaminha o diagnóstico ao arquivo rotativo do serviço. | `JAYLOG_LOG_DIR` configurado e o logger construído por `get_logger()`. |
| `http` | Envia o diagnóstico como um registro de log para o endpoint principal. | `JAYLOG_LOG_HTTP_ENDPOINT` e `JAYLOG_LOG_HTTP_API_KEY` configurados. |

Os destinos podem ser combinados:

```env
JAYLOG_DEBUG_HANDLERS=console,file,http
```

Um nome desconhecido ou uma lista vazia invalida `JaylogSettings`. Diagnósticos
destinados a `file` ou `http` que ocorrerem antes da construção dos handlers são
mantidos num buffer limitado e reproduzidos quando o logger é construído.

!!! warning

    O destino `http` deve ser usado temporariamente e de forma consciente: ele
    aumenta o volume de registros transmitidos. O handler impede recursão — uma
    transmissão diagnóstica não gera outro diagnóstico HTTP — e não anexa
    screenshots aos registros internos.

## Formato

No console, cada linha segue este formato:

```text
[jaylog] debug <componente>: <evento>; atributo=valor
```

Exemplo:

```text
[jaylog] debug schedule: contexto de execução identificado; modo=task_scheduler; cadeia=python.exe < cmd.exe < svchost.exe
[jaylog] debug schedule: consulta XML iniciada; comando=`schtasks /query /xml`; timeout=10s
[jaylog] debug schedule: análise sintática do XML concluída; tasks_lidas=42; tasks_elegíveis=3
[jaylog] debug schedule: consolidação concluída; agendas_para_envio=5
```

Nos destinos `file` e `http`, a mensagem mantém o prefixo
`[jaylog] debug <componente>:` e o nível textual `DEBUG`. Esses registros não são
suprimidos por `JAYLOG_LOG_LEVEL`, pois o canal precisa diagnosticar inclusive a
configuração dos próprios handlers.

## Etapas observadas

| Componente | Informações registradas |
| --- | --- |
| `logger` | Configuração, seleção e construção dos handlers, `QueueListener`, reutilização e encerramento. |
| `entrypoint` | Origem usada para identificar o arquivo principal: executável congelado, `__main__.__file__` ou `sys.argv[0]`. |
| `git` | Comandos executados, diretório de busca, timeout, código de saída e resumo da detecção. |
| `host` | Elegibilidade, coleta do ambiente, cache do snapshot e entrega ao endpoint. |
| `metrics` | Inicialização do sampler, coleta, ocupação do buffer, lotes HTTP e amostra final. |
| `schedule` | Modo de execução, entrypoint, consulta XML, parsing, associação de tasks, consolidação e envio. |
| `http` | Início e término das requisições de logs, status HTTP, exceções e solicitação de sincronização do host. |
| `file` | Rotação do arquivo e aplicação da política de retenção. |
| `screenshot` | Elegibilidade, resultado, tamanho, qualidade, escala ou falha da captura. |
| `safe` | Exceções que um coletor suprimiu para proteger a aplicação. |

## Segurança dos diagnósticos

O canal não imprime:

- chaves de API;
- conteúdo dos registros da aplicação;
- payloads completos;
- corpos integrais de respostas HTTP;
- conteúdo dos arquivos inspecionados.

Caminhos locais, nomes de serviço, endpoints, comandos sem credenciais, códigos
HTTP e descrições de exceções podem aparecer porque são necessários para o
diagnóstico. Antes de compartilhar a saída externamente, trate esses valores
como informações operacionais do ambiente.

## Diagnóstico do Task Scheduler

Para investigar a sincronização de agendas no Windows, habilite inicialmente
somente o console:

```powershell
$env:JAYLOG_DEBUG = "true"
$env:JAYLOG_DEBUG_HANDLERS = "console"
```

Execute a task e acompanhe, nesta ordem:

1. `modo=task_scheduler` na identificação do contexto;
2. caminho do entrypoint identificado;
3. execução de `schtasks /query /xml`;
4. quantidade de tasks lidas e elegíveis;
5. tasks associadas ao entrypoint e agendas extraídas;
6. consolidação do payload;
7. requisição e resposta de `POST /logs/host-schedules`.

Se a sequência parar, a última mensagem identifica a etapa que impediu o envio.
Veja as regras de associação e os limites em
[Agendas do Task Scheduler](../producao/agendas-do-task-scheduler.md).

## Desativação

Remova as variáveis ou defina:

```env
JAYLOG_DEBUG=false
```

Não use o diagnóstico permanentemente apenas por conveniência. Em produção,
habilite-o pelo período necessário para investigar o incidente e depois o
desative, principalmente quando `file` ou `http` estiver selecionado.
