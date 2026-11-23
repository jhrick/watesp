# Especificação: ingestão de leituras

## Objetivo

Esta funcionalidade recebe leituras do ESP32, autentica o dispositivo e persiste os pulsos brutos no PostgreSQL. A API calcula o consumo em litros e registra o resultado, seguindo o contrato desta especificação e das decisões existentes.

## Escopo

- Rota HTTPS `POST /readings` para o ESP32.
- Autenticação de dispositivo por token e vínculo com `device_id` no primeiro uso.
- Persistência idempotente de `device_id`, `pulse_total`, `boot_id` e `seq`.
- Conversão de pulsos em litros e cálculo do consumo entre leituras.
- Tratamento de leituras atrasadas e reinícios do dispositivo.
- Logs da API como observação dos resultados nesta versão.

## Fora do escopo

- Rotas `GET`, telas de consulta e interface de usuário.
- Autenticação de usuários, perfis, casas, residentes e alertas.
- Telas ou rotas para cadastro de casas, dispositivos ou moradores.
- Campo `label` neste fluxo; a associação à casa fica para uma versão futura.
- Persistência do contador no ESP32 para reter pulsos em queda de energia.

## Contrato da rota

- Método: `POST`.
- Caminho: `/readings`, sem prefixo.
- Cabeçalhos:
  - `Authorization: Bearer <token>`.
  - `Content-Type: application/json`.
- Corpo:

```json
{
  "device_id": "<identificador-do-chip-ou-mac>",
  "pulse_total": 12345,
  "boot_id": "<identificador-da-sessao>",
  "seq": 42
}
```

- `device_id`: string não vazia, identificador do chip/MAC enviado pelo ESP32.
- `pulse_total`: inteiro não negativo, contador cumulativo desde o boot.
- `boot_id`: string não vazia, novo a cada boot.
- `seq`: inteiro não negativo, sequência da leitura dentro do boot.
- A API guarda o horário de recebimento em UTC. Não há `timestamp` no corpo.

### Respostas

| Situação | Código | Corpo esperado |
|---|---:|---|
| Leitura nova processada | `201 Created` | Identificação da leitura e indicação de processamento |
| Mesma leitura reenviada | `200 OK` | `{"duplicate": true}` ou campo equivalente, sem novo consumo |
| Corpo inválido ou campos obrigatórios ausentes | `422 Unprocessable Entity` | Detalhe do erro de validação |
| Token ausente ou desconhecido | `401 Unauthorized` | Erro de autenticação; não processar a leitura |
| Token já vinculado a outro `device_id` | `403 Forbidden` | Erro de vínculo; não processar a leitura |
| Erro temporário ao persistir ou indisponibilidade | `503 Service Unavailable` | Erro temporário; o firmware deve tentar novamente |

Para o firmware, qualquer `2xx` significa que a leitura pode sair do buffer.

## Regras de comportamento da API

### Autenticação e vínculo

- O token é pré-gerado por script administrativo externo ao ESP32 e gravado no firmware.
- A API guarda apenas o hash do token, calculado sem biblioteca externa.
- Tokens desconhecidos são rejeitados com `401`.
- No primeiro uso de um token, a API vincula o `device_id` enviado ao registro desse token.
- Depois do vínculo, o mesmo token enviado com outro `device_id` é rejeitado com `403`.
- Um dispositivo perde o vínculo apenas por processo administrativo futuro de revogação ou provisionamento; não há tela ou rota para cadastro nesta versão.
- O `label` não é usado nem exigido pela ingestão.

### Idempotência

- A chave lógica é `(device_id, boot_id, seq)` e deve ser única no banco.
- Reenviar exatamente a mesma chave responde `200` com `duplicate: true` e não conta consumo novamente.
- A ordem de chegada não altera o resultado.

### Conversão e consumo

- A API persiste os pulsos brutos e não exige que o ESP32 calcule litros.
- O fator inicial é 450 pulsos/litro e fica declarado como uma linha por dispositivo.
- O consumo entre leituras com o mesmo `boot_id` é a diferença entre `pulse_total` atual e o anterior, ordenados por `seq`.
- A diferença negativa não gera consumo negativo; dados inconsistentes são rejeitados e registrados.

### Reinício do dispositivo

- Novo `boot_id` é identificado como reinício.
- Na primeira leitura do novo `boot_id`, o consumo é o próprio `pulse_total`.
- O reinício aparece nos logs; pulsos perdidos entre o último envio e queda de energia não são tratados.

### Leituras atrasadas

- Leituras atrasadas são aceitas independentemente da ordem de chegada.
- O consumo é calculado pela ordem de `seq` dentro do mesmo `boot_id`, não pelo horário de recepção.
- Uma leitura atrasada pode ser processada antes ou depois de leituras posteriores sem alteração da regra por `seq`.

## Critérios de aceite

- [ ] Uma leitura válida com token registrado é vinculada ao primeiro `device_id`, persistida e responde `201`; os pulsos brutos e litros aparecem nos logs.
- [ ] O primeiro uso vincula o token ao `device_id` enviado no corpo.
- [ ] Reusar o mesmo token com outro `device_id` responde `403` e não persiste a leitura.
- [ ] Reenviar `(device_id, boot_id, seq)` existente responde `200` com `duplicate: true` e não adiciona consumo.
- [ ] Leituras atrasadas chegam fora de ordem por `seq` e têm seu consumo calculado pela sequência cumulativa.
- [ ] Um novo `boot_id` usa `pulse_total` como consumo inicial e não produz consumo negativo.
- [ ] Token ausente ou desconhecido responde `401` e não persiste dados.
- [ ] Corpo com campo obrigatório ausente ou tipo inválido responde `422`.
- [ ] 450 pulsos conhecidos resultam em 1 litro pelo fator inicial da v1.

