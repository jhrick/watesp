# Schema do banco

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

A API precisa criar as estruturas mínimas para dispositivos, tokens por hash, leituras e fator de calibração sem introduzir mais dependências nesta versão.

## Decisão

- O PostgreSQL é criado a partir de um único arquivo `schema.sql` na raiz de `api/`.
- Este arquivo contém as tabelas e garantias de unicidade necessárias pela ingestão.
- Não serão usadas migrations nesta versão.

## Desenho das tabelas

### Tabela `devices`

Um registro por token gerado.

| Coluna | Tipo | Restrições |
|---|---|---|
| id | bigint | chave primária, gerada automaticamente |
| token_hash | text | obrigatório, único |
| device_id | text | único, nulo até o primeiro uso do token |
| pulses_per_liter | integer | obrigatório, padrão 450 |
| created_at | timestamptz | obrigatório, padrão now() |

### Tabela `readings`

Uma linha por leitura recebida.

| Coluna | Tipo | Restrições |
|---|---|---|
| id | bigint | chave primária, gerada automaticamente |
| device_ref | bigint | obrigatório, referência a `devices.id` |
| boot_id | text | obrigatório |
| seq | integer | obrigatório, maior ou igual a 0 |
| pulse_total | bigint | obrigatório, maior ou igual a 0 |
| received_at | timestamptz | obrigatório, padrão now() |

Restrição única em `readings`: `(device_ref, boot_id, seq)`. Ela garante a idempotência e também serve de índice para ordenar as leituras por `seq` dentro de um boot.

- `device_id` é o identificador enviado pelo ESP32 (chip/MAC) e fica nulo até o primeiro uso do token, quando a API o vincula. A coluna `id` é a chave interna usada nas referências, e por isso a coluna de `readings` se chama `device_ref`.
- Os litros não são guardados. São calculados a partir dos pulsos, usando `pulses_per_liter` (ver `conversao_pulsos.md`).
- O `label` (nome da casa) não entra no schema agora; será adicionado quando for implementado.
- Não incluir tabelas de usuários, casas ou alertas nesta versão.

## Consequências

- Desenvolvimento e instalação podem preparar o banco de forma simples e reproduzível.
- Alterações futuras no schema exigirão revisão manual ou a introdução posterior de migrations.
- Não é necessário adicionar uma biblioteca ou ferramenta de migrations às dependências da v1.