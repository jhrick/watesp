# Separação do projeto em módulos

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

O projeto reúne partes com linguagens, ferramentas e ciclos de mudança diferentes: o firmware do ESP32, a API e, mais adiante, a interface.

## Decisão

Cada parte do projeto vive em **uma pasta própria na raiz do repositório**, com suas dependências e seus testes:

- `esp32/`: firmware do dispositivo.
- `api/`: backend.
- Pastas novas, como a da interface, são criadas quando a parte começar.

A documentação (`intencao.md`, `requisitos.md` e `docs/decisions/`) fica fora dessas pastas.

## Consequências

- Uma parte pode mudar sem mexer nas outras, e o histórico fica mais fácil de ler e reverter.
- O contrato entre as partes (o formato das leituras enviadas pelo ESP32 e recebidas pela API) precisa ser mantido em um só lugar e respeitado nos dois lados.
- Cada módulo tem suas próprias instruções de execução e dependências.
