# Banco de dados

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

O sistema guarda cadastro (usuários, casas, dispositivos), leituras brutas e, depois, consumos agregados. A idempotência da ingestão exige uma garantia de unicidade confiável (ver [`contador_cumulativo.md`](contador_cumulativo.md)).

## Decisão

O banco de dados é o **PostgreSQL**.

## Consequências

- A unicidade de `(device_id, boot_id, seq)` pode ser garantida pelo próprio banco, e não só pelo código.
- Consultas por período e agregações por hora, dia e mês são bem atendidas na escala prevista.
- É preciso ter um PostgreSQL disponível em desenvolvimento e em produção.
- A forma de acessar o banco pela API (driver ou biblioteca) ainda precisa ser decidida.
