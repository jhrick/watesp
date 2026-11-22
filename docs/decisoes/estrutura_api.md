# Estrutura interna da API

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

A API vai crescer a partir da ingestão de leituras (autenticação de usuários e consulta de consumo vêm depois). Sem uma organização combinada desde o início, as regras tendem a se misturar com o código HTTP, o que dificulta testar e mudar.

## Decisão

A pasta `api/` separa o código por responsabilidade:

- `routes/`: recebem e respondem requisições HTTP. Não contêm regras de negócio.
- `services/`: contêm as regras (conversão de pulsos, cálculo do consumo entre leituras, tratamento de reinício). Devem funcionar sem depender do HTTP nem do banco, para serem testados de forma isolada.
- `repositories/`: único lugar que conversa com o PostgreSQL.
- `schemas/`: descrevem e validam os dados que entram e saem da API.
- `config.py`: leitura de variáveis de ambiente, sem segredos no código.
- `tests/`: espelham a estrutura acima.

Cada funcionalidade (ingestão, autenticação, consumo) é adicionada como um conjunto próprio de routes, services e repositories, e não como arquivos soltos na raiz.

Nomes de pastas, arquivos de código e termos de programação ficam sempre em inglês.

Esquema de pastas, mostrando só a ingestão (as demais funcionalidades seguem o mesmo padrão):

```
api/
├── app/
│   ├── main.py            # cria o FastAPI e registra as routes
│   ├── config.py          # variáveis de ambiente
│   ├── routes/
│   │   └── ingestion.py   # recebe as leituras (HTTP)
│   ├── services/
│   │   └── ingestion.py   # regras: conversão, consumo, reinício
│   ├── repositories/
│   │   └── readings.py    # consultas ao PostgreSQL
│   └── schemas/
│       └── readings.py    # formato das leituras (entrada e saída)
└── tests/
    ├── routes/
    ├── services/
    └── repositories/
```

## Consequências

- As regras mais críticas (conversão e consumo entre leituras) ficam testáveis sem subir servidor ou banco.
- Trocar a forma de acessar o banco não afeta as regras.
- A escolha da biblioteca de acesso ao PostgreSQL fica para outro arquivo de decisão.
- Esta estrutura é uma proposta e pode ser ajustada quando a primeira funcionalidade for implementada.
