# O que é o jaylog

O jaylog é uma biblioteca de logging para Python pensada para bots e serviços que rodam sem supervisão constante — RPAs, jobs agendados, serviços Windows — onde não dá para simplesmente olhar o terminal para saber o que aconteceu.

Em cima do módulo padrão `logging` do Python, o jaylog adiciona:

- **Rotação de arquivos**: os logs são gravados em disco com tamanho máximo e quantidade de backups configuráveis, além de uma política de retenção por dias.
- **Console colorido**: saída legível no terminal, com detecção automática de suporte a ANSI (inclusive no `cmd.exe`/PowerShell legado do Windows).
- **Envio HTTP**: cada registro de log pode ser enviado para um endpoint remoto, para centralizar logs de uma frota de bots em um único lugar.
- **Registro de ambiente (host reporting)**: no início da execução, o jaylog coleta uma "fotografia" do ambiente — sistema operacional, versão do Python, virtualenv, informações do Git — e envia para um endpoint dedicado, permitindo saber *onde* e *como* cada processo está rodando.
- **Métricas de recursos**: enquanto o processo roda, o jaylog envia a cada minuto o uso de CPU, memória e disco da máquina e do próprio bot (incluindo os programas que ele abre), junto com os limites da máquina, para acompanhar o consumo em gráficos.
- **Múltiplos loggers nomeados**: quando um mesmo processo tem mais de um serviço lógico, cada um pode ter seu próprio `app_name` e arquivo de log.

## Por que não usar só o `logging` padrão?

O módulo `logging` do Python já resolve boa parte do problema, mas configurar rotação, cores, envio HTTP e coleta de ambiente do zero, de forma consistente entre vários bots, é repetitivo e propenso a erro. O jaylog empacota essas decisões em um único ponto de configuração (`JaylogSettings`), controlado por variáveis de ambiente com o prefixo `JAYLOG_`.

## Próximo passo

Veja como [instalar o jaylog](instalacao.md).
