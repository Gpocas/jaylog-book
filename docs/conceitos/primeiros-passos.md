# Primeiros Passos

!!! important

    A partir da versão 0.2.2, `configure()` **deve** ser chamado antes de `get_logger()`. Chamar `get_logger()` sem configuração prévia lança uma exceção.

O fluxo básico de uso do jaylog é sempre o mesmo:

1. Criar uma instância de `JaylogSettings` (as configurações podem vir de variáveis de ambiente, de um arquivo `.env`, ou serem passadas diretamente no construtor).
2. Chamar `configure()` com essas configurações.
3. Obter o logger com `get_logger()` e usá-lo normalmente.

## Exemplo

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

logger.info("Meu primeiro log")
```

Por padrão, o jaylog carrega automaticamente o arquivo `.env.logging` da raiz do projeto. As variáveis usam o prefixo `JAYLOG_` — a lista completa está em [Variáveis de Ambiente](../configuracao/variaveis-de-ambiente.md).

Apenas `JAYLOG_APP_NAME` é obrigatório: ele nomeia o serviço/bot e dá nome ao arquivo de log. Se `JAYLOG_LOG_DIR` não for definido, o jaylog registra apenas no console.

## Próximo passo

Veja a lista completa de [Variáveis de Ambiente](../configuracao/variaveis-de-ambiente.md) disponíveis.
