# Leituras com contador cumulativo, `boot_id` e `seq`

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

A ingestão precisa tolerar três situações comuns: reenvio da mesma leitura, envio atrasado depois de uma queda de Wi-Fi e reinício do ESP32. Em todas elas o consumo não pode ser perdido, duplicado nem ficar negativo (ver [`requisitos.md`](../requisitos.md), seção 1).

## Decisão

Cada leitura enviada pelo dispositivo contém:

- `pulse_total`: contador cumulativo de pulsos desde o boot;
- `boot_id`: identificador da sessão, novo a cada boot;
- `seq`: número sequencial da leitura dentro do boot.

O backend trata assim:

- **Idempotência:** a tripla `(device_id, boot_id, seq)` é única. Uma leitura repetida é ignorada.
- **Consumo de um intervalo:** com o mesmo `boot_id`, é a diferença entre o `pulse_total` atual e o da leitura anterior.
- **Reinício:** com `boot_id` novo, o consumo é o próprio `pulse_total` (pulsos desde o boot) e o reinício é registrado.
- **Envio atrasado:** o consumo é calculado pela ordem de `seq`, e não pela ordem de chegada.

## Consequências

- Reenvios e leituras atrasadas são seguros por construção.
- Uma leitura perdida não perde consumo, porque o contador cumulativo da leitura seguinte já o inclui.
- Os pulsos contados entre o último envio e um reinício inesperado (ex.: queda de energia) se perdem, salvo persistência do contador no dispositivo. Essa perda é aceita na v1 e pode ser tratada depois.
- O `boot_id` precisa ser diferente a cada boot (ex.: valor aleatório); se repetisse, um reinício passaria despercebido.
