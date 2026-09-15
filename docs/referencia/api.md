# API Pública

Referência rápida do que é exportado por `import jaylog`.

## `configure(settings)`

Inicializa o jaylog. Recebe uma `JaylogSettings` ou uma lista de `JaylogSettings` (para [múltiplos loggers nomeados](../guia-de-uso/multiplos-loggers.md)). Deve ser chamada antes de `get_logger()`.

```python
from jaylog import JaylogSettings, configure

configure(JaylogSettings(app_name="meu-bot"))
```

## `get_logger(name=None)`

Retorna o logger registrado por `configure()`. Sem argumento, retorna o primeiro logger registrado.

```python
from jaylog import get_logger

logger = get_logger()
logger.info("mensagem")
```

## `shutdown()`

Encerra os handlers e os reporters de ambiente, aguardando o flush dos logs pendentes. Útil para garantir que tudo foi gravado/enviado antes do processo terminar.

## `JaylogSettings`

Modelo de configuração baseado em `pydantic-settings`. Todos os campos podem vir de variáveis de ambiente com prefixo `JAYLOG_` (veja [Variáveis de Ambiente](../configuracao/variaveis-de-ambiente.md)), de um arquivo `.env` (veja [Arquivos .env](../configuracao/arquivos-env.md)), ou serem passados diretamente no construtor.

Métodos relevantes:

- `reload_secrets()` — recarrega os campos sensíveis a partir de `secrets_dir` (veja [Secrets em Produção](../producao/secrets.md)).
- `reconfigure(**overrides)` — recria a configuração aplicando apenas os overrides informados (veja [Reconfigurando sem Repetir Argumentos](../producao/reconfigurando.md)).

## `__version__` e `PROTOCOL_VERSION`

- `__version__`: versão instalada do pacote.
- `PROTOCOL_VERSION`: versão do protocolo de comunicação com o backend de logs (enviada no header `x-jaylog-protocol`).
