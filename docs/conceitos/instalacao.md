# Instalação

Instale o jaylog com `pip`:

```bash
pip install -U --no-cache-dir jaylog
```

Se o seu projeto usa [uv](https://docs.astral.sh/uv/):

```bash
uv add jaylog
```

O `pip`/`uv` instala junto as dependências — entre elas o [psutil](https://pypi.org/project/psutil/), usado nas [métricas de recursos](../producao/metricas-de-recursos.md). Ele tem pacote pronto para Windows e Linux em todas as versões de Python suportadas (3.10 a 3.13), sem precisar de compilador.

## Próximo passo

Configure seu primeiro logger em [Primeiros Passos](primeiros-passos.md).
