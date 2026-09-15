# Registro de Ambiente (Host Reporting)

Na versão 0.3, `configure()` inicia em segundo plano um único `POST` JSON para o endpoint de host por serviço. O corpo dos logs continua compatível com a linha 0.2.x; apenas os headers `x-jaylog-protocol: 2` e `x-jaylog-run-id` são adicionados. O `run_id` permite ao backend ligar os logs à execução que os produziu.

O endpoint é derivado automaticamente trocando o último segmento de `JAYLOG_LOG_HTTP_ENDPOINT`: `https://api.example/logs/add` vira `https://api.example/logs/host`. Use `JAYLOG_HOST_HTTP_ENDPOINT` somente quando o backend publicar a rota em outro endereço.

O registro inclui sistema operacional, modo de execução, Python, virtualenv, Git e diretórios de execução. A URL remota do Git tem sempre a credencial embutida removida antes de sair da máquina. Falhas de coleta ou de rede não interrompem a aplicação: o envio tenta novamente com backoff. Se o backend aceitar um log mas devolver `x-jaylog-host-required: 1`, o jaylog reenvia o registro de host com debounce de 30 segundos; o log já foi aceito e não é reenviado.

`JAYLOG_LOG_HTTP_VERIFY=false` continua sendo o padrão desta versão para não interromper instalações atrás de proxies corporativos. Para validar TLS, use `JAYLOG_LOG_HTTP_VERIFY=true` ou informe o caminho do CA bundle corporativo. O padrão passará a `true` na 0.4.0.

## Próximo passo

Em produção, credenciais como `JAYLOG_LOG_HTTP_API_KEY` normalmente não ficam no código nem no `.env` versionado — veja [Secrets em Produção](secrets.md).
