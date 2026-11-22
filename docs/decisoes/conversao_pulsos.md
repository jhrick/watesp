# Conversão de pulsos para litros no backend

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

O sensor YF-S201 gera pulsos, e o fator de conversão para litros (≈450 pulsos/litro) é nominal, com precisão de cerca de ±10%. Esse fator precisa ser calibrado por instalação e pode mudar depois que o sistema já estiver em uso.

## Decisão

O dispositivo envia **somente pulsos brutos**. O backend guarda os pulsos e calcula os litros usando o fator de calibração do dispositivo.

## Consequências

- Recalibrar um sensor não exige atualizar o firmware.
- Os pulsos brutos são a fonte da verdade, e os litros podem ser recalculados a qualquer momento.
- O fator de calibração é um atributo do dispositivo e precisa de histórico, para que períodos antigos usem o fator que valia na época.
- O firmware fica mais simples e com menos motivos para mudar.
