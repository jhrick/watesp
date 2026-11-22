# Linguagem e framework do backend

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

O backend recebe leituras, aplica as regras de conversão e serve os dados para a interface. A escala é pequena (poucas casas, uma leitura a cada algumas dezenas de segundos), então simplicidade pesa mais do que recursos avançados.

## Decisão

O backend é escrito em **Python com FastAPI**, usando **o menor número possível de dependências**. Antes de adicionar uma biblioteca, a pergunta é se o problema se resolve com a biblioteca padrão ou com o que o FastAPI já traz.

## Consequências

- Menos dependências significam menos superfície de manutenção, de atualização e de falhas de segurança.
- Cada dependência nova precisa de uma justificativa, e as relevantes podem virar um arquivo de decisão próprio.
- O FastAPI traz validação de dados e documentação automática da API, o que evita bibliotecas extras para isso.
