# Reconfigurando sem Repetir Argumentos

`reconfigure()` recria a configuração com os mesmos argumentos do construtor, aplicando apenas os overrides informados. Vale tanto para campos quanto para os argumentos de configuração do pydantic-settings (`_env_file`, `_secrets_dir`, `_case_sensitive`, `_env_prefix`, ...):

```python
base = JaylogSettings(app_name="meu-bot", log_level="DEBUG")

homolog = base.reconfigure(_env_file="homolog.env")
prod = base.reconfigure(_env_file="producao.env", log_level="WARNING")
# app_name preservado nos dois; só o que foi informado muda
```

É sobre esse mecanismo que o `reload_secrets()` (veja [Secrets em Produção](secrets.md)) é construído — ele é só um `reconfigure(_secrets_dir=...)` com validação.

## Próximo passo

Para consultar rapidamente as funções e classes públicas do pacote, veja a [Referência da API](../referencia/api.md).
