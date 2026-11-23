# Plano: ingestão de leituras

## Dados a persistir

Consultar `spec.md` para o contrato HTTP e as regras comportamentais.

### Dispositivo

- `device_id`, inicialmente nulo até o primeiro uso do token.
- Hash do token, nunca o token em texto no banco.
- Fator de calibração de 450 pulsos/litro em uma linha por dispositivo.
- O `label` não é persistido nesta versão.

### Leitura

- `device_id`, `pulse_total`, `boot_id`, `seq` e horário UTC de recebimento pela API.
- Pulsos brutos como fonte permanente da leitura.
- Consumo calculado em litros como dado derivado para log; os pulsos continuam sendo a fonte de verdade.
- Identificação de reinício em relação ao `boot_id` anterior.

### Garantias de unicidade

- O banco garante unicidade de `(device_id, boot_id, seq)`.
- O banco garante unicidade do hash do token.
- Um reenvio não cria nova leitura efetiva nem novo consumo.
- O cálculo do intervalo usa `seq` dentro do mesmo `boot_id`, não horário de chegada.
- O primeiro `boot_id` conhecido pode usar `pulse_total` como consumo inicial.

### Schema

- O PostgreSQL é criado pelo único arquivo `api/schema.sql`, sem migrations nesta versão.
- Detalhes do arquivo são definidos em `docs/decisoes/schema_banco.md`.

## Estrutura da API

Seguir `docs/decisoes/estrutura_api.md` e manter o menor número possível de dependências.

- `api/app/main.py`: cria o FastAPI e registra `POST /readings`.
- `api/app/config.py`: lê configuração e segredos por variáveis de ambiente.
- `api/app/routes/ingestion.py`: valida HTTP e traduz resultados em códigos da especificação.
- `api/app/services/ingestion.py`: aplica vínculo de token, idempotência, conversão por 450 pulsos/litro, consumo por `seq` e reinício.
- `api/app/repositories/readings.py`: única camada que conversa com PostgreSQL.
- `api/app/schemas/readings.py`: valida `device_id`, `pulse_total`, `boot_id` e `seq`.
- Usar a biblioteca padrão para gerar e comparar o hash do token. Não adicionar biblioteca criptográfica ou ORM sem necessidade.
- Erros de validação devem ser configurados para `422 Unprocessable Entity`, usando o comportamento padrão do FastAPI.

## Script administrativo

- Criar um script mínimo em `api/` que gera um token aleatório seguro por dispositivo, calcula seu hash e grava o registro sem `device_id` vinculado.
- O script mostra o token uma única vez no terminal e não o persiste em texto no banco.
- A gravação do firmware é feita fora da API pela operação de provisionamento.

## Estrutura do firmware

Seguir `docs/decisoes/estrutura_modulos.md`:

- Código em `esp32/`, independente da API e com suas próprias dependências e testes.
- Contagem do YF-S201 por interrupção no GPIO e contador cumulativo desde o boot.
- Gerar `boot_id` novo em cada inicialização.
- Incrementar `seq` a cada leitura programada dentro do boot.
- Enviar `device_id`, `pulse_total`, `boot_id` e `seq`; não enviar litros ou `timestamp`.

## Configuração local do firmware

- Criar um arquivo de exemplo versionado com campos fictícios para:
  - token do dispositivo;
  - `device_id` ou instrução para obter o MAC/chip ID;
  - endereço da API;
  - credenciais de Wi-Fi.
- O arquivo real de configuração fica fora do repositório e não pode conter valores secretos versionados.
- O `device_id` enviado é obtido do chip/MAC do ESP32, não informado manualmente em produção.

## Comportamento do firmware

- Fazer `POST` por HTTPS em `/readings`, validando o certificado do servidor.
- Enviar `Authorization: Bearer <token>`.
- Definir o intervalo de envio como parâmetro de configuração.
- Sem conexão ou com `503`, manter a leitura no buffer e tentar novamente sem zerar `pulse_total` ou `seq`.
- Todo `2xx` permite remover a leitura correspondente do buffer, inclusive `200` de duplicata.
- Não calcular litros no firmware.

## Logs da API

A cada leitura processada, registrar:

- `device_id`, `boot_id`, `seq` e estado `new`, `duplicate`, `rejected` ou `reboot`;
- `pulse_total` recebido, fator 450 pulsos/litro e litros calculados;
- consumo em litros do intervalo e horário UTC de recebimento;
- motivo de rejeição para token ou vínculo inválido.

Não registrar o token, seu hash ou credenciais.

## Estratégia de testes

### API

- Testes unitários em `api/tests/services/`: vínculo, hash do token, conversão por 450 pulsos/litro, diferença por `seq`, reinício e leitura atrasada.
- Testes de rota em `api/tests/routes/`: `201`, duplicata `200`, corpo `422`, token `401`, vínculo conflitante `403` e `503`.
- Testes de repositório em `api/tests/repositories/`: persistência e unicidade da tripla `(device_id, boot_id, seq)`.
- Testes de integração: primeiro uso vincula token; outro `device_id` é rejeitado; reenvio não duplica consumo.

### Firmware

- Testes unitários de contador, `boot_id`, `seq` e obtenção do `device_id`.
- Testes de serialização sem `timestamp` e com os quatro campos do contrato.
- Simulação de falha de rede e confirmação de reenvio após `503` ou ausência de conexão.
- Teste em hardware com volume conhecido usando 450 pulsos/litro.

### Critério transversal

- Não persistir token em texto, nem registrar segredo em logs ou arquivo versionado.
- Não contar uma leitura duas vezes em reenvios.
