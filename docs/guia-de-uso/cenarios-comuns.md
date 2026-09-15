# Cenários Comuns

Existem alguns cenários diferentes onde a utilização dessa lib pode mudar. Abaixo estão os cenários mapeados e como realizar a configuração para cada um.

## Arquivo único

O cenário mais simples: um `main.py`, um arquivo de log.

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
__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

configure(JaylogSettings())

logger = get_logger()

logger.info("Saída apenas no console")
```

## Múltiplos arquivos (mesmo logger em vários módulos)

Depois de chamar `configure()` uma vez no ponto de entrada, qualquer módulo pode obter o mesmo logger com `get_logger()`, sem reconfigurar nada.

__*.env.logging*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_DIR=C:\logs
```

__*main.py*__
```python
from jaylog import JaylogSettings, get_logger, configure
from parse import parse_csv

configure(JaylogSettings())

logger = get_logger()

logger.info("Múltiplos Arquivos - main.py")
parse_csv()
```

__*parse.py*__
```python
from jaylog import get_logger

logger = get_logger()


def parse_csv():
    logger.info("Múltiplos Arquivos - parse.py")
```

## Próximo passo

Quando um mesmo processo precisa separar logs de serviços distintos, veja [Múltiplos Loggers Nomeados](multiplos-loggers.md).
