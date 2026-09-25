# Variáveis de Ambiente

As variáveis usam o prefixo `JAYLOG_`. Podem ser definidas no ambiente do sistema ou em um arquivo `.env` / `.env.logging` na raiz do projeto (veja [Arquivos .env](arquivos-env.md)).

| Variável                        | obrigatório? | Padrão    | Descrição                                                               |
| ------------------------------- | ------------ | --------- | ----------------------------------------------------------------------- |
| `JAYLOG_APP_NAME`               | SIM          | `null`    | Nome do serviço/bot (usado no nome do arquivo de log)                   |
| `JAYLOG_LOG_DIR`                | NÃO          | `null`    | Caminho do diretório onde os arquivos de log serão salvos. Se omitido, o handler de arquivo é desativado |
| `JAYLOG_LOG_LEVEL`              | NÃO          | `INFO`    | Nível mínimo de log (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`)   |
| `JAYLOG_LOG_MAX_BYTES`          | NÃO          | `5242880` | Tamanho máximo do arquivo de log antes de rotacionar (bytes)            |
| `JAYLOG_LOG_BACKUP_COUNT`       | NÃO          | `5`       | Quantidade de arquivos de backup mantidos após rotação                  |
| `JAYLOG_LOG_RETENTION_DAYS`     | NÃO          | `7`       | Dias para manter arquivos de log antigos                                |
| `JAYLOG_LOG_CONSOLE_ENABLED`    | NÃO          | `true`    | Habilita a saída de log no console (`true`/`false`)                     |
| `JAYLOG_LOG_CONSOLE_COLOR`      | NÃO          | `null`    | Força (`true`) ou desliga (`false`) as cores no console. Se omitido, detecta automaticamente o suporte do terminal |
| `JAYLOG_LOG_HTTP_TIMEOUT`       | NÃO          | `5.0`     | Timeout em segundos para o envio HTTP                                   |
| `JAYLOG_LOG_HTTP_ENDPOINT`      | NÃO          | `null`    | URL do endpoint que receberá os logs                                    |
| `JAYLOG_LOG_HTTP_API_KEY`       | NÃO          | `null`    | Chave de autenticação enviada no header `x-api-key`                     |
| `JAYLOG_LOG_HTTP_PROXY`         | NÃO          | `null`    | URL do proxy para o envio HTTP (ex: `http:\\user:password@server:port`) |
| `JAYLOG_LOG_HTTP_VERIFY`        | NÃO          | `false`   | `true` ou caminho para o bundle de CA usado para validar TLS            |
| `JAYLOG_LOG_SCREENSHOT_ENABLED` | NÃO          | `false`   | Captura screenshot no momento do log (`true`/`false`, apenas Windows)   |
| `JAYLOG_HOST_REPORT_ENABLED`    | NÃO          | `true`    | Envia uma fotografia do ambiente no startup                             |
| `JAYLOG_HOST_HTTP_ENDPOINT`     | NÃO          | derivado  | Override para o endpoint de host; por padrão `/logs/add` vira `/logs/host` |
| `JAYLOG_HOST_REPORT_TIMEOUT`    | NÃO          | `2 × HTTP_TIMEOUT` | Timeout do POST de ambiente                                     |
| `JAYLOG_HOST_GIT_ENABLED`       | NÃO          | `true`    | Coleta metadados do repositório Git                                     |
| `JAYLOG_HOST_GIT_DIRTY_ENABLED` | NÃO          | `true`    | Coleta se há arquivos alterados                                         |
| `JAYLOG_HOST_GIT_REMOTE_ENABLED`| NÃO          | `true`    | Coleta a URL remota sem credenciais                                     |
| `JAYLOG_HOST_GIT_TIMEOUT`       | NÃO          | `3.0`     | Timeout, em segundos, de cada chamada ao Git                            |
| `JAYLOG_HOST_GIT_DIR`           | NÃO          | `null`    | Diretório inicial para localizar o repositório                          |
| `JAYLOG_HOST_METRICS_ENABLED`   | NÃO          | `true`    | Envia uso de CPU, memória e disco a cada intervalo (exige `HOST_REPORT_ENABLED`) |
| `JAYLOG_HOST_METRICS_INTERVAL`  | NÃO          | `60`      | Segundos entre amostras de métricas (mínimo `10`)                       |
| `JAYLOG_HOST_METRICS_HTTP_ENDPOINT` | NÃO      | derivado  | Override para o endpoint de métricas; por padrão `/logs/add` vira `/logs/host-metrics` |

!!! important

    **`HTTP_ENDPOINT`** e **`HTTP_API_KEY`** (opcionais)

    - A configuração de **HTTP_ENDPOINT** e **HTTP_API_KEY** não precisa ser feita em ambiente local ou de desenvolvimento.
    - Se apenas uma das duas variáveis for definida, o envio HTTP é ignorado.
    - Caso a aplicação execute em um ambiente que usa um proxy NTLM, defina `JAYLOG_LOG_HTTP_PROXY`.

## Próximo passo

Veja como organizar essas variáveis em [Arquivos .env](arquivos-env.md).
