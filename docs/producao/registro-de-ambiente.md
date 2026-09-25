# Registro de Ambiente (Host Reporting)

Na versão 0.3, `configure()` inicia em segundo plano um único `POST` JSON para o endpoint de host por serviço. O corpo dos logs continua compatível com a linha 0.2.x; apenas os headers `x-jaylog-protocol` e `x-jaylog-run-id` são adicionados. O protocolo está na versão `3` desde a 0.3.0a4, que acrescentou os limites da máquina ao registro e as [métricas de recursos](metricas-de-recursos.md). O `run_id` permite ao backend ligar os logs à execução que os produziu.

O endpoint é derivado automaticamente trocando o último segmento de `JAYLOG_LOG_HTTP_ENDPOINT`: `https://api.example/logs/add` vira `https://api.example/logs/host`. Use `JAYLOG_HOST_HTTP_ENDPOINT` somente quando o backend publicar a rota em outro endereço.

O registro inclui sistema operacional, limites da máquina (CPUs, RAM e disco), modo de execução, Python, virtualenv, Git e diretórios de execução. A URL remota do Git tem sempre a credencial embutida removida antes de sair da máquina. Falhas de coleta ou de rede não interrompem a aplicação: o envio tenta novamente com backoff. Se o backend aceitar um log mas devolver `x-jaylog-host-required: 1`, o jaylog reenvia o registro de host com debounce de 30 segundos; o log já foi aceito e não é reenviado.

`JAYLOG_LOG_HTTP_VERIFY=false` continua sendo o padrão desta versão para não interromper instalações atrás de proxies corporativos. Para validar TLS, use `JAYLOG_LOG_HTTP_VERIFY=true` ou informe o caminho do CA bundle corporativo. O padrão passará a `true` na 0.4.0.

## Campos do Hosting Report

O payload enviado para `/logs/host` é uma fotografia do processo no momento em
que o jaylog é configurado. Campos que não puderem ser coletados são enviados
como `null`; isso diferencia uma informação desconhecida de um cliente que não
suporta o campo. `service` identifica o serviço informado em `configure()` e
`run_id` liga este retrato aos logs da mesma execução.

| Nome original (inglês) | Nome em português | O que informa |
| --- | --- | --- |
| `run_id` | ID da execução | UUID criado no início do processo para relacionar o report e todos os logs daquela execução. |
| `service` | Serviço | Nome lógico do serviço configurado no jaylog. |
| `protocol_version` | Versão do protocolo | Versão do formato de comunicação usado entre o cliente e o backend. |
| `jaylog_version` | Versão do jaylog | Versão instalada da biblioteca que enviou o report. |
| `started_at` | Iniciado em | Data e hora UTC em que o processo começou. |
| `collected_at` | Coletado em | Data e hora UTC em que a fotografia do ambiente foi coletada. |
| `hostname` | Nome da máquina | Nome do computador ou servidor que executa o processo. |
| `username` | Usuário | Usuário do sistema operacional que iniciou o processo. |
| `ipv4` | IPv4 | Endereço IPv4 local identificado para a máquina. |
| `cpu_count` | Quantidade de CPUs | CPUs lógicas da máquina; é o teto dos gráficos de CPU. |
| `memory_total_bytes` | RAM total | Memória física total da máquina, em bytes. |
| `disk_total_bytes` | Tamanho do disco | Capacidade, em bytes, do disco onde está o script ou executável. |
| `disk_path` | Disco medido | Raiz do disco medido, como `C:\` ou `/`. |
| `os_system` | Sistema operacional | Família do sistema operacional, como `Windows`, `Linux` ou `Darwin`. |
| `os_release` | Release do sistema | Release do sistema operacional, como a versão do kernel. |
| `os_version` | Versão do sistema | Detalhe textual da versão fornecido pelo sistema operacional. |
| `machine` | Arquitetura da máquina | Arquitetura de hardware, como `x86_64` ou `ARM64`. |
| `execution_mode` | Modo de execução | Como o processo foi iniciado, por exemplo script, serviço ou tarefa agendada. |
| `execution_detail` | Detalhe da execução | Informação complementar sobre o modo de execução identificado. |
| `session_id` | ID da sessão | Identificador da sessão do sistema operacional, quando disponível. |
| `process_id` | ID do processo | PID do processo que está emitindo os logs. |
| `parent_process_name` | Processo pai | Nome do processo que iniciou o processo atual. |
| `python_version` | Versão do Python | Versão do interpretador Python em uso. |
| `python_implementation` | Implementação do Python | Implementação do interpretador, como `CPython` ou `PyPy`. |
| `python_executable` | Executável do Python | Caminho do executável Python que está rodando a aplicação. |
| `python_frozen` | Python congelado | Indica se a aplicação foi empacotada como executável, por exemplo com PyInstaller. |
| `venv_active` | Ambiente virtual ativo | Indica se o processo usa um ambiente virtual Python. |
| `venv_kind` | Tipo de ambiente virtual | Tipo do ambiente virtual detectado, como `venv`, `virtualenv` ou Conda. |
| `venv_path` | Caminho do ambiente virtual | Diretório do ambiente virtual ativo. |
| `git_available` | Git disponível | Indica se o executável Git está acessível no ambiente. |
| `git_version` | Versão do Git | Versão do executável Git encontrado. |
| `git_repo` | Repositório Git | Indica se o ponto de partida da aplicação pertence a um repositório Git. |
| `git_root` | Raiz do repositório | Diretório raiz do repositório Git. |
| `git_branch` | Branch Git | Branch atualmente selecionada; fica nulo em um commit destacado (*detached HEAD*). |
| `git_commit` | Commit Git | Hash completo do último commit (`HEAD`). |
| `git_commit_short` | Commit Git curto | Forma abreviada do hash do último commit. |
| `git_commit_msg` | Mensagem do commit | Assunto da mensagem do último commit. |
| `git_commit_datetime` | Data e hora do commit | Data e hora registrada pelo Git para o último commit, em ISO 8601 e com o fuso horário do commit. |
| `git_dirty` | Alterações locais | Indica se há arquivos modificados, adicionados ou removidos ainda não confirmados no Git. |
| `git_remote_url` | URL remota do Git | URL do remote `origin`; credenciais embutidas são removidas antes do envio. |
| `cwd` | Diretório de trabalho atual | Diretório atual do processo no momento da coleta. |
| `entrypoint` | Ponto de entrada | Diretório do script ou executável que iniciou a aplicação. |

## Próximo passo

Além da fotografia do início, o jaylog acompanha o uso de CPU, memória e disco a cada minuto enquanto o processo roda — veja [Métricas de Recursos](metricas-de-recursos.md).
