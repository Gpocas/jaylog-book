# Múltiplos Loggers Nomeados

Quando o projeto possui serviços distintos, passe uma lista para `configure()`. Cada entrada usa seu próprio `app_name` e grava em arquivos separados. O campo `[service]` é exibido automaticamente no formato do log quando há mais de um logger registrado.

__*.env.logging*__
```env
JAYLOG_LOG_DIR=C:\logs
```

__*main.py*__
```python
from jaylog import JaylogSettings, configure, get_logger

settings_order = JaylogSettings(app_name="ORDER-PROCESSOR")
settings_billing = JaylogSettings(app_name="BILLING")

configure([settings_order, settings_billing])

logger = get_logger("ORDER-PROCESSOR")  # ou get_logger() — retorna o primeiro registrado
billing_logger = get_logger("BILLING")

logger.info("Pedido recebido")
billing_logger.info("Fatura emitida")
```

## Próximo passo

Veja como o jaylog lida com [Cores no Console](cores-no-console.md).
