# Heartbeat de Serviço

A partir da 0.3.0a8, você pode avisar o backend de que o loop do seu serviço está **progredindo**, mesmo quando ele não emite nenhum log. Sem isso, o painel decide que um serviço está parado só pelo horário do último log: um bot em loop silencioso aparece como "Parado" em 15 minutos e, sem logs por 45 dias, é excluído por inatividade.

## Como usar

Chame `jaylog.heartbeat()` **dentro do loop**, uma vez por iteração:

```python
import jaylog
from jaylog import JaylogSettings, configure

configure(JaylogSettings(app_name="meu-bot"))

while True:
    processar_fila()
    jaylog.heartbeat()
```

A chamada é barata (só incrementa um contador) e nunca bloqueia nem levanta exceção. O envio acontece numa thread própria, no máximo uma vez por `JAYLOG_HOST_HEARTBEAT_INTERVAL` (60 s por padrão) e só se houve `heartbeat()` desde o último envio.

O backend passa a considerar o serviço ativo pelo **mais recente** entre o último log e o último heartbeat. Quem não chama `heartbeat()` continua exatamente como antes: vale o último log.

!!! warning "Coloque a chamada onde o trabalho acontece"

    O heartbeat existe para detectar loop **travado**. Se você o chamar de uma thread separada que continua viva enquanto o loop principal está preso, o serviço parecerá saudável sem estar.

## Vários serviços no mesmo processo

Sem argumento, `heartbeat()` vale para o **primeiro** logger registrado em `configure()` (a mesma regra do `get_logger()` sem nome). Para outro serviço, informe o `app_name`:

```python
configure([JaylogSettings(app_name="ORDERS"), JaylogSettings(app_name="BILLING")])

jaylog.heartbeat()            # ORDERS
jaylog.heartbeat("BILLING")  # BILLING
```

Cada serviço usa o endpoint e a API key do próprio `JaylogSettings`. Um nome desconhecido, ou um serviço com o heartbeat desativado, é ignorado sem erro.

## Configuração

| Variável | Padrão | Descrição |
| --- | --- | --- |
| `JAYLOG_HOST_HEARTBEAT_ENABLED` | `true` | Liga o heartbeat. Só vale se `JAYLOG_HOST_REPORT_ENABLED` também estiver ligado |
| `JAYLOG_HOST_HEARTBEAT_INTERVAL` | `60` | Segundos entre envios (mínimo `10`) |
| `JAYLOG_HOST_HEARTBEAT_HTTP_ENDPOINT` | derivado | Override do endpoint; por padrão `/logs/add` vira `/logs/heartbeat` |

!!! note

    O painel considera um serviço parado após 15 minutos sem atividade. Um `JAYLOG_HOST_HEARTBEAT_INTERVAL` próximo ou acima disso faria o serviço aparecer como parado entre um envio e outro.

## Quando o backend não suporta

Se o backend responder 404/405 (versão anterior à rota) ou rejeitar a chave, o jaylog avisa **uma vez** no `stderr` e desativa o heartbeat **daquele serviço** pelo resto do processo. O restante do jaylog não é afetado. Falhas de rede e respostas 429/5xx só mantêm o beat pendente para o próximo ciclo.

Ao encerrar (`shutdown()` ou fim do processo), um último envio sai se houver beat pendente.
