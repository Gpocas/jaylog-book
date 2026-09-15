# Arquivos .env

Por padrão, o jaylog procura um arquivo `.env.logging` na raiz do projeto. Isso permite manter a configuração de logging separada de outras variáveis de ambiente da aplicação.

__*.env.logging*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_DIR=C:\logs
```

__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

configure(JaylogSettings())

logger = get_logger()

logger.info("Arquivo Único")
```

## Apenas console (sem arquivo de log)

Basta omitir `JAYLOG_LOG_DIR`. O handler de console fica ativo por padrão.

__*.env.logging*__
```env
JAYLOG_APP_NAME=meu-bot
```

## Alterando o caminho padrão do `.env`

Use o argumento `_env_file` para apontar para um arquivo diferente:

__*development.env*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_DIR=C:\logs
```

__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

configure(JaylogSettings(_env_file="development.env"))
logger = get_logger()

logger.info("Alterando caminho padrão do .env")
```

Esse mecanismo é a base para separar ambientes de desenvolvimento, homologação e produção — veja [Reconfigurando sem Repetir Argumentos](../producao/reconfigurando.md) para um exemplo mais completo com múltiplos ambientes.

## Próximo passo

Com a configuração básica dominada, avance para o [Guia de Uso](../guia-de-uso/cenarios-comuns.md) para ver os cenários mais comuns de aplicação.
