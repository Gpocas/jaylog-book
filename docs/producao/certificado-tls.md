# Certificado TLS (HTTPS)

`JAYLOG_LOG_HTTP_VERIFY` controla a verificação do certificado do servidor em **todos** os envios HTTP do jaylog: logs, [registro de ambiente](registro-de-ambiente.md), [métricas](metricas-de-recursos.md), [heartbeat](heartbeat.md) e [agendas](agendas-do-task-scheduler.md).

## Valores aceitos

| Valor | Efeito |
| --- | --- |
| `true` (também `1`, `yes`, `on`) | Verifica o certificado com o bundle padrão do `requests` (`certifi`). Se `REQUESTS_CA_BUNDLE` estiver definida, ela é usada no lugar. |
| `/caminho/corp-ca.pem` | Verifica o certificado com esse CA bundle (arquivo PEM ou diretório preparado com `c_rehash`). |
| `false` (também `0`, `no`, `off`) | **Não** verifica o certificado. Os envios ficam expostos a interceptação. |

Maiúsculas e minúsculas são indiferentes (`TRUE`, `False`), e espaços nas pontas são ignorados.

!!! warning "Padrão da 0.3.x: `false`"

    Na linha 0.3 o padrão continua `false` para não interromper, no dia do upgrade, bots atrás de proxy corporativo com inspeção TLS. **O padrão passa a ser `true` na 0.4.0.** Para se antecipar, defina `JAYLOG_LOG_HTTP_VERIFY=true` ou o caminho do CA bundle corporativo já agora.

## Proxy corporativo com inspeção TLS

Quando o proxy da empresa intercepta o HTTPS, o certificado que chega ao bot é assinado pela **CA corporativa**, e não pela CA pública do servidor. Com `true`, a verificação falha, porque o bundle do `certifi` não conhece essa CA.

A solução é apontar para um arquivo com a CA corporativa:

__*.env.logging*__
```env
JAYLOG_LOG_HTTP_ENDPOINT=https://api.example/logs/add
JAYLOG_LOG_HTTP_PROXY=http://proxy.empresa:8080
JAYLOG_LOG_HTTP_VERIFY=C:\certs\corp-ca.pem
```

O arquivo pode ser o certificado raiz exportado em formato **Base-64 (PEM)**, por exemplo pelo `certmgr.msc` do Windows, ou fornecido pelo time de infraestrutura.

## Caminho inválido falha na inicialização

Se o valor não for booleano, ele é tratado como caminho e **precisa existir**. Caso contrário, `JaylogSettings()` levanta `ValidationError` logo no início da aplicação:

```text
log_http_verify
  Value error, log_http_verify: CA bundle não encontrado: C:\certs\corp-ca.pem
```

Valor vazio (`JAYLOG_LOG_HTTP_VERIFY=`) também é rejeitado. Assim um erro de digitação aparece na hora, em vez de derrubar em silêncio todos os envios HTTP.

## Avisos no `stderr`

Quando algum endpoint configurado usa `https://` e a verificação está desligada, `configure()` imprime **uma vez por processo** um destes avisos:

- **Padrão (nada configurado):** lembra que o certificado não está sendo verificado e que o padrão muda na 0.4.0.
- **`false` explícito:** informa que a verificação foi desativada por configuração.

O aviso `InsecureRequestWarning` do `urllib3` é silenciado **somente para o host dos endpoints do jaylog**. Requisições que a própria aplicação fizer para outros hosts sem verificação continuam gerando o aviso normalmente.

!!! note "Correções na 0.3.3"

    Até a 0.3.2, os valores vindos de variável de ambiente ou `.env` não eram convertidos para booleano: `JAYLOG_LOG_HTTP_VERIFY=true` (e também `=false`) era tratado como o **caminho** de um arquivo chamado `true`, e **todos** os envios HTTP falhavam em silêncio. Se você usava um desses valores, atualize para a 0.3.3. Configurações feitas em código (`JaylogSettings(log_http_verify=True)`) não eram afetadas.
