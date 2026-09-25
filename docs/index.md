---
title: Início
---

# jaylog

Bem-vindo à documentação do **jaylog**, uma biblioteca de logging para Python com rotação de arquivos, saída colorida no console, envio HTTP para um endpoint remoto e registro automático do ambiente de execução (host reporting).

## O que você vai encontrar aqui

Esta documentação foi organizada para guiar você desde os conceitos fundamentais até a configuração avançada para produção. A sequência recomendada de leitura é:

| Etapa | Seção | O que você vai aprender |
|-------|-------|--------------------------|
| 1 | **Conceitos Básicos** | O que é o jaylog, como instalar e como configurar seu primeiro logger |
| 2 | **Configuração** | As variáveis de ambiente disponíveis e como organizá-las em arquivos `.env` |
| 3 | **Guia de Uso** | Os cenários mais comuns: arquivo único, múltiplos módulos, múltiplos loggers e cores no console |
| 4 | **Produção** | Registro de ambiente (host reporting), métricas de CPU/memória/disco, gestão de secrets e reconfiguração sem repetir argumentos |
| 5 | **Referência** | A API pública do pacote, para consulta rápida |

## Instalação rápida

```bash
pip install -U --no-cache-dir jaylog
```

## Exemplo mínimo

```python
from jaylog import JaylogSettings, configure, get_logger

configure(JaylogSettings())

logger = get_logger()
logger.info("Olá, jaylog!")
```

!!! important

    `configure()` deve ser chamado antes de usar o logger. `get_logger()` pode ser chamado antes — ele devolve um proxy preguiçoso e só levanta uma exceção no primeiro uso efetivo (`logger.info(...)`, etc.), caso `configure()` ainda não tenha rodado até lá.

Continue para [O que é o jaylog](conceitos/visao-geral.md) para entender o problema que a biblioteca resolve.
