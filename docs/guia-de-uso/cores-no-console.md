# Cores no Console

As cores são ligadas automaticamente quando o terminal suporta ANSI. A detecção cobre:

- **Windows**: o modo *virtual terminal* do console é habilitado em tempo de execução, o que faz as cores funcionarem também no `cmd.exe`/PowerShell rodando no console legado (`conhost`) do Windows 10 — antes só saía colorido no Windows Terminal.
- **Saída redirecionada** (`python main.py > saida.txt`, pipes, serviços sem console): as cores são desligadas, para o arquivo não ficar com lixo do tipo `←[32m`.
- **Consoles antigos** que não suportam ANSI de jeito nenhum: o log sai em texto limpo, com o mesmo alinhamento.
- As convenções `NO_COLOR` e `FORCE_COLOR` são respeitadas.

## Forçando um comportamento

Use `JAYLOG_LOG_CONSOLE_COLOR`:

__*.env.logging*__
```env
JAYLOG_APP_NAME=meu-bot
JAYLOG_LOG_CONSOLE_COLOR=false
```

Ou direto no código:

```python
configure(JaylogSettings(app_name="meu-bot", log_console_color=False))
```

!!! tip

    `JAYLOG_LOG_CONSOLE_COLOR=true` força as cores mesmo com a saída redirecionada — útil quando o log é consumido por uma ferramenta que entende ANSI (ex: `... | less -R`).

## Próximo passo

Com os cenários de uso cobertos, avance para [Produção](../producao/registro-de-ambiente.md) para preparar o jaylog para rodar em uma frota de bots.
