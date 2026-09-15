# Secrets em Produção

!!! tip

    Em produção é recomendado usar um diretório específico para suas secrets, para que você possa reutilizar entre diferentes aplicações.

## Cenário 1 — secrets definidos em código (hardcode)

Passe `_secrets_dir` diretamente no construtor.

__*production.env*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_DIR=C:\logs
```
```bash
$ pwd
/foo/bar/secrets

$ ls -la
JAYLOG_LOG_HTTP_ENDPOINT
JAYLOG_LOG_HTTP_API_KEY
JAYLOG_LOG_HTTP_PROXY
```

__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

configure(JaylogSettings(_env_file="production.env", _secrets_dir="/foo/bar/secrets/"))
logger = get_logger()

logger.info("Mensagem de log")
```

## Cenário 2 — secrets definido no `.env`

Quando o diretório de secrets vem do próprio `.env` (via `JAYLOG_SECRETS_DIR`), é necessário chamar explicitamente `reload_secrets()` depois de construir as configurações:

__*production.env*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_DIR=C:\logs
JAYLOG_SECRETS_DIR=/foo/bar/secrets
```

__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

# Nesse caso é necessário usar a função de classe `reload_secrets`
# pois o diretório dos secrets foi passado via variável de ambiente
configure(JaylogSettings(_env_file="production.env").reload_secrets())
logger = get_logger()

logger.info("Mensagem de log")
```

`reload_secrets()` devolve uma **nova** instância preservando tudo que foi passado explicitamente no construtor, então dá para combinar configuração em código com secrets em disco:

```python
settings = JaylogSettings(app_name="meu-bot", log_level="DEBUG").reload_secrets()
configure(settings)  # app_name e log_level mantidos; endpoint/api_key vêm dos secrets
```

!!! note

    Valores passados no construtor têm prioridade sobre os secrets: um `log_http_api_key='...'` definido em código **não** é sobrescrito pelo arquivo em `secrets/`.

## Próximo passo

`reload_secrets()` é, na verdade, um caso especial de um mecanismo mais geral — veja [Reconfigurando sem Repetir Argumentos](reconfigurando.md).
